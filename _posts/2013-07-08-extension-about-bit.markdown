---
layout: post
title: "一些关于BIT扩展的整理"
date: 2013-07-08 21:53
comments: true
categories: BIT 树状数组 数据结构
description: 对于树状数组这一数据结构整理一下其在信息学竞赛中的实用扩展。
---
树状数组（BIT）以其代码简洁、效率高见长，但又由于它的应用弹性远不如线段树而与其在竞赛中相互制衡。
然而，对于我等手残党而言，为了尽可能避免使用线段树这种又长又难调的东西，学会增强BIT是十分必要的！

###基本模型-改点球段
```cpp
void update(int *d, int x, int val) {
    for (; x<=n; d[x]+=val, x+=lowbit(x));
}

int query(int *d, int x) {
    int ret = 0;
    for (; x<=n; ret+=d[x], x-=lowbit(x));
    return ret;
}
```
树状数组的概念啊原理啊什么的都不是本文讨论的对象，因此仅献上代码，后文的几种扩展都在此基础之上。
<!--more-->
###第一次修改-改段求点
首先分析一下BIT的作用。
不加BIT的情况下，对一个数组进行点修改的复杂度为`O(1)`，段询问复杂度为`O(n)`；而加入BIT之后，两个操作的复杂度都变为了`O(log n)`。
也就是说，他能将在`O(log n)`的时间内完成**点修改**和**段询问**。而我们的目标是在同样的时间内完成**段修改**和**点询问**。

    origin:  1  7  7  6  0  4  3
    assist:  1  6  0 -1 -6  4 -1

设计一个新的数组`assist[]`，使得assist[i]=origin[i]-origin[i-1]。也就是说，`assist[]`储存的是原数组的增量。将前面的等式变形后可以得到origin[i]=assist[i]+origin[i-1]，以此类推，可以得到origin[i]=assist[i]+assist[i-1]+...+assist[1]。由此发现，assist数组的改动在原数组中会波及此元素后面所有的元素。利用这一性质便可以方便的将段修改转化为**点修改**。

    assist[l] += val;
    assist[r+1] -= val;

而原来的点询问也变成了段询问。于是就明朗了。我们只需要将树状数组建立在`assist[]`之上即可实现该段求点。

（当初学这个的时候被误导的很深啊草）

###出乎意料的简单-该段求段
很多学了BIT的人大多只学到该段求点便撒手不往下学，往往忽略了改段求段这一简单而有意想不到的实用的扩展。

先从改段求点开始考虑。

    origin:  1  7  7  6  0  4  3
    assist:  1  6  0 -1 -6  4 -1

如果要求原数组中前5个数的和的话，我们先要求出assist[]中前五个数的和构成origin[5]，然后是前四个数的和构成origin[4]，前三个数的和构成origin[3]...再把这些全都加出来即可。

![演示][1]

这样做的时间复杂度是`O(nlogn)`的，显然不是我们所预期的。注意到上图的那些橙色的线条，如果我们在他们右边补上一些线条，形成一个方块的话....

![演示][2]

只消将第一个橙条乘上6，再减去蓝条便能得到我们想要的结果。那么如何在`O(logn)`的时间内求出蓝条呢？观察图片不难发现，assist[5]在蓝条中出现了5次，assist[4]出现了4次，assist[3]出现了3次...于是我们只需要在开一个数组`assist2[]`使得assist2[i]=assist[i]*i即可。

    origin:  1  7  7  6  0  4  3
    assist:  1  6  0 -1 -6  4  -1
    assist2: 1 12  0 -4 -30 24 -7

    sum(origin,n) = sum(assist,n)*(n+1) - sum(assist2,n)

附上代码。

```cpp
void update(int *d, int p, int delta) {
	for (; p<=n; d[p]+=delta, p+=lowbit(p)) ;
}

int query(int *d, int p) {
	int ret;
	for (ret=0; p; ret+=d[p], p-=lowbit(p)) ;
	return ret;
}

void add(int a, int b, int c) {
	update(d, a, c);
	update(d, b+1, -c);
	update(d_help, a, c*a);
	update(d_help, b+1, -c*(b+1));
}

int sum(int a) {
	if (!a) return 0;
	return (a+1)*query(d, a) - query(d_help, a);
}
```

**练习题：** [POJ 3468](http://poj.org/problem?id=3468)：A Simple Problem with Integers

[view code](http://codepad.org/4TGVtHcp)

（未完待续）

  [1]: http://thumbsnap.com/i/PiShTZwn.png
  [2]: http://thumbsnap.com/i/PTWot0ij.png