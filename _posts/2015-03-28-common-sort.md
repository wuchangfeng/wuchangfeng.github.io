---
layout: post
title: 常见排序算法
date: 2015-03-30 16:18:04 +0800
categories: 
---

### 排序算法

- 归并排序[联想到合并两个有序的数组] http://flyingcat2013.blog.51cto.com/7061638/1281026

  ```java
  // 归并排序
  public static void mergeSort(int[] arr){
          // 创建一个临时数组，用来存储合并之后的数组
          int[] temp =new int[arr.length];
          internalMergeSort(arr, temp, 0, arr.length-1);
  }

  private static void internalMergeSort(int[] a, int[] b, int left, int right){
          // 当left==right的时，已经不需要再划分了
          if (left<right){
              int middle = (left+right)/2;
              internalMergeSort(a, b, left, middle);          //左子数组
              internalMergeSort(a, b, middle+1, right);       //右子数组
              mergeSortedArray(a, b, left, middle, right);    //合并两个子数组
          }
      }

  // 合并两个有序子序列 arr[left, ..., middle] 和 arr[middle+1, ..., right]。temp是辅助数组。
  private static void mergeSortedArray(int arr[], int temp[], int left, int middle, int right){
          int i=left;     
          int j=middle+1;
          int k=0;
    // 这里的核心就是合并两个有序数组，谁大放进新的数组
          while ( i<=middle && j<=right){
              if (arr[i] <=arr[j]){
                  temp[k++] = arr[i++];
              }
              else{
                  temp[k++] = arr[j++];
              }
          }
    // 左边的还有剩余
          while (i <=middle){
              temp[k++] = arr[i++];
          }
    // 右边的还有剩余
          while ( j<=right){
              temp[k++] = arr[j++];
          }
    // 把数据复制回原数组
          for (i=0; i<k; ++i){
              arr[left+i] = temp[i];
          }
      }
  ```

- 希尔排序又称缩小增量排序

  ```java
  public static void shellSort(int[] arr){
      int temp;
      for (int delta = arr.length/2; delta>=1; delta/=2){                              //对每个增量进行一次排序
          for (int i=delta; i<arr.length; i++){             
              for (int j=i; j>=delta && arr[j]<arr[j-delta]; j-=delta){ //注意每个地方增量和差值都是delta
                  temp = arr[j-delta];
                  arr[j-delta] = arr[j];
                  arr[j] = temp;
              }
          }//loop i
      }//loop delta
  }
  ```

- 堆排序 http://flyingcat2013.blog.51cto.com/7061638/1283090

  - 如何初始化一个堆
  - 排序过程中如何去调整堆
  - 堆排序存在大量的筛选和移动过程，属于不稳定的排序算法。
  - 适用于解决前 n 大的算法

  ```java
  public class ArrayHeap {
    	// 待排序的数组
      private int[] array;
      public ArrayHeap(int[] arr) {
          this.array = arr;
      }
      private int getParentIndex(int child) {
          return (child - 1) / 2;
      }
      private int getLeftChildIndex(int parent) {
          return 2 * parent + 1;
      }
      /**
       * 初始化一个大根堆。
       */
      private void initHeap() {
          int last = array.length - 1;
          for (int i = getParentIndex(last); i >= 0; --i) { // 从最后一个非叶子结点开始筛选
              int k = i;
              int j = getLeftChildIndex(k);
              while (j <= last) {
                  if (j < last) {                    
                      if (array[j] <= array[j + 1]) { // 右子节点更大
                          j++;
                      }
                  }
                  if (array[k] > array[j]) {           //父节点大于子节点中较大者，已经找到最终位置
                      break; // 停止筛选
                  } else {
                      swap(k, j);
                      k = j; // 继续筛选
                  }
                  j = getLeftChildIndex(k);
              }// loop while
          }// loop i
      }
      /**
       * 调整堆。
       */
      private void adjustHeap(int lastIndex) {
          int k = 0;
          while (k <= getParentIndex(lastIndex)) {
              int j = getLeftChildIndex(k);
              if (j < lastIndex) {
                  if (array[j] < array[j + 1]) {
                      j++;
                  }
              }
              if (array[k] < array[j]) {
                  swap(k, j);
                  k = j; // 继续筛选
              } else {
                  break; // 停止筛选
              }
          }
      }
      /**
       * 堆排序。
       * */
      public void sort() {
          initHeap();
          int last = array.length - 1;
          while (last > 0) {
              swap(0, last);
              last--;
              if (last > 0) { // 这里如果不判断，将造成最终前两个元素逆序。
                  adjustHeap(last);
              }
          }
      }
      private void swap(int i, int j) {
          int temp = array[i];
          array[i] = array[j];
          array[j] = temp;
      }
  }
  ```

- 选择排序

  设现在要给数组arr[]排序，它有n个元素。

  - 对第一个元素（Java中，下标为0）和第二个元素进行比较，如果前者大于后者，那么它一定不是最小的，**但是我们并不像冒泡排序一样急着交换**。我们可以设置一个临时变量a，存储这个目前最小的元素的下标。然后我们把这个目前最小的元素继续和第三个元素做比较，如果它仍不是最小的，那么，我们再修改a的值。如此直到和最后一个元素比较完，可以肯定a存储的一定是最小的元素的下标。
  - 如果a的值不为0（初始值，即第一个元素的下标），交换下标为a和0的两个元素。
  - 重复上述过程，这次从下标为1的元素开始比较，因为下标为0的位置已经放好了最小的元素了。
  - 如此直到只剩下最后一个元素，可以肯定这个元素就是最大的了。

  ```java
  public static void selectionSort(int[] arr) {
          int temp, min = 0;
          for (int index = 0; index < arr.length - 1; ++index) {
              min = index;
              // 循环查找最小值
              for (int j = index + 1; j < arr.length; ++j) {
                 // 如果比当前的小，就交换，不是的话就算了，继续
                  if (arr[min] > arr[j]) {
                      min = j;
                  }
              }
              // 把最小的与 index 位置元素交换
              if (min != index) {
                  temp = arr[index];
                  arr[index] = arr[min];
                  arr[min] = temp;
              }
          }
      }
  ```

- 冒泡排序 http://flyingcat2013.blog.51cto.com/7061638/1279334

  设现在要给数组arr[]排序，它有n个元素。

  1.如果n=1：显然不用排了。

  2.如果n>1：

  1)我们从第一个元素开始，把每两个相邻元素进行比较，如果前面的元素比后面的大，那么在最后的结果里面前者肯定排在后面。所以，我们把这两个元素交换。然后进行下两个相邻的元素的比较。如此直到最后一对元素比较完毕，则第一轮排序完成。可以肯定，最后一个元素一定是数组中最大的（因为每次都把相对大的放到后面了）。

  2)重复上述过程，这次我们无需考虑最后一个，因为它已经排好了。

  3)如此直到只剩一个元素，这个元素一定是最小的，那么我们的排序可以结束了。显然，进行了n-1次排序。

  上述过程中，每次（或者叫做“轮”）排序都会有一个数从某个位置慢慢“浮动”到最终的位置（画个示意图，把数组画成竖直的就可以看出来），就像冒泡一样，所以，它被称为“冒泡排序法”。

  ```java
  public static void bubbleSort(int[] arr) {
      int temp = 0;
      for (int i = arr.length - 1; i > 0; --i) { // 每次需要排序的长度
          for (int j = 0; j < i; ++j) { // 从第一个元素到第i个元素
              if (arr[j] > arr[j + 1]) {
                  temp = arr[j];
                  arr[j] = arr[j + 1];
                  arr[j + 1] = temp;
              }
          }//loop j
      }//loop i
  }// method bubbleSort
  ```

- 快速排序

  - 快排的思想很简单，从左到右依次往中间前进
  - 复杂度要记住为 O(n logn)

  ```java
  public static void quickSort(int[] arr){
      qsort(arr, 0, arr.length-1);
  }
  private static void qsort(int[] arr, int low, int high){
      if (low < high){
          int pivot=partition(arr, low, high);        //将数组分为两部分
          qsort(arr, low, pivot-1);                   //递归排序左子数组
          qsort(arr, pivot+1, high);                  //递归排序右子数组
      }
  }
  private static int partition(int[] arr, int low, int high){
      int pivot = arr[low];     //枢轴记录
      while (low<high){
          while (low<high && arr[high]>=pivot) --high;
          arr[low]=arr[high];             //交换比枢轴小的记录到左端
          while (low<high && arr[low]<=pivot) ++low;
          arr[high] = arr[low];           //交换比枢轴小的记录到右端
      }
      //扫描完成，枢轴到位,注意这时候 low 已经前进到中间的位置了
      arr[low] = pivot;
      //返回的是枢轴的位置
      return low;
  }
  ```

- 插入排序

  - 把待排序的数组分成已排序和未排序两部分，初始的时候把第一个元素认为是已排好序的。
  - 从第二个元素开始，在已排好序的子数组中寻找到该元素合适的位置并插入该位置。
  - 重复上述过程直到最后一个元素被插入有序子数组中。
  - 排序完成。

  ```java
  public static void insertionSort(int[] arr){
    // 注意 i = 1 开始排序的，即从第二个元素开始
      for (int i=1; i<arr.length; ++i){
         // 每次拿的未排序的数组中的一个元素，来与前面的比较
          int value = arr[i];
          int position;
          // 依次与前面的每个元素进行比较
          for (position=i; position>0 ; --position){
              // 如果比前面的元素小，前面的元素就自动向后面退一位
              if (arr[position-1] > value){
                  arr[position]=arr[position-1];
              }
              // 如果比前面的元素大，前面的元素不用动，这次循环结束
              else{
                  break;
              }
          }
          // 将未排序的元素插入到指定的位置
          arr[position] = value;
      }
  }
  ```