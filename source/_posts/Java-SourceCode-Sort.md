---
title: Java源码阅读-DualPivotQuicksort
tags:
  - Java
  - 源码阅读
  - Sort
category: Technology
toc: true
date: 2017-01-06 22:15:52
---

&emsp;&emsp;之前一直对JDK的排序算法很好奇，抽了点时间看了下源码，收获良多。其核心思想在于，根据数据的不同特性，动态的采用不同的排序算法，充分利用各种排序算法的优势，以期达到更好的综合效果。这篇文章只是Sort相关的源码一部分内容，之后会继续深入探究其他部分的内容。
# 基本流程：

* 如果长度小于`QUICKSORT_THRESHOLD(286)`，则采用非归并排序
    * 如果长度小于`INSERTION_SORT_THRESHOLD(47)`，则采用插入排序
        - 最左区间（以初始left开始的区间） `leftmost`：普通插入排序
        - 否则：`pair insertion sort`
    * 否则，快速排序
        - 将数组划分为7段（大约），然后找出第2、3、4、5、6段的右端点对应的位置
        - 对这5个位置上的数字进行插入排序，作为枢轴的候选
        - 如果5个数都不相等
            + 选取排序后的2、4作为枢轴，进行双枢轴排序`Dual-Pivot Quicksort`
            + 效果：
            
            ```  
                left part         center part                  right part
                +----------------------------------------------------------+
                | == pivot1 |  pivot1 < && < pivot2  |    ?    | == pivot2 |
                +----------------------------------------------------------+
            ```
            + 排序后如果，中间部分元素过多，可能原因是等于pivort1和等于pivort2的元素过多，则将其调整为：
            
            ```
               left part         center part                  right part
               +----------------------------------------------------------+
               | == pivot1 |  pivot1 < && < pivot2  |    ?    | == pivot2 |
               +----------------------------------------------------------+
            ```
        - 否则，进行普通快速排序，枢轴排序后为第3个元素
* 否则，考虑Timsort(归并排序的优化版本，对一会升序、一会降序的混合情况处理比较好)
    - 创建Timsort run数组，大小为`MAX_RUN_COUNT(67) + 1`
        + a[run[i]] ~ a[run[i + 1]]之间为升序数组
        + 检查当前待排序数组是否适合使用Timsort,即run数组中升序数组个数，如果个数不小于`MAX_RUN_COUNT`则认为数组内元素排序比较混乱，适合非归并排序
            * 注：对于连续下降的元素会将其调整为连续上升
    - 如果通过上述检测，则进行归并排序      

# 代码详解：

* 普通的插入排序
```java
for (int i = left, j = i; i < right; j = ++i) {
    int ai = a[i + 1];      // 带插入元素
    while (ai < a[j]) {     // 寻找插入位置
        a[j + 1] = a[j];
        if (j-- == left) {
            break;
        }
    }
    a[j + 1] = ai;          // 插入新元素
}
```
* 改进的插入排序：pair insertion sort,每次插入两个元素
    - pair insertion之所以不使用在做区间的原因，如果将其应用在左区间，需要增加额外的边界控制。但为什么没有左边界没有使用这种改良的插入排序呢？这一点还需要探究
```java
/**
 * 注：left左侧的内容均已排好序，默认的前提条件
 */
/*
 * Skip the longest ascending sequence.
 */
do {
    if (left >= right) {
        return;
    }
} while (a[++left] >= a[left - 1]);

/*
 * Every element from adjoining part plays the role
 * of sentinel, therefore this allows us to avoid the
 * left range check on each iteration. Moreover, we use
 * the more optimized algorithm, so called pair insertion
 * sort, which is faster (in the context of Quicksort)
 * than traditional implementation of insertion sort.
 * 一次遍历插入两个元素
 */
for (int k = left; ++left <= right; k = ++left) {
    int a1 = a[k], a2 = a[left];
    
    // 使得 a1 >= a2
    if (a1 < a2) {
        a2 = a1; a1 = a[left];
    }

    // 寻找a1的插入位置，相隔距离为2
    while (a1 < a[--k]) {
        a[k + 2] = a[k];
    }
    a[++k + 1] = a1;

    // 寻找a2的插入位置，相隔距离为1
    while (a2 < a[--k]) {
        a[k + 1] = a[k];
    }
    a[k + 1] = a2;
}

// 将最后一个位置插入到合适位置
int last = a[right];

while (last < a[--right]) {
    a[right + 1] = a[right];
}
a[right + 1] = last;
```
* 双枢轴排序`Dual-Pivot Quicksort`
```java
// Inexpensive approximation of length / 7
int seventh = (length >> 3) + (length >> 6) + 1;

/*
 * Sort five evenly spaced elements around (and including) the
 * center element in the range. These elements will be used for
 * pivot selection as described below. The choice for spacing
 * these elements was empirically determined to work well on
 * a wide variety of inputs.
 */
int e3 = (left + right) >>> 1; // The midpoint
int e2 = e3 - seventh;
int e1 = e2 - seventh;
int e4 = e3 + seventh;
int e5 = e4 + seventh;

// Sort these elements using insertion sort
if (a[e2] < a[e1]) { long t = a[e2]; a[e2] = a[e1]; a[e1] = t; }

if (a[e3] < a[e2]) { long t = a[e3]; a[e3] = a[e2]; a[e2] = t;
    if (t < a[e1]) { a[e2] = a[e1]; a[e1] = t; }
}
if (a[e4] < a[e3]) { long t = a[e4]; a[e4] = a[e3]; a[e3] = t;
    if (t < a[e2]) { a[e3] = a[e2]; a[e2] = t;
        if (t < a[e1]) { a[e2] = a[e1]; a[e1] = t; }
    }
}
if (a[e5] < a[e4]) { long t = a[e5]; a[e5] = a[e4]; a[e4] = t;
    if (t < a[e3]) { a[e4] = a[e3]; a[e3] = t;
        if (t < a[e2]) { a[e3] = a[e2]; a[e2] = t;
            if (t < a[e1]) { a[e2] = a[e1]; a[e1] = t; }
        }
    }
}

// Pointers
int less  = left;  // The index of the first element of center part
int great = right; // The index before the first element of right part

if (a[e1] != a[e2] && a[e2] != a[e3] && a[e3] != a[e4] && a[e4] != a[e5]) {
    /*
     * Use the second and fourth of the five sorted elements as pivots.
     * These values are inexpensive approximations of the first and
     * second terciles of the array. Note that pivot1 <= pivot2.
     */
    long pivot1 = a[e2];
    long pivot2 = a[e4];

    /*
     * The first and the last elements to be sorted are moved to the
     * locations formerly occupied by the pivots. When partitioning
     * is complete, the pivots are swapped back into their final
     * positions, and excluded from subsequent sorting.
     */
    a[e2] = a[left];
    a[e4] = a[right];

    /*
     * Skip elements, which are less or greater than pivot values.
     */
    while (a[++less] < pivot1);
    while (a[--great] > pivot2);

    /*
     * Partitioning:
     *
     *   left part           center part                   right part
     * +--------------------------------------------------------------+
     * |  < pivot1  |  pivot1 <= && <= pivot2  |    ?    |  > pivot2  |
     * +--------------------------------------------------------------+
     *               ^                          ^       ^
     *               |                          |       |
     *              less                        k     great
     *
     * Invariants:
     *
     *              all in (left, less)   < pivot1
     *    pivot1 <= all in [less, k)     <= pivot2
     *              all in (great, right) > pivot2
     *
     * Pointer k is the first index of ?-part.
     */
    outer:
    for (int k = less - 1; ++k <= great; ) {
        long ak = a[k];
        if (ak < pivot1) { // Move a[k] to left part
            a[k] = a[less];
            /*
             - Here and below we use "a[i] = b; i++;" instead
             - of "a[i++] = b;" due to performance issue.
             */
            a[less] = ak;
            ++less;
        } else if (ak > pivot2) { // Move a[k] to right part
            while (a[great] > pivot2) {
                if (great-- == k) {
                    break outer;
                }
            }
            if (a[great] < pivot1) { // a[great] <= pivot2
                a[k] = a[less];
                a[less] = a[great];
                ++less;
            } else { // pivot1 <= a[great] <= pivot2
                a[k] = a[great];
            }
            /*
             - Here and below we use "a[i] = b; i--;" instead
             - of "a[i--] = b;" due to performance issue.
             */
            a[great] = ak;
            --great;
        }
    }

    // Swap pivots into their final positions
    a[left]  = a[less  - 1]; a[less  - 1] = pivot1;
    a[right] = a[great + 1]; a[great + 1] = pivot2;

    // Sort left and right parts recursively, excluding known pivots
    sort(a, left, less - 2, leftmost);
    sort(a, great + 2, right, false);

    /*
     * If center part is too large (comprises > 4/7 of the array),
     * swap internal pivot values to ends.
     */
    if (less < e1 && e5 < great) {
        /*
         * Skip elements, which are equal to pivot values.
         */
        while (a[less] == pivot1) {
            ++less;
        }

        while (a[great] == pivot2) {
            --great;
        }

        /*
         * Partitioning:
         *
         *   left part         center part                  right part
         * +----------------------------------------------------------+
         * | == pivot1 |  pivot1 < && < pivot2  |    ?    | == pivot2 |
         * +----------------------------------------------------------+
         *              ^                        ^       ^
         *              |                        |       |
         *             less                      k     great
         *
         * Invariants:
         *
         *              all in (*,  less) == pivot1
         *     pivot1 < all in [less,  k)  < pivot2
         *              all in (great, *) == pivot2
         *
         * Pointer k is the first index of ?-part.
         */
        outer:
        for (int k = less - 1; ++k <= great; ) {
            long ak = a[k];
            if (ak == pivot1) { // Move a[k] to left part
                a[k] = a[less];
                a[less] = ak;
                ++less;
            } else if (ak == pivot2) { // Move a[k] to right part
                while (a[great] == pivot2) {
                    if (great-- == k) {
                        break outer;
                    }
                }
                if (a[great] == pivot1) { // a[great] < pivot2
                    a[k] = a[less];
                    /*
                     * Even though a[great] equals to pivot1, the
                     * assignment a[less] = pivot1 may be incorrect,
                     * if a[great] and pivot1 are floating-point zeros
                     * of different signs. Therefore in float and
                     * double sorting methods we have to use more
                     * accurate assignment a[less] = a[great].
                     */
                    a[less] = pivot1;
                    ++less;
                } else { // pivot1 < a[great] < pivot2
                    a[k] = a[great];
                }
                a[great] = ak;
                --great;
            }
        }
    }

    // Sort center part recursively
    sort(a, less, great, false);

} else { // Partitioning with one pivot
    /*
     * Use the third of the five sorted elements as pivot.
     * This value is inexpensive approximation of the median.
     */
    long pivot = a[e3];

    /*
     * Partitioning degenerates to the traditional 3-way
     * (or "Dutch National Flag") schema:
     *
     *   left part    center part              right part
     * +-------------------------------------------------+
     * |  < pivot  |   == pivot   |     ?    |  > pivot  |
     * +-------------------------------------------------+
     *              ^              ^        ^
     *              |              |        |
     *             less            k      great
     *
     * Invariants:
     *
     *   all in (left, less)   < pivot
     *   all in [less, k)     == pivot
     *   all in (great, right) > pivot
     *
     * Pointer k is the first index of ?-part.
     */
    for (int k = less; k <= great; ++k) {
        if (a[k] == pivot) {
            continue;
        }
        long ak = a[k];
        if (ak < pivot) { // Move a[k] to left part
            a[k] = a[less];
            a[less] = ak;
            ++less;
        } else { // a[k] > pivot - Move a[k] to right part
            while (a[great] > pivot) {
                --great;
            }
            if (a[great] < pivot) { // a[great] <= pivot
                a[k] = a[less];
                a[less] = a[great];
                ++less;
            } else { // a[great] == pivot
                /*
                 * Even though a[great] equals to pivot, the
                 * assignment a[k] = pivot may be incorrect,
                 * if a[great] and pivot are floating-point
                 * zeros of different signs. Therefore in float
                 * and double sorting methods we have to use
                 * more accurate assignment a[k] = a[great].
                 */
                a[k] = pivot;
            }
            a[great] = ak;
            --great;
        }
    }

    /*
     * Sort left and right parts recursively.
     * All elements from center part are equal
     * and, therefore, already sorted.
     */
    sort(a, left, less - 1, leftmost);
    sort(a, great + 1, right, false);
```

* Timsort归并排序
```java
/*
 * Index run[i] is the start of i-th run
 * (ascending or descending sequence).
 */
int[] run = new int[MAX_RUN_COUNT + 1];
int count = 0; run[0] = left;

// Check if the array is nearly sorted
for (int k = left; k < right; run[count] = k) {
    if (a[k] < a[k + 1]) { // ascending
        while (++k <= right && a[k - 1] <= a[k]);
    } else if (a[k] > a[k + 1]) { // descending
        // 将降序数组变为升序
        while (++k <= right && a[k - 1] >= a[k]);
        for (int lo = run[count] - 1, hi = k; ++lo < --hi; ) {
            int t = a[lo]; a[lo] = a[hi]; a[hi] = t;
        }
    } else { // equal
        for (int m = MAX_RUN_LENGTH; ++k <= right && a[k - 1] == a[k]; ) {
            if (--m == 0) {
                sort(a, left, right, true);
                return;
            }
        }
    }

    /*
     * The array is not highly structured,
     * use Quicksort instead of merge sort.
     */
    if (++count == MAX_RUN_COUNT) {
        sort(a, left, right, true);
        return;
    }
}

// Check special cases
// Implementation note: variable "right" is increased by 1.
if (run[count] == right++) { // The last run contains one element
    run[++count] = right;
} else if (count == 1) { // The array is already sorted
    return;
}

// Determine alternation base for merge
// 确定归并排序的迭代次数（每迭代一次，将相邻升序子序列合并，即run内元素数目减半）
// 简单示例：a[1, 5, 2, 6, 3, 7, 4, 8] ==> a[1, 2, 5, 6, 3, 4, 7, 8]
byte odd = 0;
for (int n = 1; (n <<= 1) < count; odd ^= 1);

// Use or create temporary array b for merging
int[] b;                 // temp array; alternates with a
int ao, bo;              // array offsets from 'left'
int blen = right - left; // space needed for b
if (work == null || workLen < blen || workBase + blen > work.length) {
    work = new int[blen];
    workBase = 0;
}
// 根据归并的迭代次数，更改a,b
// a,b中必有一个数组为原始数组，另一个为临时数组
// 在归并的过程中，每迭代一次，run内元素数目减半，同时a,b会交换一次
// 为了保证最后一次迭代后，原始数组内存有归并好的数据，需要进行如下考虑
if (odd == 0) {
    System.arraycopy(a, left, work, workBase, blen);
    b = a;
    bo = 0;
    a = work;
    ao = workBase - left;
} else {
    b = work;
    ao = 0;
    bo = workBase - left;
}

// Merging
// a是原始数组，b是目标数组
for (int last; count > 1; count = last) {
    // 合并两个相邻升序序列
    for (int k = (last = 0) + 2; k <= count; k += 2) {
        // 确定边界
        int hi = run[k], mi = run[k - 1];
        for (int i = run[k - 2], p = i, q = mi; i < hi; ++i) {
            if (q >= hi || p < mi && a[p + ao] <= a[q + ao]) {
                b[i + bo] = a[p++ + ao];
            } else {
                b[i + bo] = a[q++ + ao];
            }
        }
        // 更新子序列标示
        run[++last] = hi;
    }
    // 如果升序子序列个数为奇数，之前两两合并时，最后会剩余一个，将剩余的直接拷贝到b, 等待下一次合并
    if ((count & 1) != 0) {
        for (int i = right, lo = run[count - 1]; --i >= lo;
            b[i + bo] = a[i + ao]
        );
        run[++last] = right;
    }

    // 交换a b
    int[] t = a; a = b; b = t;
    int o = ao; ao = bo; bo = o;
}
```

Reference:

* [形式化方法的逆袭——如何找出Timsort算法和玉兔月球车中的Bug？
](http://bindog.github.io/blog/2015/03/30/use-formal-method-to-find-the-bug-in-timsort-and-lunar-rover)
* [排序算法---快速排序（JDK1.7 DualPivotQuicksort 源码解析）](http://lib.csdn.net/article/datastructure/9282)
* [JDK源码解析(1)——数据数组排序：Arrays.sort()](http://www.voidcn.com/blog/octopusflying/article/p-6181740.html)
