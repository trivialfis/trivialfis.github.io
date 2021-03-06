---
layout: post
title: 展示 DES 解密过程实质是加密过程的逆过程。
category: Mumbo
description:
---

# DES 与 Feistel Cipher Structure 的不同点
DES是在 Feistel Cipher Structure (FCS) 基础上构建的加密算法。在加密过程中， DES在FCS开始前添加 Initial Permutation 和对原来 FCS 的输出添加了 逆向的 Initial Permutation。 同时，也通过circular shift 和 permute 实现了 SubKey 的生成。

# 解密是加密的逆过程
基于上述所说的不同点， 同时已知 FCS 本身解密是加密的逆过程，那么我们只要证明 DES 跟 FCS 的不同点不会对算法带来影响即可。

## Permutation
首先展示 $IP$ 和 $IP^{-1}$ 不会对算法带来影响。 把 Permutation 操作看作一个由 Permutation Matrix 实现的线性变换， 由 Permutation Matrix 本身的性质可知 $PP^{-1} = I$， 也就是加密的 preoutput 和解密过程开始时经过 $P^{-1}$ 的密文是一样的。

## Subkey
然后是 SubKey 的生成。先对 circular shift 进行讨论。 加密时进行正向的 circular shift， 解密时进行反向 circular shift。 而解密时初始化使用的是加密过程最后一个 round 的密钥。所以加密和解密过程中每个 round 所对应的密钥是一样的。 然后到 SubKey 生成中使用的 permutation。既然每个 round 使用的是同一条密钥，而 permutation 的方法也一样，生成的 Subkey 自然也就一样了。

# 结论
综上所述， DES 所增加的特性对 Feistel Cipher Structure 的性质并不影响。 而 FCS 的解密过程是加密的逆过程，所以 DES 的解密过程同样是加密的逆过程。
