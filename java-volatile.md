---
title: java多线程-volatile 解惑
date: 2017-11-06 20:23:44
tags: [java,多线程]
categories: java
---
> volatile 总结很好的文章。

# 前言
volatile关键字可能是Java开发人员“熟悉而又陌生”的一个关键字。本文将从volatile关键字的作用、开销和典型应用场景以及Java虚拟机对volatile关键字的实现这几个方面为读者全面深入剖析volatile关键字。

volatile字面上有“挥发性的，不稳定的”意思，它是用于修饰可变共享变量（Mutable Shared Variable）的一个关键字。所谓“共享”是指一个变量能够被多个线程访问（包括读/写），所谓“可变”是指变量的值可以发生变化。换而言之，volatile关键字用于修饰多个线程并发访问的同一个变量，这些线程中至少有一个线程会更新这个变量的值。我们称volatile修饰的变量为volatile变量。我们知道锁的作用包括保障原子性、保障可见性以及保障有序性。volatile常被称为“轻量级锁”，其作用与锁有类似的地方——volatile也能够保障原子性（仅保障long/double型变量访问操作的原子性）、保障可见性以及保障有序性。

本文所提及的“Java虚拟机”如无特别说明，均特指Oracle公司的HotSpot Java虚拟机。

<!-- more -->
# 保障long/double型变量访问操作的原子性
不可分割的操作被称为原子操作（Atomic Operation）。所谓不可分割（Indivisible）是指一个操作从其执行线程以外的其他线程看来，该操作要么已经完成要么尚未开始，也就是说其他线程不会看到该操作的中间结果。如果一个操作是原子操作，那么我们就称该操作具有原子性（Atomicity）。

Java语言规范（Java Language Specification，JLS）规定，Java语言中针对long/double型以外的任何变量（包括基础类型变量和引用型变量）进行的读、写操作都是原子操作，即Java语言规范本身并不规定针对long/double型变量进行读、写操作具有原子性。一个long/double型变量的读/写操作在32位Java虚拟机下可能会被分解为两个子步骤（比如先写低32位，再写高32位）来实现，这就导致一个线程对long/double型变量进行的写操作的中间结果可以被其他线程所观察到，即此时针对long/double型变量的访问操作不是原子操作。清单1所示的实验展示了这点。

清单1 long/double型变量写操作的原子性问题Demo