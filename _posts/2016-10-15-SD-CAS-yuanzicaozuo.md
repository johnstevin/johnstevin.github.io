---
layout: post
title: CAS原子操作实现无锁及性能分析
categories: SoftwareDesign
description: CAS原子操作实现无锁及性能分析：无锁操作在性能上远远优于加锁操作，消耗时间仅为加锁操作的1/3左右，无锁编程方式确实能够比传统加锁方式效率高，经上面测试可以发现，可以快到3倍左右。所以在极力推荐在高并发程序中采用无锁编程的方式可以进一步提高程序效率。
keywords: CAS,无锁,性能分析,并发,性能
---

CAS原子操作实现无锁及性能分析：无锁操作在性能上远远优于加锁操作，消耗时间仅为加锁操作的1/3左右，无锁编程方式确实能够比传统加锁方式效率高，经上面测试可以发现，可以快到3倍左右。所以在极力推荐在高并发程序中采用无锁编程的方式可以进一步提高程序效率。

最近在研究nginx的自旋锁的时候，又见到了GCC CAS原子操作，于是决定动手分析下CAS实现的无锁到底性能如何，网上关于CAS实现无锁的文章很多，但少有研究这种无锁的性能提升的文章，这里就以实验结果和我自己的理解逐步展开。

#### 1. 什么是CAS原子操作

在研究无锁之前，我们需要首先了解一下CAS原子操作——Compare & Set，或是 Compare & Swap，现在几乎所image有的CPU指令都支持CAS的原子操作，X86下对应的是 CMPXCHG 汇编指令。

大家应该还记得操作系统里面关于“原子操作”的概念，一个操作是原子的(atomic)，如果这个操作所处的层(layer)的更高层不能发现其内部实现与结构。原子操作可以是一个步骤，也可以是多个操作步骤，但是其顺序是不可以被打乱，或者切割掉只执行部分。有了这个原子操作这个保证我们就可以实现无锁了。

CAS原子操作在维基百科中的代码描述如下:

```cpp
int compare_and_swap(int* reg, int oldval, int newval)
{
  ATOMIC();
  int old_reg_val = *reg;
  if (old_reg_val == oldval)
     *reg = newval;
  END_ATOMIC();
  return old_reg_val;
}
```

也就是检查内存*reg里的值是不是oldval，如果是的话，则对其赋值newval。上面的代码总是返回old_reg_value，调用者如果需要知道是否更新成功还需要做进一步判断，为了方便，它可以变种为直接返回是否更新成功，如下：

```cpp
bool compare_and_swap (int *accum, int *dest, int newval)
{
  if ( *accum == *dest ) {
      *dest = newval;
      return true;
  }
  return false;
}
```

除了CAS还有以下原子操作：

Fetch And Add，一般用来对变量做 +1 的原子操作。

```cpp
<< atomic >>
function FetchAndAdd(address location, int inc) {
    int value := *location
    *location := value + inc
    return value
}
```
 
Test-and-set，写值到某个内存位置并传回其旧值。汇编指令BST。

```cpp
#define LOCKED 1
 
int TestAndSet(int* lockPtr) {
    int oldValue;
 
    // Start of atomic segment
    // The following statements should be interpreted as pseudocode for
    // illustrative purposes only.
    // Traditional compilation of this code will not guarantee atomicity, the
    // use of shared memory (i.e. not-cached values), protection from compiler
    // optimization, or other required properties.
    oldValue = *lockPtr;
    *lockPtr = LOCKED;
    // End of atomic segment
 
    return oldValue;
}

```

Test and Test-and-set，用来实现多核环境下互斥锁。

```cpp
boolean locked := false // shared lock variable
procedure EnterCritical() {
  do {
    while (locked == true) skip // spin until lock seems free
  } while TestAndSet(locked) // actual atomic locking
}
```

#### 2.CAS 在各个平台下的实现

2.1 Linux GCC 支持的 CAS

GCC4.1+版本中支持CAS的原子操作（完整的原子操作可参看 GCC Atomic Builtins）

```cpp
bool __sync_bool_compare_and_swap (type *ptr, type oldval type newval, ...)
type __sync_val_compare_and_swap (type *ptr, type oldval type newval, ...)
```

2.2  Windows支持的CAS

在Windows下，你可以使用下面的Windows API来完成CAS：（完整的Windows原子操作可参看MSDN的InterLocked Functions）

```cpp
InterlockedCompareExchange ( __inout LONG volatile *Target,
                                __in LONG Exchange,
                                __in LONG Comperand);
```

2.3  C++ 11支持的CAS

C++11中的STL中的atomic类的函数可以让你跨平台。（完整的C++11的原子操作可参看 Atomic Operation Library）

```cpp
template< class T >
bool atomic_compare_exchange_weak( std::atomic<T>* obj,
                                   T* expected, T desired );
template< class T >
bool atomic_compare_exchange_weak( volatile std::atomic<T>* obj,
                                   T* expected, T desired );
```

#### 3.CAS原子操作实现无锁的性能分析
 
3.1 测试方法描述
 
这里由于只是比较性能，所以采用很简单的方式，创建10个线程并发执行，每个线程中循环对全局变量count进行++操作（i++)，循环加2000000次，这必然会涉及到并发互斥操作，在同一台机器上分析 加普通互斥锁、CAS实现的无锁、Fetch And Add实现的无锁消耗的时间，然后进行分析。
 
3.2 加普通互斥锁代码

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <time.h>
#include "timer.h"
 
pthread_mutex_t mutex_lock;
static volatile int count = 0;
void *test_func(void *arg)
{
        int i = 0;
        for(i = 0; i < 2000000; i++)
        {
                pthread_mutex_lock(&mutex_lock);
                count++;
                pthread_mutex_unlock(&mutex_lock);
        }
        return NULL;
}
 
int main(int argc, const char *argv[])
{
    Timer timer; // 为了计时，临时封装的一个类Timer。
    timer.Start();    // 计时开始
    pthread_mutex_init(&mutex_lock, NULL);
    pthread_t thread_ids[10];
    int i = 0;
    for(i = 0; i < sizeof(thread_ids)/sizeof(pthread_t); i++)
    {
        pthread_create(&thread_ids[i], NULL, test_func, NULL);
    }
 
    for(i = 0; i < sizeof(thread_ids)/sizeof(pthread_t); i++)
    {
        pthread_join(thread_ids[i], NULL);
    }
 
    timer.Stop();// 计时结束
    timer.Cost_time();// 打印花费时间
    printf("结果:count = %d\n",count);
 
    return 0;
}
```

注：Timer类仅作统计时间用，其实现在文章最后给出。
 
3.2 CAS实现的无锁

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>
#include <time.h>
#include "timer.h"
 
int mutex = 0;
int lock = 0;
int unlock = 1;
 
static volatile int count = 0;
void *test_func(void *arg)
{
        int i = 0;
        for(i = 0; i < 2000000; i++)
    {
        while (!(__sync_bool_compare_and_swap (&mutex,lock, 1) ))usleep(100000);
         count++;
         __sync_bool_compare_and_swap (&mutex, unlock, 0);
        }
        return NULL;
}
 
int main(int argc, const char *argv[])
{
    Timer timer;
    timer.Start();
    pthread_t thread_ids[10];
    int i = 0;
 
    for(i = 0; i < sizeof(thread_ids)/sizeof(pthread_t); i++)
    {
            pthread_create(&thread_ids[i], NULL, test_func, NULL);
    }
 
    for(i = 0; i < sizeof(thread_ids)/sizeof(pthread_t); i++)
    {
            pthread_join(thread_ids[i], NULL);
    }
 
    timer.Stop();
    timer.Cost_time();
    printf("结果:count = %d\n",count);
 
    return 0;
}
```
 
3.4 Fetch And Add 原子操作
 
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>
#include <time.h>
#include "timer.h"
 
static volatile int count = 0;
void *test_func(void *arg)
{
        int i = 0;
        for(i = 0; i < 2000000; i++)
        {
            __sync_fetch_and_add(&count, 1);
        }
        return NULL;
}
 
int main(int argc, const char *argv[])
{
    Timer timer;
    timer.Start();
    pthread_t thread_ids[10];
    int i = 0;
 
    for(i = 0; i < sizeof(thread_ids)/sizeof(pthread_t); i++){
            pthread_create(&thread_ids[i], NULL, test_func, NULL);
    }
 
    for(i = 0; i < sizeof(thread_ids)/sizeof(pthread_t); i++){
            pthread_join(thread_ids[i], NULL);
    }
 
    timer.Stop();
    timer.Cost_time();
    printf("结果:count = %d\n",count);
    return 0;
}
```
 
4 实验结果和分析

在同一台机器上，各运行以上3份代码10次，并统计平均值，其结果如下：（单位微秒）

![原子性能测试](http://o9w2lbvnn.bkt.clouddn.com/images/sd/yuanziceshi.jpg)

由此可见，无锁操作在性能上远远优于加锁操作，消耗时间仅为加锁操作的1/3左右，无锁编程方式确实能够比传统加锁方式效率高，经上面测试可以发现，可以快到3倍左右。所以在极力推荐在高并发程序中采用无锁编程的方式可以进一步提高程序效率。

#### 5.时间统计类Timer

timer.h

```cpp
#ifndef TIMER_H
#define TIMER_H
 
#include <sys/time.h>
class Timer
{
public:    
    Timer();
    // 开始计时时间
    void Start();
    // 终止计时时间
    void Stop();
    // 重新设定
    void Reset();
    // 耗时时间
    void Cost_time();
private:
    struct timeval t1;
    struct timeval t2;
    bool b1,b2;
};
#endif
```


timer.cpp

```cpp
#include "timer.h"
#include <stdio.h>
 
Timer::Timer()
{
    b1 = false;
    b2 = false;
}
void Timer::Start()
{
    gettimeofday(&t1,NULL);  
    b1 = true;
    b2 = false;
}
 
void Timer::Stop()
{    
    if (b1 == true)
    {
        gettimeofday(&t2,NULL);  
        b2 = true;
    }
}
 
void Timer::Reset()
{    
    b1 = false;
    b2 = false;
}
 
void Timer::Cost_time()
{
    if (b1 == false)
    {
        printf("计时出错，应该先执行Start()，然后执行Stop()，再来执行Cost_time()");
        return ;
    }
    else if (b2 == false)
    {
        printf("计时出错，应该执行完Stop()，再来执行Cost_time()");
        return ;
    }
    else
    {
        int usec,sec;
        bool borrow = false;
        if (t2.tv_usec > t1.tv_usec)
        {
            usec = t2.tv_usec - t1.tv_usec;
        }
        else
        {
            borrow = true;
            usec = t2.tv_usec+1000000 - t1.tv_usec;
        }
 
        if (borrow)
        {
            sec = t2.tv_sec-1 - t1.tv_sec;
        }
        else
        {
            sec = t2.tv_sec - t1.tv_sec;
        }
        printf("花费时间:%d秒 %d微秒\n",sec,usec);
    }
}
```
