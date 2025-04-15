---
layout:     post
title:      "10 sort algorithm"
subtitle:   "sort algorithm"
date:       2025-04-15 12:00:00
author:     "细胞膜"
catalog: true
tags:
    - DSA
    - C++
---

# 10 sort algorithm

Here is the 10 most commen sort algorithm code by C++.

```
#include <iostream>
#include <vector>
#include <algorithm>

void bubbleSort(std::vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; ++i) {
        for (int j = 0; j < n - 1 - i; ++j) {
            if (arr[j] > arr[j + 1]) {
                std::swap(arr[j], arr[j + 1]);
            }
        }
    }
}

void selectionSort(std::vector<int>& arr) {
    int n = arr.size();
    for (int i = 0;i < n - 1;i++) {
        int minin = i;
        for (int j = i;j < n;j++) {
            if (arr[minin] > arr[j]) {
                minin = j;
            }
        }
        if (minin != i) {
            std::swap(arr[minin], arr[i]);
        }
    }
}

void insertionSort(std::vector<int>& arr) {
    int n = arr.size();
    for (int i = 1;i < n;i++) {
        int val = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > val) {
            arr[j+1] = arr[j];
            j--;
        }
        arr[j + 1] = val;
    }
}

void shellSort(std::vector<int>& arr) {
    int n = arr.size();
    for (int gap = n / 2;gap > 0;gap /= 2) {
        for (int i = gap;i < n;i++) {
            int temp = arr[i];
            int j = i;
            while (j >= gap && arr[j - gap] > temp) {
                arr[j] = arr[j - gap];
                j -= gap;
            }
            arr[j] = temp;
        }
    }
}

void merge(std::vector<int>& arr, int left, int mid, int right) {
    std::vector<int> temp(right - left + 1);
    int i = left;
    int j = mid + 1;
    int k = 0;
    while (i <= mid && j <= right) {
        if (arr[i] < arr[j]) {
            temp[k++] = arr[i++];
        }
        else {
            temp[k++] = arr[j++];
        }
    }
    while (i <= mid)
        temp[k++] = arr[i++];
    while (j <= right)
        temp[k++] = arr[j++];
    for (int p = 0;p < temp.size();p++) {
        arr[left + p] = temp[p];
    }
}

void mergesort(std::vector<int>& arr, int left, int right) {
    if (left >= right) {
        return;
    }
    int mid = left + (right - left) / 2;
    mergesort(arr, left, mid);
    mergesort(arr, mid + 1, right);
    merge(arr, left, mid, right);
}

void mergeSort(std::vector<int>& arr) {
    int n = arr.size();
    mergesort(arr, 0, n - 1);
}

int findmid(std::vector<int>& arr, int left, int right) {
    int temp = arr[left];
    int i = right + 1;
    for (int j = right;j > left;j--) {
        if (arr[j] > temp) {
            i--;
            std::swap(arr[j], arr[i]);
        }
    }
    std::swap(arr[i - 1], arr[left]);
    return i - 1;
}

void quicksort(std::vector<int>& arr, int left, int right) {
    if (left >= right)
        return;
    int mid = findmid(arr, left, right);
    quicksort(arr, left, mid-1);
    quicksort(arr, mid + 1, right);
}

void quickSort(std::vector<int>& arr) {
    int n = arr.size();
    int mid = findmid(arr, 0, n - 1);
    quicksort(arr, 0, mid - 1);
    quicksort(arr, mid + 1, n - 1);
}

void buildheap(std::vector<int>& arr, int i, int n) {
    int largest = i;
    int left = 2 * i + 1;
    int right = 2 * i + 2;
    if (left<n && arr[left] > arr[largest])
        largest = left;
    if (right<n && arr[right] > arr[largest])
        largest = right;
    if (largest != i) {
        std::swap(arr[i], arr[largest]);
        buildheap(arr, largest, n);
    }
}

void heapSort(std::vector<int>& arr) {
    int n = arr.size();
    for (int i = n / 2 - 1;i >= 0;i--)
        buildheap(arr, i,n);
    for (int j = n - 1;j > 0;j--) {
        std::swap(arr[0], arr[j]);
        buildheap(arr, 0, j);
    }
}

void countingSort(std::vector<int>& arr) {
    int n = arr.size();
    int maxval = 0;
    for (int i = 0;i < n;i++) {
        if (maxval < arr[i])
            maxval = arr[i];
    }
    std::vector<int> vec(maxval + 1, 0);
    for (int i = 0;i < n;i++) {
        vec[arr[i]]++;
    }
    int index = 0;
    for (int j = 0;j < vec.size();j++) {
        while (vec[j]-- > 0) {
            arr[index++] = j;
        }
    }
}

void bucketSort(std::vector<int>& arr) {
    int bucketSize = arr.size();
    if (arr.empty()) return;
    int minVal = *std::min_element(arr.begin(), arr.end());
    int maxVal = *std::max_element(arr.begin(), arr.end());
    int bucketCount = (maxVal - minVal) / bucketSize + 1;
    std::vector<std::vector<int>> buckets(bucketCount);
    for (int num : arr) {
        int index = (num - minVal) / bucketSize;
        buckets[index].push_back(num);
    }
    arr.clear();
    for (auto& bucket : buckets) {
        std::sort(bucket.begin(), bucket.end());
        arr.insert(arr.end(), bucket.begin(), bucket.end());
    }
}

void radixSort(std::vector<int>& arr) {
    if (arr.empty()) return;
    int maxVal = *std::max_element(arr.begin(), arr.end());
    int exp = 1;
    while (maxVal / exp > 0) {
        std::vector<int> output(arr.size());
        std::vector<int> count(10, 0);
        for (int num : arr)
            count[(num / exp) % 10]++;
        for (int i = 1; i < 10; i++)
            count[i] += count[i - 1];
        for (int i = arr.size() - 1; i >= 0; i--) {
            int digit = (arr[i] / exp) % 10;
            output[--count[digit]] = arr[i];
        }
        arr = output;
        exp *= 10;
    }
}

int main() {
    std::vector<int> data = { 64, 34, 25, 12, 22, 11, 90 };
    std::cout << "排序前: ";
    for (int num : data)
        std::cout << num << " ";
    std::cout << std::endl;

    //bubbleSort(data);
    //selectionSort(data);
    //insertionSort(data);
    //shellSort(data);
    //mergeSort(data);
    //quickSort(data);
    //heapSort(data);
    //countingSort(data);
    //bucketSort(data);
    //radixSort(data);

    std::cout << "排序后: ";
    for (int num : data)
        std::cout << num << " ";
    std::cout << std::endl;
    return 0;
}
```

 