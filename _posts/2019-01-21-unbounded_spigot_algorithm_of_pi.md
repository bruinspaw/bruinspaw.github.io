---
layout: post
title:  "Unbounded Spigot Algorithms for Pi"
date:   2019-01-21 20:24:20 +0800
author: bruinspaw
categories: Algorithm 
---

近来研究了圆周率的算法，发现Jeremy Gibbons的[Unbounded Spigot Algorithms for Pi](https://www.cs.ox.ac.uk/jeremy.gibbons/publications/spigot.pdf)是最优雅的一种算法（可能不是最快的）。陆续又找到一些关于算法细节的说明，特别是关于数列收敛的介绍（[Computing the Digits in Pi](https://www.cs.umb.edu/~offner/files/pi.pdf)）。算法用到了无限精度整数的运算，现把Python实现列于下面：

```
"""Implementation of Gibbons' Unbounded Spigot Algorithms for Pi

Formula:
  Leibniz formula with convergence acceleration

References:
  https://www.cs.ox.ac.uk/jeremy.gibbons/publications/spigot.pdf
  https://seminar-materials.iijlab.net/iijlab-seminar/iijlab-seminar-20170327.pdf
  https://www.cs.umb.edu/~offner/files/pi.pdf
"""


def __compose(a, b):
    """Composition of two liner fractional transformation:
    i.e. multiplication of matrices [a0, a1; a2, a3] and
    [b0, b1; b2, b3], and note that a2 and b2 are always 0.
    """

    return (a[0]*b[0], a[0]*b[1]+a[1]*b[3], 0, a[3]*b[3])


def __lft(a, x):
    """Linear transformation of x corresponding to the matrix:
    [a0, a1; a2, a3] (a2 = 0), and only extract integer part.
    """

    return (a[0]*x+a[1])//a[3]


def __produce(a, n):
    """Extract the next decimal digit of π from a (and incorporate
    that digit in the output stream), and multiply a on the left
    by a linear fractional transformation that reflects the fact
    that it has “lost that value”.
    """

    return __compose((10, -10*n, 0, 1), a)


def pi_stream():
    k = 1
    s = (1, 0, 0, 1)               # stream of LFT
    while True:
        f = (k, 4*k+2, 0, 2*k+1)   # LFT
        m = __lft(s, 3)
        n = __lft(s, 4)
        if m == n:
            s = __produce(s, m)
            yield m
        else:
            s = __compose(s, f)
            k += 1

if __name__ == '__main__':
    n = 0		
    for d in pi_stream():
        if n == 0: print('%d.' % d)
        else: print(d, end='')
        if n and n%10 == 0: print(' ', end='')
        if n and n%50 == 0: print('  :%d' % n)
        n += 1
        if n > 2000: break
```
