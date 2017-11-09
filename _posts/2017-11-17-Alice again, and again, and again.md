---
layout: post
title: Alice again, and again, and again.
category: Mumbo
description:
---
# 题目：
Alice decides to use RSA with the public key N = 1889570071. In order to guard against transmission errors, Alice has Bob encrypt his message twice, once using the encryption exponent e1 = 1021763679 and once using the encryption exponent e2 = 519424709. Eve intercepts the two encrypted messages

c1 = 1244183534 and c2 = 732959706. Assuming that Eve also knows N and the two encryption exponents e1 and e2. Please help Eve recover Bob’s plaintext without finding a factorization of N.

# 推算
Eve 知道N， e_1, e_2。

c1 = m^{e_1} % N
c2 = m^{e_2} % N
由于Eve 有两个加密密钥，可以计算得到
gcd(e_1, e_2) = 1
 
则有：
m = m^{e_1 x + e_2 y} = c_1^x \dot c_2^y
其中，x, y 又可以通过：
xgcd(e_1, e_2)计算获得。

# 结果
我在python上算出来的结果是 m = 64074575。
