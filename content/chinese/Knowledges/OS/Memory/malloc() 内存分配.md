---
title: "malloc() 内存分配"
author: "Roser"
date: 2025-05-08
image: "images/content/OS.png"
draft: false
tags:
  - OS
  - Cpp
  - Review
draft: true
sr-due: 2025-05-11
sr-interval: 27
sr-ease: 250
---
`malloc()` 是 C 语言的内存分配函数，同时也是 C++ `new` 内部调用的函数。该函数用于申请动态内存。该函数不是系统调用，但是该函数会使用系统调用分配内存。

`malloc()` 申请的为**虚拟内存**，因此如果申请内存后，程序没有用到这部分内存，则不会占用物理内存，在查看系统状态时是看不到内存增加的。

在 Linux 系统上，`malloc()` 有两种分配策略：`brk()` 申请和 `mmap()` 申请。
- 如果用户申请的内存小于 128KB，则通过 `brk()` 申请。
- 如果用户申请的内存大于 128KB，则通过 `mmap()` 申请。

> 不同的 `glibc` 版本定义的阈值不同。
### `brk()` 分配

这种方式会通过**系统调用**从[堆](../Linux-系统虚拟内存空间分布)分配内存。该函数会将**堆顶**指针向高地址移动，获得新的内存空间。

![](images/brk申请内存.webp)
### `mmap()` 分配

这种方式会通过**系统调用中私有匿名映射**的方式，在[文件映射段](../Linux-系统虚拟内存空间分布)分配一块内存，实际是映射（偷）了一段文件映射段的内存。

![](images/mmap申请内存.webp)

***
通过 `brk()` 分配内存时，实际会分配一段超过 128KB（阈值）的连续内存空间作为预留区域。这样能减少系统调用的次数，后续继续分配小内存时，能直接使用这部分已经分配的内存，不需要再次分配。

`mmap()` 则会按照用户实际需要的大小来分配内存，但为了内存对齐，有时会比所需内存大一些。