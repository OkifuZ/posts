---
title: Everything(I know) about monotonic stack and monotonic queue
date: 2021-03-05 20:15:29
tags:
excerpt: what is monotonic data structure and how it works when comes to specific situation
---



## 单调栈的性质与创建

单调栈作为一种栈式数据结构，可以分为递增单调栈以及递减单调栈两类，在增减性方面又可以分为严格的与不严格的。在栈的基础上，**只需在数据结构的任何状态都要满足数据的单调性即可**。

我这里以严格单调递增栈`monotonic_stack`为例，演示其创建过程。

现有一组数据如下所示，

```cpp
auto data = vector<int>{1, 2, 3, 4, 3, 1}
```

先要将其压入栈`monotonic_stack`中，前4个数压栈的过程与普通的栈无异，因为这四个数据是严格单调递增的：

```cpp
stack<int> monotonic_stack; 
for (int i = 0; i < 4; i++) {
    monotonic_stack.push(data[i]);
    // or: monotonic_stack.push(i); depends on specific situation 
}
```

对于后倒数第二个数3，在压栈时小于等于栈顶，故需要将栈顶pop掉。直至栈顶元素严格小于3，因此，创建操作可以重写为：

```cpp
for (int i = 0; i < data.size(); i++) {
	while (monotonic_stack.top() >= data[i]) { // i == 4
        monotonic_stack.pop();
    }
    monotonic_stack.push(data[i]);
    // or: monotonic_stack.push(i);
}
```

对与最后一个数1也是同理。将`data`全部压栈后，将得到仅含一个元素1的单调栈。

创建一个单调栈就这么简单。关于创建操作的时间复杂度，虽然难以衡量单次压栈的时间，但是最坏情况下（最后一个元素是最小值），我们所作的无外乎将所有数据压入栈中，又将其全部弹出，故为O(N)；又由于至少要顺序遍历一遍，故时间复杂度为$\Theta (N)$。



## 单调栈的功能

那么单调栈能做到哪些普通栈做不到的事情？大致有以下两点（还是以严格单调递增栈为例）：

- 可以在**顺序遍历**一组数时找到已经遍历过的数的**最小值**，也就是当前单调栈的**栈底元素**。这个功能和优先队列一样。
- 可以在**顺序遍历**一组数时，找到当前栈顶元素top的左侧，**距离最近的**，**小于等于top**的元素的位置，即栈顶下面的元素，若此时栈中只含有一个元素top，则top是遍历过的元素中最小的那个。

这的两种功能实质上都是查询操作，有时以外的实用，且时间复杂度为O(1)。



## 可以使用单调栈的题目

最典型的就是`LeetCode84`了，直方图里最大矩阵面积。

tbc...



## 单调队列

