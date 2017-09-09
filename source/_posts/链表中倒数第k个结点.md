---
title: 链表中倒数第k个结点
date: 2017-09-09 09:46:13
tags:
    - Java
categories:
    - Algorithm
---

输入一个链表，输出该链表中倒数第k个结点。

**分析：**

这题比较笨的方法是将链表逆转，再查找第k个元素。参考[从尾到头打印链表](https://jochen-m.github.io/2017/09/04/%E4%BB%8E%E5%B0%BE%E5%88%B0%E5%A4%B4%E6%89%93%E5%8D%B0%E9%93%BE%E8%A1%A8/)。

**比较巧妙的方法是：**

采用两个指针，开始时都指向头节点。首先第一个指针向前移动k步（如果可以移动k步），然后两个指针同时向前移动，当第一个指针指向null时，第二个指针就指向了倒数第k个元素。妙哉~

```
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
public class Solution {
    public ListNode FindKthToTail(ListNode head,int k) {
        if(head == null || k <= 0)
            return null;

        ListNode left = head;
        ListNode right = head;

        for(int i = 0; i < k; i++) {
            if(right == null)
                return null;
            right = right.next;
        }

        while(right != null) {
            right = right.next;
            left = left.next;
        }

        return left;
    }
}
```
