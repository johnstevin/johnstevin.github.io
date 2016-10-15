---
layout: post
title: 踢猴子游戏
categories: PHP
description: 一群猴子排成一圈，按1，2，…，n依次编号。然后从第1只开始数，数到第m只,把它踢出圈，从它后面再开始数，再数到第m只，在把它踢出去…，
keywords: PHP,编程
---

PHP编程

```php
/**
 * 一群猴子排成一圈，按1，2，…，n依次编号。然后从第1只开始数，数到第m只,把它踢出圈，从它后面再开始数，再数到第m只，在把它踢出去…，
 * 如此不停的进行下去，直到最后只剩下一只猴子为止，那只猴子就叫做大王。
 * 要求编程模拟此过程，输入m、n, 输出最后那个大王的编号。用程序模拟该过程。
 *
 * @param $n 编号
 * @param $m 指定个数
 */
function monkey($n, $m) {
    $monkeys = range(1, $n);
    $step = 0; // 计步器
    $unsetNum = 0; // 剔除计数器
    while (1) {
       for ($i = 0; $i < $n; $i++) {
           if ($monkeys[$i] != 0) {
               $step++;
               if ($step == $m) {
                   $unsetNum++;
                   if ($unsetNum == $n) die( "-> $unsetNum : 猴王是$monkeys[$i] \n");
                   echo "-> $unsetNum : unset $monkeys[$i] \n";
                   $monkeys[$i] = 0 && $step = 0;
               }
           }
       }
    }
}

monkey(10, 4);

/** 第二种解法 **/
function monkey ($n = 0, $m = 0) {
	$monkeys = range(1, $n);
	$i = 0; // 跟踪索引号
	$step = 0; // 步长计数器(排除剔除的猴子)
	$unNum = 0; // 剔除计数器
	while (1) {
		if ($monkeys[$i] != 0) $step++; // 剔除的猴子不计数
		if ($unNum >= $n) break;
		if ($step == $m) {
			$unNum++;
			$temp = $monkeys[$i];
			$monkeys[$i] = 0;
			$step = 0;
			echo "剔除 $temp 号猴子\n";
		}
		$i++;
		if ($i >= $n) $i = 0; // 当索引号大于n
	}
}

monkey(10, 6);

```

JAVA编程

```java
import java.util.Scanner;

public class monkey {
    public static void main (String[] args) {
        //// 获取N值
        System.out.println("请输入N值");
        Scanner sc = new Scanner(System.in);
        int temp= sc.nextInt();
        //// 初始化数组
        int [] temparray=new int[temp];
        for(int i=0;i<temp;i++){
            temparray[i]=i+1;
        }

        //// 获取M值
        System.out.println("请输入M值");
        int tempM = sc.nextInt();

        //// 计数器,步进值
        int flag=0;
        int outtimes=1;

        for(int i=temp;i>0;){
            for(int j=0;j<temparray.length;j++){
                if(temparray[j]!=0){
                    if(flag==tempM-1){
                        System.out.println("第"+outtimes+"次,输出数据为:"+temparray[j]);
                        temparray[j]=0;
                        flag=0;
                        outtimes++;
                    }else{
                        flag++;
                    }
                }
            }
        }
    }
}
```
