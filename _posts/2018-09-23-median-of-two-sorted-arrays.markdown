---
layout: post
title:  "LeetCode #4 两个排序数组的中位数 Median of Two Sorted Arrays"
date:   2018-09-16 23:12:20 +0800
categories: 算法
---

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>

## LeetCode #4 两个排序数组的中位数 Median of Two Sorted Arrays

给定两个大小为 `m` 和 `n` 的有序数组 `nums1` 和 `nums2` 。

请找出这两个有序数组的中位数。要求算法的时间复杂度为 `O(log (m+n))` 。

你可以假设 `nums1` 和 `nums2` 不同时为空。

**示例 1**:

```
nums1 = [1, 3]
nums2 = [2]

中位数是 2.0
```

**示例 2**:

```
nums1 = [1, 2]
nums2 = [3, 4]

中位数是 (2 + 3)/2 = 2.5
```

### 迭代法

这个问题简化成在两个已排序数组中寻找第`k`大的值，代码如下：

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int count = nums1.length + nums2.length;
		if (count % 2 == 1) {
			return findKth(nums1, 0, nums2, 0, count / 2 + 1);
		}
		return (findKth(nums1, 0, nums2, 0, count / 2) + findKth(nums1, 0, nums2, 0, count / 2 + 1)) / 2.0;
    }

    private int findKth(int[] nums1, int i, int[] nums2, int j, int k) {
		if (nums1.length - i > nums2.length - j) {
			return findKth(nums2, j, nums1, i, k);
		}
		if (nums1.length == i) {
			return nums2[j + k - 1];
		}
		if (k == 1) {
			return Math.min(nums1[i], nums2[j]);
		}
		int a = Math.min(i + k / 2, nums1.length);
		int b = j + k - a + i;
		if (nums1[a - 1] < nums2[b - 1]) {
			return findKth(nums1, a, nums2, j, k - a + i);
		} else if (nums1[a - 1] > nums2[b - 1]) {
			return findKth(nums1, i, nums2, b, k - b + j);
		}
		return nums1[a - 1];
	}
}
```

时间复杂度`O(log(m+n))`。

### 循环法

这种方法是官方题解：

为了解决这个问题，我们需要理解“中位数的作用是什么”。在统计中，中位数被用来：

将一个集合划分为两个长度相等的子集，其中一个子集中的元素总是大于另一个子集中的元素。

如果理解了中位数的划分作用，我们就很接近答案了。

首先，让我们在任一位置 \\(i\\) 将 \\(\text{A}\\) 划分成两个部分：

          left_A             |        right_A
    A[0], A[1], ..., A[i-1]  |  A[i], A[i+1], ..., A[m-1]
由于 \\(\text{A}\\) 中有 \\(m\\) 个元素， 所以我们有 \\(m+1\\) 种划分的方法\\(（i = 0 \sim m）\\)。

我们知道：

$$
\text{len}(\text{left\_A}) = i, \text{len}(\text{right\_A}) = m - i
$$

注意：当 \\(i = 0\\) 时，\\(\text{left\_A}\\) 为空集， 而当 \\(i = m\\) 时, \\(\text{right\_A}\\) 为空集。

采用同样的方式，我们在任一位置 \\(j\\) 将 \\(\text{B}\\) 划分成两个部分：

          left_B             |        right_B
    B[0], B[1], ..., B[j-1]  |  B[j], B[j+1], ..., B[n-1]
将 \\(\text{left\_A}\\) 和 \\(\text{left\_B}\\) 放入一个集合，并将 \\(\text{right\_A}\\) 和 \\(\text{right\_B}\\) 放入另一个集合。 再把这两个新的集合分别命名为 \\(\text{left\_part}\\) 和 \\(\text{right\_part}\\)：

          left_part          |        right_part
    A[0], A[1], ..., A[i-1]  |  A[i], A[i+1], ..., A[m-1]
    B[0], B[1], ..., B[j-1]  |  B[j], B[j+1], ..., B[n-1]
如果我们可以确认：
$$
\text{len}(\text{left\_part}) = \text{len}(\text{right\_part})并且
\max(\text{left\_part}) \leq \min(\text{right\_part})
$$

那么，我们已经将 \\(\{\text{A}, \text{B}\}\\) 中的所有元素划分为相同长度的两个部分，且其中一部分中的元素总是大于另一部分中的元素。那么：

$$
\text{median} = \frac{\text{max}(\text{left}\_\text{part}) + \text{min}(\text{right}\_\text{part})}{2}
$$

要确保这两个条件，我们只需要保证：

$$
i + j = m - i + n - j（或：m - i + n - j + 1） 如果 n \geq m，只需要使 \ i = 0 \sim m,\ j = \frac{m + n + 1}{2} - i \\ ​并且
\text{B}[j-1] \leq \text{A}[i] 以及 \text{A}[i-1] \leq \text{B}[j]
$$

ps.1 为了简化分析，我假设 \\(\text{A}[i-1], \text{B}[j-1], \text{A}[i], \text{B}[j]\\) 总是存在，哪怕出现 \\(i=0，i=m，j=0\\)，或是 \\(j=n\\) 这样的临界条件。 我将在最后讨论如何处理这些临界值。

ps.2 为什么 \\(n \geq m\\)？由于\\(0 \leq i \leq m\\) 且 \\(j = \frac{m + n + 1}{2} - i\\)​	，我必须确保 \\(j\\) 不是负数。如果 \\(n &lt; m\\)，那么 \\(j\\) 将可能是负数，而这会造成错误的答案。

所以，我们需要做的是：

在 \\([0，m]\\) 中搜索并找到目标对象 \\(i\\)，以使：

$$
\qquad \text{B}[j-1] \leq \text{A}[i]  且 \ \text{A}[i-1] \leq \text{B}[j],  其中 j = \frac{m + n + 1}{2} - i$$
接着，我们可以按照以下步骤来进行二叉树搜索：

设 \\(\text{imin} = 0imin=0\\)，\\(\text{imax} = m\\), 然后开始在 \\([\text{imin}, \text{imax}][imin,imax]\\) 中进行搜索。
令 \\(i = \frac{\text{imin} + \text{imax}}{2}， j = \frac{m + n + 1}{2} - i\\)

现在我们有 \\(\text{len}(\text{left}\_\text{part})=\text{len}(\text{right}\_\text{part})\\)。 而且我们只会遇到三种情况：

\\(\text{B}[j-1] \leq \text{A}[i] 且 \text{A}[i-1] \leq \text{B}[j]\\)：
这意味着我们找到了目标对象 \\(i\\)，所以可以停止搜索。

\\(\text{B}[j-1] &gt; \text{A}[i]\\)：
这意味着 \\(\text{A}[i]\\) 太小，我们必须调整 \\(i\\) 以使 \\(\text{B}[j-1] \leq \text{A}[i]\\)。
我们可以增大 \\(i\\) 吗？
      是的，因为当 \\(i\\) 被增大的时候，\\(j\\) 就会被减小。
      因此 \\(\text{B}[j-1]\\) 会减小，而 \\(\text{A}[i]\\) 会增大，那么 \\(\text{B}[j-1] \leq \text{A}[i]\\) 就可能被满足。
我们可以减小 \\(i\\) 吗？
      不行，因为当 \\(i\\) 被减小的时候，\\(j\\) 就会被增大。
      因此 \\(\text{B}[j-1]\\) 会增大，而 \\(\text{A}[i]\\) 会减小，那么 \\(\text{B}[j-1] \leq \text{A}[i]\\) 就可能不满足。
所以我们必须增大 \\(i\\)。也就是说，我们必须将搜索范围调整为 \\([i+1, \text{imax}]\\)。 因此，设 \\(\text{imin} = i+1\\)，并转到步骤 2。

\\(\text{A}[i-1] &gt; \text{B}[j]\\)： 这意味着 \\(\text{A}[i-1]\\) 太大，我们必须减小 \\(i\\) 以使 \\(\text{A}[i-1]\leq \text{B}[j]\\)。 也就是说，我们必须将搜索范围调整为 \\([\text{imin}, i-1]\\)。
因此，设 \\(\text{imax} = i-1\\)，并转到步骤 2。

当找到目标对象 \\(i\\) 时，中位数为：

$$\max(\text{A}[i-1], \text{B}[j-1]),  当 m + n 为奇数时$$

$$\frac{\max(\text{A}[i-1], \text{B}[j-1]) + \min(\text{A}[i], \text{B}[j])}{2},  当 m + n 为偶数时$$

现在，让我们来考虑这些临界值 \\(i=0,i=m,j=0,j=n\\)，此时 \\(\text{A}[i-1],\text{B}[j-1],\text{A}[i],\text{B}[j] 可能不存在。 其实这种情况比你想象的要容易得多。\\)

我们需要做的是确保 \\(\text{max}(\text{left}\_\text{part}) \leq \text{min}(\text{right}\_\text{part})\\)。 因此，如果 \\(i\\) 和 \\(j\\) 不是临界值（这意味着 \\(\text{A}[i-1], \text{B}[j-1],\text{A}[i],\text{B}[j]\\) 全部存在）, 那么我们必须同时检查 \\(\text{B}[j-1] \leq \text{A}[i]\\) 以及 \\(\text{A}[i-1] \leq \text{B}[j]\\) 是否成立。 但是如果 \\(\text{A}[i-1],\text{B}[j-1],\text{A}[i],\text{B}[j]\\) 中部分不存在，那么我们只需要检查这两个条件中的一个（或不需要检查）。 举个例子，如果 \\(i = 0\\)，那么 \\(\text{A}[i-1]\\) 不存在，我们就不需要检查 \\(\text{A}[i-1] \leq \text{B}[j]\\) 是否成立。 所以，我们需要做的是：

在 \\([0，m]\\) 中搜索并找到目标对象 \\(i\\)，以使：

$$
(j = 0 or i = m or \text{B}[j-1] \leq \text{A}[i]) 或是 (i = 0 or j = n or \text{A}[i-1] \leq \text{B}[j]), 其中 j = \frac{m + n + 1}{2} - i​
$$

在循环搜索中，我们只会遇到三种情况：

$$
(j = 0 or i = m or \text{B}[j-1] \leq \text{A}[i])
或是
(i = 0 or j = n or \text{A}[i-1] \leq \text{B}[j])
$$

这意味着 \\(i\\) 是完美的，我们可以停止搜索。

$$
j &gt; 0 and i &lt; m and \text{B}[j - 1] &gt; \text{A}[i] 
$$

这意味着 ii 太小，我们必须增大它。
i &gt; 0i>0 and j &lt; nj<n and \text{A}[i - 1] &gt; \text{B}[j]A[i−1]>B[j] 
这意味着 ii 太大，我们必须减小它。
感谢 @Quentin.chen 指出： i &lt; m \implies j &gt; 0i<m⟹j>0 以及 i &gt; 0 \implies j &lt; ni>0⟹j<n 始终成立，这是因为：

m \leq n,\ i &lt; m \implies j = \frac{m+n+1}{2} - i &gt; \frac{m+n+1}{2} - m \geq \frac{2m+1}{2} - m \geq 0m≤n, i<m⟹j= 
2
m+n+1
​	
 −i> 
2
m+n+1
​	
 −m≥ 
2
2m+1
​	
 −m≥0

m \leq n,\ i &gt; 0 \implies j = \frac{m+n+1}{2} - i &lt; \frac{m+n+1}{2} \leq \frac{2n+1}{2} \leq nm≤n, i>0⟹j= 
2
m+n+1
​	
 −i< 
2
m+n+1
​	
 ≤ 
2
2n+1
​	
 ≤n

所以，在情况 2 和 3中，我们不需要检查 j &gt; 0j>0 或是 j &lt; nj<n 是否成立。