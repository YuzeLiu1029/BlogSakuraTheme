---
title: Sorting_Algorithm
author: Liu Yuze
avatar: https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg
authorLink: liuyuze.site
authorAbout: 好少年光芒万丈
authorDesc: 好少年光芒万丈
categories: 技术
mathjax: true
comments: true
date: 2019-09-08 13:53:53
tags:
- 技术
- 面试
- 算法
keywords: 排序
description: 总结常用的排序算法,Java
photos: https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/123829.jpg
---
# 冒泡排序（Bubble Sort）

``` java
public static void bubbleSort(int[] arr){
    if(arr.length == 0){
        return;
    }
    for(int i = 0; i < arr.length; i++){
        boolean flag = false;
        for(int j = 0; j < array.length - 1 - i; j++){
            if(arr[j] > arr[j+1]){
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
                flag = true;
            }
        }
        if(!flag) break; 
    }
}

```
最佳情况： $T(n) = O(n)$
最差情况：$T(n) = O(n^2)$
平均情况：$T(n) = O(n^2)$

# 选择排序(Selection Sort)
```java
public static void selectionSort(int[] arr){
    if(arr.length <= 1){
        return;
    }
    for(int i = 0; i < arr.length; i++){
        int minIndex = i;
        for(int j = i; j < arr.length; j++){
            if(arr[j] < arr[minIndex]){
                minIndex = j;
            }
        }
        int temp = arr[minIndex];
        arr[minIndex] = arr[i];
        arr[i] = temp;
    }
    return;
}
```
最佳情况：$T(n) = O(n^2)$ 
最差情况：$T(n) = O(n^2)$  
平均情况：$T(n) = O(n^2)$

# 插入排序(Insertion Sort)
```java
public static void insertionSort(int[] arr){
    if(arr.length <= 1){
        return;
    }
    int cur;
    for(int i = 0; i < arr.length-1; i++){
        cur = arr[i+1];
        int preIndex = i;
        while(preIndex >= 0 && cur < arr[preIndex]){
            arr[preIndex+1] = arr[preIndex];
            preIndex--;
        }
        arr[preIndex + 1] = cur;
    }
    return;
}
```
最佳情况：$T(n) = O(n)$
最差情况：$T(n) = O(n^2)$
平均情况：$T(n) = O(n^2)$


# 归并排序(Merge Sort)
分治法(Divide and Conquer)

```java
public static void MergeSort(int[] arr){
    if(arr.length < 2) return arr;
    int mid = arr.length / 2;
    int[] left = Arrays.copyOfRange(arr,0,mid);
    int[] right = Arrays.copyOfRange(arr,mid,arr.length);
    return merge(MergeSort(left),MergeSort(right));
}

public static int[] merge(int[] left, int[] right){
    int[] result = new int[left.length + right.length];
    for(int index = 0, i = 0, j = 0; index < result < result.length; index++){
        if(i >= left.length) result[index] = right[j++];
        else if(j >= right.length) result[index] = left[i++];
        else if(left[i] > right[j]) result[index] = right[j++];
        else result[index] = left[i++];
    }
    return result;
}
```
最佳情况：$T(n) = O(n)$
最差情况：$T(n) = O(nlogn)$
平均情况：$T(n) = O(nlogn)$

# 快速排序(Quick Sort)
```java

    public static int[] QuickSort(int[] array, int start, int end) {
        if (array.length < 1 || start < 0 || end >= array.length || start > end) return null;
        int smallIndex = partition(array, start, end);
        if (smallIndex > start)
            QuickSort(array, start, smallIndex - 1);
        if (smallIndex < end)
            QuickSort(array, smallIndex + 1, end);
        return array;
    }
   
    public static int partition(int[] array, int start, int end) {
        int pivot = (int) (start + Math.random() * (end - start + 1));
        int smallIndex = start - 1;
        swap(array, pivot, end);
        for (int i = start; i <= end; i++)
            if (array[i] <= array[end]) {
                smallIndex++;
                if (i > smallIndex)
                    swap(array, i, smallIndex);
            }
        return smallIndex;
    }


    public static void swap(int[] array, int i, int j) {
        int temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }
```
最佳情况：$T(n) = O(nlogn)$ 
最差情况：$T(n) = O(n^2)$ 
平均情况：$T(n) = O(nlogn)$

# 堆排序(Heap Sort)

```java
//声明全局变量，用于记录数组array的长度；
static int len;
    /**
     * 堆排序算法
     *
     * @param array
     * @return
     */
    public static int[] HeapSort(int[] array) {
        len = array.length;
        if (len < 1) return array;
        //1.构建一个最大堆
        buildMaxHeap(array);
        //2.循环将堆首位（最大值）与末位交换，然后在重新调整最大堆
        while (len > 0) {
            swap(array, 0, len - 1);
            len--;
            adjustHeap(array, 0);
        }
        return array;
    }
    /**
     * 建立最大堆
     *
     * @param array
     */
    public static void buildMaxHeap(int[] array) {
        //从最后一个非叶子节点开始向上构造最大堆
        for (int i = (len/2 - 1); i >= 0; i--) { //感谢 @让我发会呆 网友的提醒，此处应该为 i = (len/2 - 1) 
            adjustHeap(array, i);
        }
    }
    /**
     * 调整使之成为最大堆
     *
     * @param array
     * @param i
     */
    public static void adjustHeap(int[] array, int i) {
        int maxIndex = i;
        //如果有左子树，且左子树大于父节点，则将最大指针指向左子树
        if (i * 2 < len && array[i * 2] > array[maxIndex])
            maxIndex = i * 2;
        //如果有右子树，且右子树大于父节点，则将最大指针指向右子树
        if (i * 2 + 1 < len && array[i * 2 + 1] > array[maxIndex])
            maxIndex = i * 2 + 1;
        //如果父节点不是最大值，则将父节点与最大值交换，并且递归调整与父节点交换的位置。
        if (maxIndex != i) {
            swap(array, maxIndex, i);
            adjustHeap(array, maxIndex);
        }
    }　
```
最佳情况：$T(n) = O(nlogn)$
最差情况：$T(n) = O(nlogn)$
平均情况：$T(n) = O(nlogn)$

# 计数排序(Count Sort)

```java
**
     * 计数排序
     *
     * @param array
     * @return
     */
    public static int[] CountingSort(int[] array) {
        if (array.length == 0) return array;
        int bias, min = array[0], max = array[0];
        for (int i = 1; i < array.length; i++) {
            if (array[i] > max)
                max = array[i];
            if (array[i] < min)
                min = array[i];
        }
        bias = 0 - min;
        int[] bucket = new int[max - min + 1];
        Arrays.fill(bucket, 0);
        for (int i = 0; i < array.length; i++) {
            bucket[array[i] + bias]++;
        }
        int index = 0, i = 0;
        while (index < array.length) {
            if (bucket[i] != 0) {
                array[index] = i - bias;
                bucket[i]--;
                index++;
            } else
                i++;
        }
        return array;
    }
```
最佳情况：$T(n) = O(n+k)$  最差情况：$T(n) = O(n+k)$  平均情况：$T(n) = O(n+k)$



# 不同时间复杂度表达形式

$O：f(n)=O(g(n))$，也就是 $∃c>0，n_0$ 
当 
$n≥n_0$时候，有$f(n)≤cg(n)$ 
$g(n)$是$f(n)$的渐进上界。
​
​$\Omega : f(n) = \Omega(g(n))$ :
$n≥n_0$时候，有$f(n) \geq cg(n)$ 
$g(n)$是$f(n)$的渐进下界。	
 
​$\Theta : f(n) = \Theta(g(n))$ :
$n≥ n_0$时候，有$ c_1g(n) \leq  f(n) \leq c_2g(n)$ 
$g(n)$是$f(n)$的渐进同阶。



