---
title: "SICP Chapter 1 习题全解"
date: 2021-07-31T14:22:21+08:00
---

<!--more-->

## Exercise 1.1

```scheme
10
10

(+ 5 3 4)
12

(- 9 1)
8

(/ 6 2)
3

(+ (* 2 4) (- 4 6))
6

(define a 3) 
a

(define b (+ a 1)) //b 4
b

(+ a b (* a b))
19

(= a b)
#f

(if (and (> b a) (< b (* a b)))
		b 
		a)
4

(cond ((= a 4) 6)
			((= b 4) (+ 6 7 a))
			(else 25))
16

(+ 2 (if (> b a) b a))
6

(* (cond ((> a b) a) 
				 ((< a b) b)
				 (else -1)) 
   (+ a 1))
16
```

## Exercise 1.2

```scheme
(/ (+ 5 
			4 
			(- 2 
      	 (- 3 
         		(+ 6 
            	 (/ 4 5)))))
		(* 3 
    	 (- 6 2) 
       (- 2 7)))
```

## Exercise 1.3

```scheme
(define (square x)
    (* x x))

(define (sum-of-the-two-larger-number x y z)
    (cond ((and (> x z) (> y z)) (+ (square x) (square y)))
        	((and (> x y) (> z y)) (+ (square x) (square z)))
        	((and (> z x) (> y x)) (+ (square z) (square y)))))
```

## Exercise 1.4

```scheme
(define (a-plus-abs-b a b) 
  ((if (> b 0) + -) a b))
```

**Model of evaluation allows for combinations whose operators are compound expressions.** 

这里`(if (> b 0) + -)`是一个复合表达式的组合式，它作为了运算符，也很好理解，从procedure的名字可以看出，过程的行为是`a+|b|`,当`b>0`时，返回值是`a+b` ,而当`a<b`时，返回值为`a-b`。

## Exercise 1.5

```scheme
(define (p) (p)) 
(define (test x y)
	(if (= x 0) 0 y))

(test 0 (p))
```

首先，可以确定的是，无论解释器使用的是什么求值方式，调用 `(p)` 总是进入一个无限循环(infinite loop)，因为函数 `p` 会不断调用自身。

在应用序中，所有被传入的实际参数都会立即被求值，因此，在使用应用序的解释器里执行 `(test 0 (p))` 时，实际参数 `0` 和 `(p)` 都会被求值，而对 `(p)` 的求值将使解释器进入无限循环，因此，如果一个解释器在运行 Ben 的测试时陷入停滞，那么这个解释器使用的是应用序求值模式。

另一方面，在正则序中，传入的实际参数只有在有需要时才会被求值，因此，在使用正则序的解释器里运行 `(test 0 (p))` 时， `0` 和 `(p)` 都不会立即被求值，当解释进行到 `if` 语句时，形式参数 `x` 的实际参数(也即是 `0`)会被求值(求值结果也是为 `0` )，然后和另一个 `0` 进行对比(`(= x 0)`)，因为对比的值为真(`#t`),所以 `if` 返回 `0` 作为值表达式的值，而这个值又作为 `test` 函数的值被返回。

因为在正则序求值中，调用 `(p)` 从始到终都没有被执行，所以也就不会产生无限循环，因此，如果一个解释器在运行 Ben 的测试时顺利返回 `0` ，那么这个解释器使用的是正则序求值模式。

参考资料 https://sicp.readthedocs.io/en/latest/chp1/5.html

## Exercise 1.6

 `sqrt-iter` 函数，如果使用 `trace` 来跟踪它的调用过程的话，就会发现它执行了大量的递归调用，这些调用数量非常庞大，最终突破解释器的栈深度，造成错误。

`if` 语句是一种特殊形式，当它的 `predicate` 部分为真时， `then-clause` 分支会被求值，否则的话， `else-clause` 分支被求值，两个 `clause` 只有一个会被求值。

而另一方面，新定义的 `new-if` 只是一个普通函数，它没有 `if` 所具有的特殊形式，根据解释器所使用的应用序求值规则，每个函数的实际参数在传入的时候都会被求值，因此，当使用 `new-if` 函数时，无论 `predicate` 是真还是假， `then-clause` 和 `else-clause` 两个分支都会被求值。

参考资料 https://sicp.readthedocs.io/en/latest/chp1/6.html

## Exercise 1.7

```scheme
(define (good-enough? guess x)
    (< (abs (- (improve guess x) guess)) 0.001))

我的答案还是取了差值，还是没有解决问题

(define (good-enough? old-guess new-guess)
    (> 0.01
       (/ (abs (- new-guess old-guess))
          old-guess)))

网上的答案取了变化率，当变化率小于1%时，就good enough了

(define (sqrt-iter guess x)
    (if (good-enough? guess (improve guess x))  ; 调用新的 good-enough?
        (improve guess x)
        (sqrt-iter (improve guess x)
                   x)))
但是他的解法需要修改原来的程序。

(define (good-enough? guess x)
    (< (/ (abs (- (improve guess x) guess)) guess) 0.01))
修改我的代码，不用修改原来的程序。
```

## Exercise 1.8

写一个修改版的improve过程即可。

```scheme
(define (improve guess x)
    (/ (+ (/ x (square guess)) (* 2 guess)) 3))
```

## Exercise 1.9

```scheme
(define (+ a b)
(if (= a 0) b (inc (+ (dec a) b))))
;recursive process
(+ 4 5)
(inc (+ 3 5))
(inc (inc (+ 2 5)))
(inc (inc (inc (+ 1 5))))
(inc (inc (inc (inc(+ 0 5)))))
(inc (inc (inc (inc 5))))
(inc (inc (inc 6)))
(inc (inc 7))
(inc 8)
9
```

从计算过程中可以很明显地看到伸展和收缩两个阶段，且伸展阶段所需的额外存储量和计算所需的步数都正比于参数 `a` ，说明这是一个线性递归计算过程(recursive process)。

```scheme
(define (+ a b)
(if (= a 0) b (+ (dec a) (inc b))))
;iterative process
(+ 4 5)
(+ 3 6)
(+ 2 7)
(+ 1 8)
(+ 0 9)
9
```

从计算过程中可以看到，这个版本的 `plus` 函数只使用常量存储大小，且计算所需的步骤正比于参数 `a` ，说明这是一个线性迭代计算过程(iterative process)。

## Exercise 1.10

```scheme
(define (A x y) 
  (cond ((= y 0) 0)
				((= x 0) (* 2 y))
				((= y 1) 2)
				(else (A (- x 1) (A x (- y 1))))))
;Ackermann’s function.
```

手算我是不大行了，我改写成了go代码如下，运行结果:

```go
func A(x, y int) int {
	if y == 0 {
		return 0
	}
	if x == 0 {
		return 2 * y
	}
	if y == 1 {
		return 2
	}
	return A(x-1, A(x, y-1))
}
//Ackermann’s function.
//(A 1 10)   1024
//(A 2 4)    65536
//(A 3 3)    65536
```

数学表达式:

```scheme
(define (f n) (A 0 n)) 
```

推导易得:
$$
f(n)=2*n
$$

```scheme
(define (g n) (A 1 n))
```

推导易得:
$$
g(n)=2^n
$$

```scheme
(define (h n) (A 2 n))
```

函数 `h` 计算的是连续求 n 次二次幂，即
$$
2^{2^{.^{.2}}}
$$

## Exercise 1.11

递归过程的代码很直接:

```scheme
(define (f *n*)
	(if (< n 3) 
      n
      (+ (f (- n 1))
				 (* 2 (f (- n 2)))
				 (* 3 (f (- n 3)))))))
;recursive process
```

迭代过程的代码可以参考书中求fib的过程完成:

```scheme
(define (f-iter a b c cnt)
(if (= cnt 0) 
    a
    (f-iter b c (+ (* 3 a ) (* 2 b) c) (- cnt 1))
))
;iterative process
```

## Exercise 1.12

用Scheme实在是不大会写:

```scheme
(define (pascal row col)
    (cond ((> col row)
            (error "unvalid col value"))
          ((or (= col 0) (= row col))
            1)
          (else (+ (pascal (- row 1) (- col 1))
                   (pascal (- row 1) col)))))
;recursive process

;公式求杨辉三角数
(define (pascal row col)
    (/ (factorial row)
       (* (factorial col)
          (factorial (- row col)))))
;iterative process
```

$$
pascal(col,row)=\frac{row!}{col!·(row-col)!}
$$

https://sicp.readthedocs.io/en/latest/chp1/12.html

## Exercise 1.13

数学证明没啥兴趣

## Exercise 1.14

![tree recursive](https://sicp.readthedocs.io/en/latest/_images/14.png)

## Exercise 1.15

https://sicp.readthedocs.io/en/latest/chp1/15.html

```scheme
(define (cube x) (* x x x))

(define (p x) (- (* 3 x) (* 4 (cube x))))

(define (sine angle)
    (if (not (> (abs angle) 0.1))
        angle
        (p (sine (/ angle 3.0)))))
```

+ a) How many times is the procedure p applied when(sine 12.15) is evaluated?

 `p` 共运行了 5 次

+ b)What is the order of growth in space and number of steps (as a function of a) used by the process generated by the sine procedure when (sine a) is evaluated?

在求值 `(sine a)` 的时候， `a` 每次都被除以 `3.0` ，而 `sine` 是一个递归程序，因此它的时间和空间复杂度都是 O(log a),每当 `a` 增大一倍(准确地说，是乘以因子 `3`)， `p` 的运行次数就会增加一次。

## Exercise 1.16

```scheme
(define (fast-expt b n)
    (expt-iter b n 1))

(define (expt-iter b n a)
    (cond ((= n 0)
            a)
          ((even? n)
            (expt-iter (square b)
                       (/ n 2)
                       a))
          ((odd? n)
            (expt-iter b
                       (- n 1)
                       (* b a)))))
```

利用恒等式：
$$
(b^\frac{n}{2})^2 = (b^2)^\frac{n}{2}
$$
不变量`a*b^n`，当n从n变化到0时，a就等于`b^n`，当n为奇数时，`a*b^n`和`a*b*b^n-1` 相等，当n为偶数时，`a*b^n`和`a*(b^2)^(n/2)` 相等。翻译成go代码如下:

```go
func fastExp(b, n int) int {
	var fastExpIter func(a, b, n int) int
	fastExpIter = func(a, b, n int) int {
		if n == 0 {
			return a
		}
		if n%2 == 0 {
			return fastExpIter(a, b*b, n/2)
		} else {
			return fastExpIter(a*b, b, n-1)
		}
	}
	return fastExpIter(1, b, n)
}
```

## Exercise 1.17

用go来描述：

```go
func multi(a, b int) int {
	if b == 0 {
		return 0
	}
	if b%2 == 0 {
		return multi(a, b>>1) << 1
	} else {
		return a + multi(a, b-1)
	}
}
//recursive process
```

迭代形式参考Exercise 1.18.

## Exercise 1.18

```go
func multi(a,b int) int{
	var multiIter func(a,b,res int) int
	multiIter = func(a, b, res int) int {
		if b==0{
			return res
		}
		if b%2==0 {
			return multiIter(a<<1,b>>1,res)
		}else {
			return multiIter(a,b-1,res+a)
		}
	}
	return multiIter(a,b,0)
}
//iterative process
```

## Exercise 1.19

大概就是矩阵快速幂(幸好这些内容我有基础)
$$
\left(\begin{matrix}1 & 1  \\1 & 0  \\\end{matrix}\right)*
 \left(\begin{matrix}f(n-1)\\ f(n-2)\\\end{matrix}\right) =
 \left(\begin{matrix}f(n)\\ f(n-1)\\\end{matrix}\right)
$$
即求左边这个矩阵的n次幂，可以用到之前的快速幂算法。

```scheme
(define (fib n)
    (fib-iter 1 0 0 1 n))

(define (fib-iter a b p q n)
    (cond ((= n 0)
            b)
          ((even? n)
            (fib-iter a 
                      b
                      (+ (square p) (square q))     ; 计算 p'
                      (+ (* 2 p q) (square q))      ; 计算 q'
                      (/ n 2)))
          (else
            (fib-iter (+ (* b q) (* a q) (* a p))
                      (+ (* b p) (* a q))
                      p
                      q
                      (- n 1)))))
```

参考 https://sicp.readthedocs.io/en/latest/chp1/19.html 此题相当于特别实现了二阶矩阵的乘法。

通用的算法即是实现矩阵的乘法，利用快速幂即可解决此类问题。

## Exercise 1.20

应用序和正则序 https://sicp.readthedocs.io/en/latest/chp1/20.html

## Exercise 1.21

```scheme
(smallest-divisor 199)
199
(smallest-divisor 1999)
1999
(smallest-divisor 19999)
7
```

## Exercise 1.22

简单题，不做了。

https://sicp.readthedocs.io/en/latest/chp1/22.html

## Exercise 1.23

简单题，不做了。

https://sicp.readthedocs.io/en/latest/chp1/23.html

## Exercise 1.24

简单题，不做了。

https://sicp.readthedocs.io/en/latest/chp1/24.html

## Exercise 1.25

数太大，没有办法求完幂再求余，因为费马检查在对一个非常大的数进行素数检测的时候，可能需要计算一个很大的乘幂，比如说，求十亿的一亿次方，这种非常大的数值计算的速度非常慢，而且很容易因为超出实现的限制而造成溢出。而书本 的 `expmod` 函数，通过每次对乘幂进行 `remainder` 操作，从而将乘幂限制在一个很小的范围内（不超过参数 `m` ），这样可以最大限度地避免溢出，而且计算速度快得多。

## Exercise 1.26

```scheme
(expmod base (/ exp 2) m) ;算了两遍
```

所以每次当 `exp` 为偶数时， Louis 的 `expmod` 过程的计算量就会增加一倍，因此原本 Θ(log⁡n) 的计算过程变成了 Θ(n) 。

## Exercise 1.27

```scheme
(define (carmichael-test n)
    (test-iter 1 n))

(define (test-iter a n)
    (cond ((= a n)
            #t)
          ((congruent? a n)
            (test-iter (+ a 1) n))
          (else
            #f)))

(define (congruent? a n)           ; 同余检测
    (= (expmod a n n) a))
```

将这个测试函数称之为 `carmichael-test` ，对于给定参数 n ，它要检验所有少于 n 的数 a， a^n是否都与 a 模 n同余.

## Exercise 1.28

证明参考wiki [米勒罗宾素数测试](https://www.wikiwand.com/zh-hans/%E7%B1%B3%E5%8B%92-%E6%8B%89%E5%AE%BE%E6%A3%80%E9%AA%8C)

这里的 `nontrivial square root of 1 modulo n` 意思是，如果a^2mod n == 1,n必定不是素数，在快速幂里面加上这一步验证即可，附上用go写的代码：

```go
func millerRabinPrimeTest(n int) bool {
	var testIter func(times int) bool
	testIter = func(times int) bool {
		if times == 0 {
			return true
		}
		if fastExpMod(nonZeroRandom(n), n-1, n) == 1 {
			return testIter(times - 1)
		} else {
			return false
		}
	}
	return testIter(n / 2)
}

func nonZeroRandom(n int) int {
	r := rand.Intn(n)
	if r == 0{
		return nonZeroRandom(n)
	}else {
		return r
	}
}

func nontrivialSquareRoot(a, n int) bool {
	return a != 1 && a != n-1 && (a*a)%n == 1
}

func square(x int) int {
	return x * x
}

func fastExpMod(base, exp, mod int) int {
	if exp == 0 {
		return 1
	}
	if nontrivialSquareRoot(base, mod) {
		return 0
	}
	if exp%2 == 0 {
		return square(fastExpMod(base, exp/2, mod)) % mod
	} else {
		return (base * fastExpMod(base, exp-1, mod)) % mod
	}
}
```

## Exercise 1.29

```python
def sum(term, a, next, b):
    if a > b:
        return 0
    return term(a) + sum(term, next(a), next, b)


def simpson(f, a, b, n):
    h = (b - a) / n

    def y(k):
        return f(a + k * h)

    def factor(k):
        if k == 0 or k == n:
            return 1
        if k % 2 == 1:
            return 4
        if k % 2 == 0:
            return 2

    def term(k):
        return factor(k) * y(k)

    def next(k):
        return k + 1

    if n % 2 != 0:
        print("error n can not be odd")
    return (h / 3) * sum(term, 0, next, n)


def cube(x):
    return x * x * x


if __name__ == '__main__':
    import sys
    sys.setrecursionlimit(3000)
    print(simpson(cube, 0, 1, 100))
    print(simpson(cube, 0, 1, 1000))
```

## Exercise 1.30

```python
def sum(term, a, next, b):
    def iter(a, result):
        if a > b:
            return result
        else:
            return iter(next(a), term(a) + result)

    return iter(a, 0)
```

## Exercise 1.31

+ a)

```python
def product(factor, a, next, b):
    if a > b:
        return 1
    else:
        return factor(a) * product(factor, next(a), next, b)


def pi(n):
    def factor(n):
        if n % 2 == 1:
            molecular = n + 1  # 分子
            denominator = n + 2  # 分母
        else:
            molecular = n + 2  # 分子
            denominator = n + 1  # 分母
        return molecular / denominator

    def next(n):
        return n + 1

    return 4 * product(factor, 1, next, 1000)


if __name__ == '__main__':
    import sys
    sys.setrecursionlimit(3000)
    
    print(pi(1000))

```

+ b)

```python
def product(factor, a, next, b):
    def iter(a, result):
        if a > b:
            return result
        else:
            return iter(next(a), result * factor(a))

    return iter(a, 1)
```

## Exercise 1.32

+ a

```python
def accumulate(combiner, null_value, term, a, next, b):
    if a > b:
        return null_value
    else:
        return combiner(term(a), accumulate(combiner, null_value, term, next(a), next, b))
```

+ b

```python
def accumulate(combiner, null_value, term, a, next, b):
    def iter(a, result):
        if a > b:
            return result
        else:
            return iter(next(a), combiner(term(a), result))

    return iter(a, null_value)
```

## Exercise 1.33

```python
import random


def filter_accumulate(combiner, null_value, term, a, next, b, filter):
    if a > b:
        return null_value
    else:
        if filter(a):
            return combiner(term(a), filter_accumulate(combiner, null_value, term, next(a), next, b, filter))
        else:
            return filter_accumulate(combiner, null_value, term, next(a), next, b, filter)


def square(x):
    return x * x


def is_nontrivial_square_root(a, mod):
    return a != 1 and a != mod - 1 and a * a % mod == 1


def fast_exp(a, b, mod):
    if b == 0:
        return 1
    if is_nontrivial_square_root(a, mod):
        return 0
    if b % 2 == 0:
        return square(fast_exp(a, b / 2, mod)) % mod
    else:
        return a * fast_exp(a, b - 1, mod) % mod


def rand(n):
    return random.randint(1, n - 1)


def is_prime(n):
    if n == 1:
        return False

    def iter(times):
        if times == 0:
            return True
        if fast_exp(rand(n), n - 1, n) == 1:
            return iter(times - 1)
        else:
            return False

    return iter(n//2)


def plus(x, y):
    return x + y


def identity(x):
    return x


def next(x):
    return x + 1


def sum_of_prime(a, b):
    return filter_accumulate(plus, 0, identity, a, next, b, is_prime)


def gcd(a, b):
    if b == 0:
        return a
    else:
        return gcd(b, a % b)


def multi(x, y):
    return x * y


def sum_of_relatively_prime(n):
    def is_relatively_prime(a):
        return gcd(a, n) == 1

    return filter_accumulate(multi, 1, identity, 1, next, n, is_relatively_prime)


if __name__ == '__main__':
    print(sum_of_prime(1,10)) # 17
    print(sum_of_relatively_prime(10)) # 189
```

## Exercise 1.35

```python
tolerance = 0.0001


def fixed_point(f, first_guess):
    def close_enough(v1, v2):
        return abs(v1 - v2) < tolerance

    def helper(guess):
        next_guess = f(guess)
        if close_enough(guess, next_guess):
            return next_guess
        return helper(next_guess)

    return helper(first_guess)


def golden_ratio():
    return fixed_point(lambda x: 1 + 1 / x, 1) # 1.6180555555555556
```

## Exercise 1.36

```python
def fixed_point_modified(f, first_guess):
    def close_enough(v1, v2):
        return abs(v1 - v2) < tolerance

    def helper(guess):
        next_guess = f(guess)
        print(next_guess)
        if close_enough(guess, next_guess):
            return next_guess
        return helper(next_guess)

    return helper(first_guess)


def average_damping(f):
    return lambda x: (x + f(x)) / 2


if __name__ == '__main__':
    print(fixed_point_modified(lambda x: log(1000) / log(x), 10))
    print("---------------------")
    print(fixed_point_modified(average_damping(lambda x: log(1000) / log(x)), 10))
    
# 2.9999999999999996
# 6.2877098228681545
# 3.7570797902002955
# 5.218748919675316
# 4.1807977460633134
# 4.828902657081293
# 4.386936895811029
# 4.671722808746095
# 4.481109436117821
# 4.605567315585735
# 4.522955348093164
# 4.577201597629606
# 4.541325786357399
# 4.564940905198754
# 4.549347961475409
# 4.5596228442307565
# 4.552843114094703
# 4.55731263660315
# 4.554364381825887
# 4.556308401465587
# 4.555026226620339
# 4.55587174038325
# 4.555314115211184
# 4.555681847896976
# 4.555439330395129
# 4.555599264136406
# 4.555493789937456
# 4.555563347820309
# 4.555563347820309
# ---------------------
# 6.5
# 5.095215099176933
# 4.668760681281611
# 4.57585730576714
# 4.559030116711325
# 4.55613168520593
# 4.555637206157649
# 4.55555298754564
# 4.55555298754564
```

可以看到，用了average_damping的过程收敛速度明显变快。

## Exercise 1.37

+ a

```python
def cont_frac(n, d, k):
    def helper(i):
        if i == k:
            return n(i) / d(i)
        else:
            return n(i) / (d(i) + helper(i + 1))

    return helper(1)
  
if __name__ == '__main__':
    print(
        cont_frac(
            lambda i: 1,
            lambda i: 1,
            100
        )
    )
# 0.6180339887498948
```

+ b

```python
def cont_frac_iter(n, d, k):
    def helper(i, res):
        if i == 0:
            return res
        else:
            return helper(i - 1, n(i) / (d(i) + res))

    return helper(k, 0)
```

## Exercise 1.38

```python
def approximate_e(k):
    def d(i):
        if (i + 1) % 3 == 0:
            return (i + 1) / 3 * 2
        else:
            return 1

    return 2 + cont_frac_iter(lambda i: 1, d, k)


if __name__ == '__main__':
    print(approximate_e(100))
# 2.7182818284590455
```

## Exercise 1.39

```python
def tan_cf(x, k):
    def n(i):
        if i == 1:
            return x
        else:
            return - x * x

    return cont_frac(n, lambda i: 2 * i - 1, k)


if __name__ == '__main__':
    print(tan_cf(0,100))
    print(tan_cf(3.1415926/2,100))
# 0.0
# 37320539.58514773 相当于无穷大，因为这里的pi，是近似值
```

## Exercise 1.40

```python
def cubic(a, b, c):
    return newtons_method(lambda x: cube(x) + a * square(x) + b * x + c, 1.0)
```

## Exercise 1.41

```python
def inc(x):
    return x + 1


def double(f):
    return lambda x: f(f(x))
  
double(double(double))(inc)(5) 
# 21
```

## Exercise 1.42

```python
def compose(f, g):
    return lambda x: f(g(x))
```

## Exercise 1.43

```python
def repeated(f, n):
    if n == 1:
        return f
    else:
        return compose(f, repeated(f, n - 1))
# recursive
def repeated_iter(f, n):
    def _iter(times, repeated_f):
        if times == 1:
            return repeated_f
        else:
            return _iter(times - 1, compose(f, repeated_f))

    return _iter(n, f)
# iterative
```

## Exercise 1.44

```python
def smooth(f):
    dx = 0.00001
    return lambda x: (f(x - dx) + f(x) + f(x + dx)) / 3


def n_fold_smoothed(f, n):
    return repeated(smooth, n)(f)
```

## Exercise 1.45

```python
def nth_root(n):
    def root(x):
        k = math.floor(math.log2(n))

        def f(y):
            return x / exp(y, n - 1)

        return fix_point_of_transform(f, repeated(average_damping, k), 1.0)

    return root
```

## Exercise 1.46

```python
def iterative_improve(good_enough, improve_method):
    def helper(guess):
        next_guess = improve_method(guess)
        if good_enough(guess, next_guess):
            return next_guess
        else:
            return helper(next_guess)

    return helper


def sqrt_by_iterative_improve(x):
    def close_enough(v1, v2):
        return abs(v1 - v2) < tolerance

    def f(y):
        return x / y

    return iterative_improve(close_enough, average_damping(f))(1.0)


def fixed_point(f, first_guess):
    def close_enough(v1, v2):
        return abs(v1 - v2) < tolerance

    return iterative_improve(close_enough, f)(first_guess)
```

