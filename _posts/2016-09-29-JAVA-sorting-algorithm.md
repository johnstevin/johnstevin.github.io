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

1. 直接插入排序（从后向前找到合适位置后插入）

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

