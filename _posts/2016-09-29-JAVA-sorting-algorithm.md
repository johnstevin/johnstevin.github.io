---
layout: post
title: 排序算法大全
categories: JAVA
description: 各种排序算法的分析及java实现
keywords: JAVA,编程
---

排序大的分类可以分为两种：内排序和外排序。在排序过程中，全部记录存放在内存，则称为内排序，如果排序过程中需要使用外存，则称为外排序。
下面讲的排序都是属于内排序。

内排序有可以分为以下几类：

- 插入排序：直接插入排序、二分法插入排序、希尔排序。
- 选择排序：简单选择排序、堆排序。
- 交换排序：冒泡排序、快速排序。
- 归并排序
- 基数排序

#### 插入排序 

1. 直接插入排序（从后向前找到合适位置后插入）基本思想：每步将一个待排序的记录，按其顺序码大小插入到前面已经排序的字序列的合适位置（从后向前找到合适位置后），直到全部插入排序完为止。文件初态不同时，直接插入排序所耗费的时间有很大差异。若文件初态为正序，则每个待插入的记录只需要比较一次就能够找到合适的位置插入，故算法的时间复杂度为O(n)，这时最好的情况。若初态为反序，则第i个待插入记录需要比较i+1次才能找到合适位置插入，故时间复杂度为O(n2)，这时最坏的情况。直接插入排序的平均时间复杂度为O(n2)。

```java
public class Sort {
    public static void main(String[] args) {
        int[] a={49,38,65,97,76,13,27,49,78,34,12,64,1};
        System.out.println("排序之前：");
        for (int i = 0; i < a.length; i++) {
            System.out.print(a[i]+" ");
        }
        //直接插入排序
        for (int i = 1; i < a.length; i++) {
            //待插入元素
            int temp = a[i];
            int j;
            /*for (j = i-1; j>=0 && a[j]>temp; j--) {
                //将大于temp的往后移动一位
                a[j+1] = a[j];
            }*/
            for (j = i-1; j>=0; j--) {
                //将大于temp的往后移动一位
                if(a[j]>temp){
                    a[j+1] = a[j];
                }else{
                    break;
                }
            }
            a[j+1] = temp;
        }
        System.out.println();
        System.out.println("排序之后：");
        for (int i = 0; i < a.length; i++) {
            System.out.print(a[i]+" ");
        }
        System.out.println();
    }

}
```

2. 二分法插入排序（按二分法找到合适位置插入）基本思想：二分法插入排序的思想和直接插入一样，只是找合适的插入位置的方式不同，这里是按二分法找到合适的位置，可以减少比较的次数。当然，二分法插入排序也是稳定的。二分插入排序的比较次数与待排序记录的初始状态无关，仅依赖于记录的个数。当n较大时，比直接插入排序的最大比较次数少得多。但大于直接插入排序的最小比较次数。算法的移动次数与直接插入排序算法的相同，最坏的情况为n2/2，最好的情况为n，平均移动次数为O(n2)。

```java
public class Sort {
    public static void main(String[] args) {
        int[] a={49,38,65,97,176,213,227,49,78,34,12,164,11,18,1};
        System.out.println("排序之前：");
        for (int i = 0; i < a.length; i++) {
            System.out.print(a[i]+" ");
        }
        //二分插入排序
        sort(a);
        System.out.println();
        System.out.println("排序之后：");
        for (int i = 0; i < a.length; i++) {
            System.out.print(a[i]+" ");
        }
        System.out.println();
    }

    private static void sort(int[] a) {
        for (int i = 0; i < a.length; i++) {
            int temp = a[i];
            int left = 0;
            int right = i-1;
            int mid = 0;
            while(left<=right){
                mid = (left+right)/2;
                if(temp<a[mid]){
                    right = mid-1;
                }else{
                    left = mid+1;
                }
            }
            for (int j = i-1; j >= left; j--) {
                a[j+1] = a[j];
            }
            if(left != i){
                a[left] = temp;
            }
        }
    }
}
```

3. 希尔排序，基本思想：先取一个小于n的整数d1作为第一个增量，把文件的全部记录分成d1个组。所有距离为d1的倍数的记录放在同一个组中。先在各组内进行直接插入排序；然后，取第二个增量d2<d1重复上述的分组和排序，直至所取的增量dt=1(dt<dt-l<…<d2<d1)，即所有记录放在同一组中进行直接插入排序为止。该方法实质上是一种分组插入方法。

我们知道一次插入排序是稳定的，但在不同的插入排序过程中，相同的元素可能在各自的插入排序中移动，最后其稳定性就会被打乱，所以希尔排序是不稳定的。希尔排序的时间性能优于直接插入排序，原因如下：

- 当文件初态基本有序时直接插入排序所需的比较和移动次数均较少。
- 当n值较小时，n和n2的差别也较小，即直接插入排序的最好时间复杂度O(n)和最坏时间复杂度0(n2)差别不大。
- 在希尔排序开始时增量较大，分组较多，每组的记录数目少，故各组内直接插入较快，后来增量di逐渐缩小，分组数逐渐减少，而各组的记录数目逐渐增多，但由于已经按di-1作为距离排过序，使文件较接近于有序状态，所以新的一趟排序过程也较快。

因此，希尔排序在效率上较直接插人排序有较大的改进。希尔排序的平均时间复杂度为O(nlogn)。

```java
//不稳定
public class Sort {
    public static void main(String[] args) {
        int[] a={49,38,65,97,76,13,27,49,78,34,12,64,1};
        System.out.println("排序之前：");
        for (int i = 0; i < a.length; i++) {
            System.out.print(a[i]+" ");
        }
        //希尔排序
        int d = a.length;
        while(true){
            d = d / 2;
            for(int x=0;x<d;x++){
                for(int i=x+d;i<a.length;i=i+d){
                    int temp = a[i];
                    int j;
                    for(j=i-d;j>=0&&a[j]>temp;j=j-d){
                        a[j+d] = a[j];
                    }
                    a[j+d] = temp;
                }
            }
            if(d == 1){
                break;
            }
        }
        System.out.println();
        System.out.println("排序之后：");
        for (int i = 0; i < a.length; i++) {
            System.out.print(a[i]+" ");
        }
    }

}
```
