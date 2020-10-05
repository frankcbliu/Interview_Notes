## 工具函数

```java
// 交换 a[i] 和 a[j]
public static void swap(int[] arr, int i, int j) {
    if (i == j) return;
    arr[i] = arr[i] ^ arr[j];
    arr[j] = arr[i] ^ arr[j];
    arr[i] = arr[i] ^ arr[j];
}
```



## 1. 冒泡排序【重要】

> 时间复杂度`O(N^2)`，额外空间复杂度`O(1) `

```java
public static void bubbleSort(int[] arr) {
    for (int i = 0; i < arr.length; i++) {
        for (int j = 0; j < arr.length - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr, j, j + 1);
            }
        }
    }
}
```

## 2. 选择排序

> 时间复杂度`O(N^2)`，额外空间复杂度`O(1) `

```java
public static void selectSort(int[] arr) {
    for (int i = 0; i < arr.length; i++) {
        int minIndex = i;
        for (int j = i + 1; j < arr.length; j++) {
            minIndex = arr[minIndex] > arr[j] ? j : minIndex;
        }
        swap(arr, i, minIndex);
    }
}
```

## 3. 插入排序

> 时间复杂度`O(N^2)`，额外空间复杂度`O(1)` 

```java
public static void insertSort(int[] arr) {
    for (int i = 1; i < arr.length; i++) {
        for (int j = i-1; j >=0 && arr[j] > arr[j+1] ; j--) {
            swap(arr, j, j+1);
        }
    }
}
```

## 4. 归并排序【重要】

> 时间复杂度`O(N*logN)`，额外空间复杂度`O(N) `

```java
private static void merge(int[] arr, int l, int mid, int r) {
    int[] help = new int[r - l + 1];
    int i = 0;
    int p1 = l;
    int p2 = mid + 1;
    while (p1 <= mid && p2 <= r) {
        help[i++] = arr[p1] > arr[p2] ? arr[p2++] : arr[p1++];
    }

    while (p1 <= mid) {
        help[i++] = arr[p1++];
    }

    while (p2 <= r) {
        help[i++] = arr[p2++];
    }

    for (int j = 0; j < help.length; j++) { // 可用 System.arraycopy 代替
        arr[l + j] = help[j];
    }
}

private static void mergeSort(int[] arr, int l, int r) {
    if (l < r) {
//      int mid = l + ((r - l) >> 1); // 注意位运算的优先级
        int mid = l + (r - l) / 2;
        mergeSort(arr, l, mid);
        mergeSort(arr, mid + 1, r);
        merge(arr, l, mid, r);
    }
}

public static void mergeSort(int[] arr) {
    mergeSort(arr, 0, arr.length - 1);
}
```

## 5. 快速排序【重要】

随机快速排序的复杂度分析 

> 时间复杂度O(N*logN)，额外空间复杂度O(logN) 

```java
public static void quickSort(int[] arr) {
    quickSort(arr, 0, arr.length - 1);
}

private static void quickSort(int[] arr, int l, int r) {
    if (l < r) {
        // Math.random() [0,1) --> [0, r-l] + l --> [l, r]
        int random = (int) (Math.random() * (r - l + 1)) + l;
        swap(arr, random, r);
        int[] p = partition(arr, l, r);
        quickSort(arr, l, p[0]);
        quickSort(arr, p[1], r);
    }
}

private static int[] partition(int[] arr, int l, int r) {
    int less = l - 1;
    int more = r;
    while (l < more) {
        if (arr[l] < arr[r]) {
            swap(arr, ++less, l++);
        } else if (arr[l] > arr[r]) {
            swap(arr, --more, l);
        } else {
            l++;
        }
    }
    swap(arr, more, r);
    // 此时 more 的位置就是我们的锚点位置，返回其左右边界
    // 对于 2,1,4,5,3 而言，假设选取最后一位进行比较，那么比 3 小的都在左边，比 3 大的都在右边
    // 最终排成： ... less, 3 (more), more + 1, ...
    return new int[]{less, more + 1};
}
```

## 6. 堆排序【重要】

> 时间复杂度`O(N*logN)`，额外空间复杂度`O(1) `

```java
// 堆排序
public static void heapSort(int[] arr) {
    if (arr.length < 2) return;
    // 建堆
    for (int i = 0; i < arr.length; i++) {
        heapInsert(arr, i);
    }
    // 交换最后一个
    int size = arr.length - 1;
    while (size > 0) {
        swap(arr, 0, size);
        heapify(arr, size--);
    }
}

public static void heapInsert(int[] arr, int i) { // 升序排序，最大堆
    // 插入到数组末尾，从下往上走，比父亲结点大就交换
    while (arr[i] > arr[(i - 1) / 2]) {
        swap(arr, i, (i - 1) / 2);
        i = (i - 1) / 2;
    }
}

// 将当前堆最大值取出后，重新调整成最大堆
public static void heapify(int[] arr, int size) {
    int cur = 0; // 根节点一定为 0
    int left = 2 * cur + 1; // 实际上就是 1
    while (left < size) {
        // 判断左右孩子结点哪个大
        int max = (left + 1 < size && arr[left + 1] > arr[left]) ? left + 1 : left;
        // 判断最大的孩子结点和当前结点哪个大
        max = arr[cur] > arr[max] ? cur : max;
        if (max == cur) // 如果当前结点就是最大值，那么无需继续调整，终止即可
            return;
        swap(arr, cur, max);
        cur = max;
        left = 2 * cur + 1;
    }
}
```

## 7. 希尔排序

```java
public static void shellSort(int[] arr) {
    int gap = 1;
    while (gap < arr.length) {
        gap = gap * 3 + 1;
    }

    while (gap > 0) {
        for (int i = gap; i < arr.length; i++) {
            for (int j = i - gap; j >= 0 && arr[j] > arr[j + gap]; j -= gap) {
                swap(arr, j, j + gap);
            }
        }
        gap = gap / 3;
    }
}
```



## 各大排序时间空间复杂度比较

![image-20200306135946992](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjejp83kpsj31k20ngq8c.jpg)



## 总结

标记了**【重要】**的排序方法需要熟练记忆，能手撸，尤其归并排序、快速排序，往往稍作变形就能解决某些编程题，非常重要！！！

