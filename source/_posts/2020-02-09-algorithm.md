---
title: algorithm
toc: true
categories:
  - 技术笔记
date: 2020-02-09 18:21:47
tags:
  - 算法
---
### 常见排序列表

![常见排序列表](https://casparthh.github.io/2020/02/09/algorithm/algorithm.png)
<!--more-->

### 选择排序
选择排序是给定位置去找数。选择排序是按顺序比较，找最大值或者最小值，每一轮比较都只需要换一次位置；  

```
/**
 * 由于每一次是前一个和后一个对比，所以循环的次数应该是 length - 1;
 * @param nums
 */
public static void selection(int[] nums) {
    for (int i = 0; i < nums.length - 1; i++) {
        int minIndex = i;
        for (int j = i + 1; j < nums.length; j++) {
            minIndex = nums[minIndex] > nums[j] ? j : minIndex;
        }
        swap(nums, i, minIndex);
    }
}

/**
 * 选择排序优化版，每次外层循环找出最小值和最大值，分别放到数组的前后，减少外层循环
 * @param nums
 */
public static void selection(int[] nums) {
    for (int i = 0, len = nums.length; i < len; i++) {
        int minIndex = i == 0 ? 0 : len;
        int maxIndex = i == 0 ? len - 1 : 0;
        for (int j = i; j < len; j++) {
            minIndex = nums[minIndex] > nums[j] ? j : minIndex;
            maxIndex = nums[maxIndex] > nums[j] ? maxIndex : j;
        }
        swap(nums, i, minIndex);
        maxIndex = maxIndex == i ? minIndex : maxIndex;
        swap(nums, maxIndex, len - 1);
        len--;
    }
}
```

### 冒泡排序
冒泡排序是通过数去找位置，比较相邻位置的两个数，每一轮比较后，位置不对都需要换位置
```
/**
 * 由于前后两位对比，所以外层循环的次数应该是 length - 1;
 * 内存循环次数是当前数组未排序长度 - 1
 *
 * @param nums
 */
public static void bubble(int[] nums) {
    for (int i = 1; i < nums.length; i++) {
        for (int j = 0; j < nums.length - i; j++) {
            if (nums[j] > nums[j + 1]) {
                swap(nums, j, j + 1);
            }
        }
    }
}
```


### 插入排序
每一步将一个待排序的记录，插入到前面已经排好序的有序序列中去，直到插完所有元素为止。
```
/**
 * 插入排序
 *
 * @param nums
 */
public static void insertion(int[] nums) {
    for (int i = 1; i < nums.length; i++) {
        for (int j = i; j > 0; j--) {
            if (nums[j] < nums[j - 1]) {
                swap(nums, j, j - 1);
            } else {
                //如果当前元素大于等于上一个元素了，跳出内循环。
                break;
            }
        }
    }
}
```


### 归并排序
归并排序（MERGE-SORT）是利用归并的思想实现的排序方法，该算法采用经典的分治（divide-and-conquer）策略（分治法将问题分(divide)成一些小的问题然后递归求解，而治(conquer)的阶段则将分的阶段得到的各答案"修补"在一起，即分而治之)。  
分解：将列表越分越小，直至分成一个元素。  
终止条件：一个元素是有序的。    
合并：将两个有序列表归并，列表越来越大。  

```
public static void merge(int[] nums, int left, int right) {
    if (right == left) {
        return;
    }
    //分两半
    int mid = (right - left) / 2 + left;
    merge(nums, left, mid);
    merge(nums, mid + 1, right);
    merge(nums, left, mid + 1, right);
}

public static void merge(int[] nums, int leftIndex, int rightIndex, int rightBound) {
    int leftBound = rightIndex - 1;
    int[] buffer = new int[rightBound - leftIndex + 1];

    int l = leftIndex, r = rightIndex, bufferIndex = 0;

    while (l <= leftBound && r <= rightBound) {
        buffer[bufferIndex++] = nums[l] <= nums[r] ? nums[l++] : nums[r++];
    }

    while (l <= leftBound) {
        buffer[bufferIndex++] = nums[l++];
    }

    while (r <= rightBound) {
        buffer[bufferIndex++] = nums[r++];
    }
    for (int i = 0; i < buffer.length; i++) {
        nums[leftIndex + i] = buffer[i];
    }
}
```

#### 快速排序
速排序是常用的几种排序中空间时间最优秀的排序，可以适用各种数据结构。
快排的方法是先给定一个数，然后将待排序的数据分别于这个数比较，从左到右的遍历下标，
比这个数大的放在右边，比这个数小的放在左边，
而后分别对左边和右边的数据做递归引用排序，最终就能得到有序的数据
```
public static void quick(int[] nums, int left, int rightBound) {
    int pivot = nums[(rightBound - left) / 2 + left];
    int l = left;
    int rb = rightBound;
    while (l <= rb) {
        while (nums[l] < pivot) {
            l++;
        }
        while (nums[rb] > pivot) {
            rb--;
        }
        if (l <= rb) {
            swap(nums, l, rb);
            l++;
            rb--;
        }
    }
    if (left < l) {
        quick(nums, left, rb);
    }
    if (l < rightBound) {
        quick(nums, l, rightBound);
    }
}
```
