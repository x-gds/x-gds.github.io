---
layout: post
title:  "LeetCode #2 两数相加 Add Two Numbers"
date:   2018-09-15 21:51:55 +0800
categories: 算法
---

## LeetCode #2 两数相加 Add Two Numbers

给定两个非空链表来表示两个非负整数。位数按照逆序方式存储，它们的每个节点只存储单个数字。将两数相加返回一个新的链表。

你可以假设除了数字 0 之外，这两个数字都不会以零开头。

**示例**：

```
输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807
```

下面是单向链表的定义：

```java
Definition for singly-linked list.
public class ListNode {
   int val;
   ListNode next;
   ListNode(int x) { val = x; }
}
```

这道题还是比较简单的，由于正好是逆序存储的，跟我们平时做加法的顺序相同，也不用移动到链表尾部开始。使用链表时，一般的停止条件是`listNode.next == null`，另外就是注意非空判断。加法可能需要进位，注意这一点基本就没问题了。可以写出以下代码：

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        if (l1 == null) {
			return l2;
		}
		if (l2 == null) {
			return l1;
		}
		ListNode head = new ListNode(0);
		ListNode last = head;
		ListNode l1Node = l1;
		ListNode l2Node = l2;
		int carry = 0;
		while (l1Node != null && l2Node != null) {
			int sum = l1Node.val + l2Node.val + carry;
			carry = sum / 10;
			last.next = new ListNode(sum % 10);
			last = last.next;
			if (l1Node != null) {
				l1Node = l1Node.next;
			}
			if (l2Node != null) {
				l2Node = l2Node.next;
			}
		}
		while (l1Node != null) {
			if (carry > 0) {
				int sum = l1Node.val + carry;
				carry = sum / 10;
				last.next = new ListNode(sum % 10);
			} else {
				last.next = l1Node;
				break;
			}
			l1Node = l1Node.next;
			last = last.next;
		}
		while (l2Node != null) {
			if (carry > 0) {
				int sum = l2Node.val + carry;
				carry = sum / 10;
				last.next = new ListNode(sum % 10);
			} else {
				last.next = l2Node;
				break;
			}
			l2Node = l2Node.next;
			last = last.next;
		}
		if (carry > 0) {
			last.next = new ListNode(carry);
		}
		return head.next;
    }

}
```

这里用了两个额外的`while`循环来尽可能地缩短循环次数，也可以只用一个`while`循环，循环条件改为`l1Node != null || l2Node != null`，下面循环体内做出相应的更改。

算法的时间复杂度是`max(m, n)`，`m`和`n`分别是两个链表的长度。