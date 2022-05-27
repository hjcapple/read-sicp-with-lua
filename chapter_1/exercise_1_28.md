## P37 - [练习 1.28，Miller-Rabin 测试]

### 费马小定理

利用 [费马小定理](./fermat_test.md) 中描述的同余符号。费马小定理描述如下

如果 p 为素数，当 a < p 时，有

$$a^{p}\equiv a\pmod p$$

变换一下公式

$$a^{p-1}\equiv 1\pmod p$$

### Miller-Rabin 测试

题目的中文 "1 取模的非平凡平方根", 英文原文为 “nontrivial square root of 1 modulo n”。

英文中，trivial 这个词，意思是“不重要的”，“微不足道的", “琐琐碎碎" 的。用在数学中，是指某个结论或者性质显然易见，没有什么了不起。而 nontrivial 意思就跟 trivial 相反。

假设 x 是 1 取模的平方根（先不管 trivial 或者 nontrivial), 就有

$$x^{2}\equiv 1\pmod p$$

上面公式可以转为

$$x ^ {2} - 1 \equiv (x + 1)(x - 1)\equiv 0\pmod p$$

于是一定包含两个解。

$$x \equiv 1\pmod p$$

$$x \equiv -1\pmod p$$

上面推导没有应用到 p 的性质。不管 p 是是素数还是合数，这两个解都一定会成立。于是这两个解，没有什么了不起的，不能用来区分 p 的性质。换句话说这两个解是 trivial 的。

除了上面两个解，根据 p 的性质，上面模方程还可能有其它的解。这些解才能用于区分 p 的性质，这些解是 nontrivial 的。

而 -1 在模运算中，跟 p - 1 等价(想象时钟 12 点时，向后拨动 -1 时，就变成 11 点了)。因此在 [0, p) 的范围内，平凡解(trivial) 就是 1 和 p - 1。非平凡解(nontrivial) 就是除了这两个解之外的其它解。

当 p 为素数时，因为 p 已经不可拆分。$(x + 1)(x - 1)\equiv 0\pmod p$ 只有两个平凡解。但当 p 为合数时，还可以有其它解。比如当 p 为 8，就有

$$3^{2} \equiv 1\pmod 8$$

$$5^{2} \equiv 1\pmod 8$$

3 和 5 就是非平凡解。

------

Miller-Rabin 检查除了应用费马小定理的变形公式

$$a^{p-1}\equiv 1\pmod p$$

还应用了上述结论，添加了额外的一个检查。假如 p 是素数，那么上面方程

$$x ^ {2} \equiv 1\pmod p$$

一定没有非平凡解(nontrivial)。假如 [0, p) 的范围内，除了 1 和 p - 1 外，还有其它解，那么 p 就是合数。

### 代码

``` Scheme
#lang racket

(define (square x) (* x x))

(define (nontrivial? a n)
  (and (not (= a 1))
       (not (= a (- n 1)))
       (= (remainder (square a) n) 1)))

(define (expmod base exp m)
  (cond ((= exp 0) 1)
        ((nontrivial? base m) 0)
        ((even? exp)
         (remainder (square (expmod base (/ exp 2) m))
                    m))
        (else
          (remainder (* base (expmod base (- exp 1) m))
                     m))))  

(define (fmiller-rabin-test n)
  (define (try-it a)
    (= (expmod a (- n 1) n) 1))
  (try-it (+ 1 (random (- n 1)))))

(define (fast-prime? n times)
  (cond ((= times 0) true)
        ((fmiller-rabin-test n) (fast-prime? n (- times 1)))
        (else false)))

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
(module* test #f
  (require rackunit)
  (for-each (lambda (num)
             (check-true (fast-prime? num 100)))
           '(2 3 5 7 11 13 17 19 23))
  (for-each (lambda (num)
             (check-false (fast-prime? num 100)))
           '(36 25 9 16 4 561 1105 1729 2465))
)
```

注意看到，561, 1105, 1729, 2465 这几个 carmichael 数字也被检测出并非素数。

