---
tags: [dsa, sorting, bubble-sort, merge-sort, quick-sort]
links: ["[[00_Index]]", "[[Sorting_Algorithms_Problems]]"]
---

# Sorting Algorithms -- Templates

*<- [[00_Index|Index]] · [[Sorting_Algorithms_Problems|Problems ->]]*

---

## Summary of Sorting Algorithms

| Algorithm | Best Time | Avg Time | Worst Time | Space | Stable? | Method |
|---|---|---|---|---|---|---|
| **Selection Sort** | $O(n^2)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | No | Selection |
| **Bubble Sort** | $O(n)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | Yes | Exchanging |
| **Insertion Sort**| $O(n)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | Yes | Insertion |
| **Merge Sort** | $O(n \log n)$| $O(n \log n)$| $O(n \log n)$| $O(n)$ | Yes | Divide & Conquer |
| **Quick Sort** | $O(n \log n)$| $O(n \log n)$| $O(n^2)$ | $O(\log n)$| No | Partitioning |

---

## 1. Selection Sort (Min-Finding)
- **Concept**: Repeatedly find the minimum element from the unsorted part and swap it with the first element of the unsorted part.

```cpp
#include <vector>
#include <algorithm>

void selectionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIdx]) minIdx = j;
        }
        swap(arr[i], arr[minIdx]);
    }
}
```

---

## 2. Insertion Sort (Card Deck Insertion)
- **Concept**: Insert elements from the unsorted part into their correct sorted position one by one. Excellent for nearly-sorted arrays.

```cpp
#include <vector>

void insertionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 1; i < n; i++) {
        int key = arr[i];
        int j = i - 1;
        // Shift elements of arr[0..i-1] that are greater than key
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}
```

---

## 3. Merge Sort (Divide & Conquer)
- **Concept**: Recursively split the array into halves, sort them, and merge the sorted halves using an auxiliary array.

```cpp
#include <vector>

void merge(vector<int>& arr, int l, int mid, int r) {
    int n1 = mid - l + 1;
    int n2 = r - mid;
    vector<int> L(n1), R(n2);

    for (int i = 0; i < n1; i++) L[i] = arr[l + i];
    for (int j = 0; j < n2; j++) R[j] = arr[mid + 1 + j];

    int i = 0, j = 0, k = l;
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) {
            arr[k++] = L[i++];
        } else {
            arr[k++] = R[j++];
        }
    }
    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}

void mergeSort(vector<int>& arr, int l, int r) {
    if (l >= r) return;
    int mid = l + (r - l) / 2;
    mergeSort(arr, l, mid);
    mergeSort(arr, mid + 1, r);
    merge(arr, l, mid, r);
}
```

---

## 4. Quick Sort (Partitioning)
- **Concept**: Choose a "pivot" element. Partition the array such that elements smaller than pivot go left, and larger elements go right. Recurse on left and right partitions.

```cpp
#include <vector>
#include <algorithm>

int partition(vector<int>& arr, int low, int high) {
    int pivot = arr[high]; // choose last element as pivot
    int i = low - 1; // index of smaller element

    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i + 1], arr[high]);
    return i + 1; // return pivot index
}

void quickSort(vector<int>& arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}
```

---

*<- [[00_Index]] · [[Sorting_Algorithms_Problems|Problems ->]]*
