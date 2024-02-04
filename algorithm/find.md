---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 排序算法

## **常见排序算法**

* [x] 冒泡排序 通过成对比较元素并交换，直到列表排序完成。
* [x] 选择排序 在冒泡排序的基础上改进，每遍历列表一次只交换一次。
* [ ] 插入排序 一次一个元素地构建最终排序的数组，假设第一个元素已经排好序。
* [ ] 归并排序 将未排序的列表分割成n个子列表，每个子列表包含一个元素，然后反复合并子列表直到只剩下一个子列表。
* [x] 快速排序 从数组中选择一个'基准'元素，并将其他元素分成两个子数组，根据它们是大于还是小于基准。
* [ ] 堆排序 将输入列表转换成堆，然后重复地从堆中提取最大元素并重建堆。
* [ ] 基数排序 是一种非比较整数排序算法，通过将键按各个位上共享相同的重要位置和值进行分组来排序。
* [ ] 希尔排序 是一种内部比较排序，它通过首先对相距较远的元素对进行排序，然后逐步减少要比较的元素之间的间隔。

## 冒泡排序

<figure><img src="../.gitbook/assets/冒泡排序" alt=""><figcaption><p>冒泡</p></figcaption></figure>

```java
public class BubbleSort {
    public static void bubbleSort(int[] arr) {
        if (arr == null || arr.length == 0) {
            return;
        }
        boolean swapped;
        for (int i = 0; i < arr.length - 1; i++) {
            swapped = false;
            for (int j = 0; j < arr.length - 1 - i; j++) {
                if (arr[j] > arr[j + 1]) {
                    int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                    swapped = true;
                }
            }
            // 如果没有发生交换,说明已经排好序,避免不必要数组遍历
            if (!swapped) {
                break;
            }
        }
    }
}
```

## 选择排序

<figure><img src="../.gitbook/assets/选择排序" alt=""><figcaption><p>选择排序</p></figcaption></figure>

```java
public class SelectionSort {
    public static void selectionSort(int[] arr) {
        // 数组为空或长度为0，直接返回
        if (arr == null || arr.length == 0) {
            return;
        }
        for (int i = 0; i < arr.length - 1; i++) {
            // 假设当前位置就是最小元素位置
            int minIndex = i;
            for (int j = i + 1; j < arr.length; j++) {
                // 寻找实际的最小元素位置
                if (arr[j] < arr[minIndex]) {
                    minIndex = j;
                }
            }
            // 如果最小元素位置不是当前位置，交换它们
            if (minIndex != i) {
                int temp = arr[i];
                arr[i] = arr[minIndex];
                arr[minIndex] = temp;
            }
        }
    }
}
```

## 快速排序

<figure><img src="../.gitbook/assets/快速排序" alt=""><figcaption><p>快速排序-图中以第一个元素为基准</p></figcaption></figure>

```java
public class QuickSort {
    public static void quickSort(int[] arr, int low, int high) {
        if (low < high) {
            // 找到划分点
            int pivotIndex = partition(arr, low, high);
            // 递归排序左侧
            quickSort(arr, low, pivotIndex - 1);
            // 递归排序右侧
            quickSort(arr, pivotIndex + 1, high);
        }
    }

    private static int partition(int[] arr, int low, int high) {
        // 选择最后一个元素作为基准
        int pivot = arr[high];
        // 初始化小于基准的指针
        int i = (low - 1);
        for (int j = low; j < high; j++) {
            // 如果当前元素小于基准
            if (arr[j] < pivot) {
                // 移动指针
                i++;
                // 交换元素
                int temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }
        }
        // 将基准放到正确的位置
        int temp = arr[i + 1];
        arr[i + 1] = arr[high];
        arr[high] = temp;
        // 返回基准的位置
        return i + 1;
    }
}
```
