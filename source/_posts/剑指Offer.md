---
title: 剑指Offer
date: 2017-09-03 23:45:59
tags:
---

#### 二维数组中的查找
在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

```
java:

public boolean Find(int target, int [][] array) {
    if(array == null || array.length < 1 || array[0].length < 1)
        return false;

    int rows = array.length;
    int cols = array[0].length;

    int row = 0;
    int col = cols - 1;

    while(row > -1 && row < rows && col > -1 && col < cols){
        if(array[row][col] == target){
            return true;
        }else if(array[row][col] > target){
            col--;
        }else if(array[row][col] < target){
            row++;
        }
    }

    return false;
}
```

#### 从尾到头打印链表
输入一个链表，从尾到头打印链表每个节点的值。

```
/**
*    public class ListNode {
*        int val;
*        ListNode next = null;
*
*        ListNode(int val) {
*            this.val = val;
*        }
*    }
*
*/
import java.util.ArrayList;
public class Solution {
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        ArrayList<Integer> list = new ArrayList<>();

        ListNode next = null;
        ListNode pre = null;

        while(listNode != null){
            next = listNode.next;
            listNode.next = pre;
            pre = listNode;
            listNode = next;
        }

        while(pre != null){
            list.add(pre.val);
            pre = pre.next;
        }

        return list;
    }
}
```
