---
title: "SICP 习题全解 (2)"
date: 2021-08-22T15:52:21+08:00
---

<!--more-->

## Exercise 2.1

```scheme
(define (make-rat n d)
  (let ((g (gcd n d)))
    (if (< d 0)
        (cons (/ (- n) g) (/ (- d) g))
        (cons (/ n g) (/ d g)))))
```

## Exercise 2.2

```scheme
(define (print-point p)
  (newline)
  (display "(")
  (display (x-point p))
  (display ",")
  (display (y-point p))
  (display ")"))

(define make-point cons)

(define x-point car)

(define y-point cdr)

(define (make-segement start end)
  (cons start end))

(define (start-segement segement)
  (car segement))

(define (end-segement segement)
  (cdr segement))

(define (midpoint-segement segement)
  (make-point (/ (+ (x-point (start-segement segement)) (x-point (end-segement segement))) 2.0)
              (/ (+ (y-point (start-segement segement)) (y-point (end-segement segement))) 2.0)))

(define x (make-point 1 5))
(define y (make-point 5 8))

(define segement (make-segement x y))

(print-point (midpoint-segement segement))
```

## Exercise 2.3

用对角线来表示矩形，只要不论矩形的具体表示方法，只要 `width-rectangle` 和 `height-rectangle` 能够正确的返回矩形的宽和高，周长和面积的计算即为正确的。

```scheme
(define (abs x)
  (if (< x 0)
      (- x)
      x))

(define make-rectangle make-segement)

(define (width-rectangle rectangle)
  (abs (- (x-point (start-segement rectangle)) (x-point (end-segement rectangle)))))

(define (height-rectangle rectangle)
  (abs (- (y-point (start-segement rectangle)) (y-point (end-segement rectangle)))))

(define (perimeter rectangle)
  (* (+ (width-rectangle rectangle) (height-rectangle rectangle)) 2))

(define (area rectangle)
  (* (width-rectangle rectangle) (height-rectangle rectangle)))

(define rec (make-rectangle x y))

(perimeter rec)

(area rec)
```

## Exercise 2.4

很有意思的一道题，这里的 `cons` 返回一个 `procedure`，接收的参数 `m` 也是一个 `procedure`，`(car z)` 相当于把 `(lambda (p q) p)` 作为 m，就变成 `((lambda (p q) p) x y)`

完整的相当于

```scheme
((lambda (m) (m x y)) (lambda (p q) p))
```

```scheme
(define (cons x y)
  (lambda (m) (m x y)))

(define (car z)
  (z (lambda (p q) p)))

(define (cdr z)
  (z (lambda (p q) q)))
```

## Exercise 2.5

```scheme
(define (cons a b)
  (* (expt 2 a) (expt 3 b)))

(define (car x)
  (if (= 0 (remainder x 2))
      (+ 1 (car (/ x 2)))
      0))

(define (cdr x)
  (if (= 0 (remainder x 3))
      (+ 1 (cdr (/ x 3)))
      0))
```

## Exercise 2.6

看不懂

https://www.wikiwand.com/zh-cn/%E9%82%B1%E5%A5%87%E6%95%B0

lamba 演算，暂时不想了解，多学习之后再说吧

## Exercise 2.7

```scheme
(define (make-interval a b) (cons a b))

(define (lower-bound interval) (car interval))

(define (upper-bound interval) (cdr interval))
```

## Exercise 2.8

```scheme
(define (sub-interval x y)
  (make-interval (- (lower-bound x) (upper-bound y))
                 (- (upper-bound x) (lower-bound y))))
```

## Exercise 2.9

对于区间的加减法，组合 (加法或减法) 区间的宽度就是被加 (或减) 的区间的宽度的函数。对于乘除法而说，则没有这个规律。

## Exercise 2.10

```scheme
(define (div-interval x y)
  (if (< 0 (/ (upper-bound y) (lower-bound y)))
    0
    (mul-interval x
                  (make-interval
                    (/ 1.0 (upper-bound y))
                    (/ 1.0 (lower-bound y))))))  
```

## Exercise 2.11

题目说的 9 种是怎么形成的，无非就是区间大于 0，小于 0，横跨 0，这三种情况，然后两个区间组合的话，就有 9 种了。

## Exercise 2.12

```scheme
(define (make-center-width c w)
   (make-interval (- c w) (+ c w)))

(define (center i)
   (/ (+ (lower-bound i) (upper-bound i)) 2))

(define (width i)
   (/ (- (upper-bound i) (lower-bound i)) 2))

(define (make-center-percent c p)
  (make-center-width c (* c p)))
;center can not be zero
(define (percent i)
  (/ (width i) (center i)))
```

## Exercise 2.13

证明略

## Exercise 2.14-2.16

https://github.com/jiacai2050/sicp/blob/master/exercises/02/2.14_2.16.md

https://www.wikiwand.com/en/Interval_arithmetic

## Exercise 2.17

```scheme
(define (last-pair items)
  (if (null? (cdr items))
      items
      (last-pair (cdr items))))
```

## Exercise 2.18

```scheme
(define (reverse items)
  (if (null? items)
      '()
      (append (reverse (cdr items)) (list (car items)))))
```

## Exercise 2.19

```scheme
(define (first-denomination coin-values)
    (car coin-values))

(define (except-first-denomination coin-values)
    (cdr coin-values))

(define (no-more? coin-values)
    (null? coin-values))

(define (cc amount coin-values)
    (cond ((= amount 0)
            1)
          ((or (< amount 0) (no-more? coin-values))
            0)
          (else
            (+ (cc amount
                   (except-first-denomination coin-values))
               (cc (- amount
                      (first-denomination coin-values))
                   coin-values)))))
```

不会影响

## Exercise 2.20

```scheme
(define (same-parity first . others)
  (define (iter items res)
    (cond ((null? items) (reverse res))
          ((= (remainder (car items) 2) (remainder first 2))
           (iter (cdr items) (cons (car items) res)))
          (else (iter (cdr items) res))))
  (iter (cons first others) '()))
```

## Exercise 2.21

```scheme
(define (square-list items)
    (if (null? items)
        '()
        (cons (square (car items))
              (square-list (cdr items)))))

(define (square-list items)
    (map square items))
```

## Exercise 2.22

https://sicp.readthedocs.io/en/latest/chp2/22.html

解释如上

```scheme
(define (square-list items)
    (define (iter things answer)
        (if (null? things)
            (reverse answer) ;modified
            (iter (cdr things)  
                  (cons (square (car things))
                        answer))))
    (iter items '()))
```

## Exercise 2.23

```scheme
(define (for-each f items)
  (cond ((not (null? items))
         (f (car items))
        (for-each f (cdr items)))))
```

## Exercise 2.24

画图

```scheme
(1 (2 (3 4)))
```

## Exercise 2.25

```scheme
(define a (list 1 3 (list 5 7) 9))

(car (cdr (car (cdr (cdr a)))))
(cadr (caddr a))


(define b (list (list 7)))

(car (car b))
(caar b)

(define c (list 1 (list 2(list 3 (list 4(list 5(list 6 7)))))))
(cadr (cadr (cadr (cadr (cadr (cadr c))))))
```

## Exercise 2.26

略

## Exercise 2.27

```scheme
(define (deep-reverse items)
  (define (iter things answer)
    (cond ((null? things) answer)
          ((not (pair? (car things)))
           (iter (cdr things) (cons (car things) answer)))
          (else (iter (cdr things) (cons (deep-reverse (car things)) answer)))))
  (iter items nil))

(define x (list (list 1 2) (list 3 4)))

> (deep-reverse x)
((4 3) (2 1))
> (deep-reverse (list (list 1 2) (list 3 4) (list 5 6)))
((6 5) (4 3) (2 1))
```

## Exercise 2.28

```scheme
(define (fringe tree)
  (cond ((null? tree) nil)
        ((not (pair? tree)) (list tree))
        (else (append (fringe (car tree))
                 (fringe (cdr tree))))))

> (fringe x)
(1 2 3 4)
> (fringe (list x x))
(1 2 3 4 1 2 3 4)
```

## Exercise 2.29

```scheme
(define (make-mobile left right)
  (list left right))

(define (make-branch length structure)
  (list length structure))

;Q(a)
(define (left-branch mobile)
  (car mobile))

(define (right-branch mobile)
  (cadr mobile))

(define (branch-length branch)
  (car branch))

(define (branch-structure branch)
  (cadr branch))

;Q(b)
(define (branch-weight branch)
    (if (pair? (branch-structure branch));the branch's structure is a mobile
        (total-weight (branch-structure branch))
        (branch-structure branch)))

(define (total-weight mobile)
  (+ (branch-weight (left-branch mobile))
     (branch-weight (right-branch mobile))))
         
(define mobile (make-mobile (make-branch 10 25)
                                  (make-branch 5 20)))


> (total-weight mobile)
45
;Q(c)
(define (torque branch)
  (* (branch-length branch)
     (branch-weight branch)))

(define (balance-branch? branch)
  (if (pair? (branch-structure branch))
      (balance? (branch-structure branch))
      #t))
  
(define (balance? mobile)
  (and (= (torque (left-branch mobile))
          (torque (right-branch mobile)))
       (balance-branch? (left-branch mobile))
       (balance-branch? (right-branch mobile))))
     
  
(define balance-mobile (make-mobile (make-branch 10 10)
                                          (make-branch 10 10)))

(define unbalance-mobile (make-mobile (make-branch 0 0)
                                            (make-branch 10 10)))

(define mobile-with-sub-mobile (make-mobile (make-branch 10 balance-mobile)
                                                  (make-branch 10 balance-mobile)))

> (balance? balance-mobile)
#t
> (balance? unbalance-mobile)
#f
> (balance? mobile-with-sub-mobile)
#t


;Q(d)
;only nedd to change the selector and constructor
```

## Exercise 2.30

```scheme
(define (square-tree tree)
  (cond ((null? tree) nil)
        ((not (pair? tree)) (square tree))
        (else (cons (square-tree (car tree))
                    (square-tree (cdr tree))))))

(define tree (list 1
                   (list 2 (list 3 4) 5)
                   (list 6 7)))

>(square-tree tree)
(1 (4 (9 16) 25) (36 49))

(define (square-tree-map tree)
  (map (lambda (sub-tree)
         (if (pair? sub-tree)
             (square-tree-map sub-tree)
             (square sub-tree)))
       tree))

>(square-tree-map tree)
(1 (4 (9 16) 25) (36 49))
```

## Exercise 2.31

```scheme
;Def 1
(define (tree-map f tree)
    (cond ((null? tree) nil)
        ((not (pair? tree)) (f tree))
        (else (cons (tree-map f (car tree))
                    (tree-map f (cdr tree))))))
;Def 2
(define (tree-map f tree)
    (map (lambda (sub-tree)
             (if (pair? sub-tree)
                 (tree-map f sub-tree) 
                 (f sub-tree)))
         tree))
```

## Exercise 2.32

```scheme
(define (subsets s)
  (if (null? s)
      (list nil)
      (let ((rest (subsets (cdr s))))
        (append rest (map (lambda (x) (cons (car s) x)) rest)))))
```

## Exercise 2.33

```scheme
(define (map p sequence)
  (accumulate (lambda (x y) (cons (p x) y)) nil sequence))

(define (append seq1 seq2)
  (accumulate cons seq2 seq1))

(define (length sequence)
  (accumulate (lambda (x y) (+ 1 y)) 0 sequence))
```

## Exercise 2.34

```scheme
(define (horner-eval x coefficient-sequence)
  (accumulate (lambda (this-coeff higher-terms)
                (+ (* higher-terms x) this-coeff))
              0
              coefficient-sequence))
```

## Exercise 2.35

```scheme
(define (count-leaves t)
  (accumulate +
              0
              (map (lambda (sub-tree)
                     (if (pair? sub-tree)
                         (count-leaves sub-tree)
                         1)) t)))
```

## Exercise 2.36

```scheme
(define (accumulate-n op init seqs)
  (if (null? (car seqs))
      nil
      (cons (accumulate op
                        init
                        (map (lambda (seq) (car seq)) seqs))
            (accumulate-n op
                          init
                          (map (lambda (seq) (cdr seq)) seqs)))))
```

## Exercise 2.37

```scheme
(define (dot-product v w)
  (accumulate + 0 (map * v w)))

(define (matrix-*-vector m v)
  (map (lambda (row) (dot-product v row)) m))

(define (transpose mat)
  (accumulate-n cons nil mat))

(define (matrix-*-matrix m n)
  (let ((cols (transpose n)))
    (map (lambda (w) (matrix-*-vector cols w)) m)))

(define m1 (list (list 1 2 3 4)
                (list 4 5 6 6)
                (list 6 7 8 9)))

(define m2 (list (list 1 2 3)
                (list 4 5 6)
                (list 7 8 9)))

(define v (list 1 2 3 4))

(matrix-*-vector m1 v)
;Value: (30 56 80)

(transpose m1)
;Value: ((1 4 6) (2 5 7) (3 6 8) (4 6 9))

(matrix-*-matrix m2 m2)
;Value: ((30 36 42) (66 81 96) (102 126 150))
```

## Exercise 2.38

要求 `op` 参数，也即是传入的操作函数必须符合结合律和交换律

## Exercise 2.39

```scheme
(define (reverse sequence)
  (fold-right (lambda (x y) (append y (list x))) nil sequence))
```
```scheme
(define (reverse sequence)
  (fold-left (lambda (x y) (cons y x)) nil sequence))
```

## Exercise 2.40

```scheme
(define (unique-pairs n)
  (flatmap (lambda (i) (map (lambda (j) (list i j))
                            (enumerate-interval 1 (- i 1))))
           (enumerate-interval 1 n)))


(define (prime-sum-pairs n)
    (map make-pair-sum
         (filter prime-sum? (unique-pairs n))))
```

## Exercise 2.41

```scheme
;Def 1 to generate triples
(define (triples n)
  (flatmap (lambda (i) (flatmap (lambda (j) (map (lambda (k) (list i j k))
                                            (enumerate-interval 1 (- j 1))))
                            (enumerate-interval 1 (- i 1))))
             (enumerate-interval 1 n)))
;Def 2 to generate triples, using unique-pairs in Exercise 2.40
(define (triples n)
  (flatmap (lambda (i) (map (lambda (j) (cons i j))
                            (unique-pairs (- i 1))))
           (enumerate-interval 1 n)))


(define (n-triples-sum-s n s)
  (filter (lambda (triple)(= s (+ (car triple)
                                  (cadr triple)
                                  (cadr (cdr triple)))))
          (triples n)))
```

## Exercise 2.42

```scheme
(define (queens board-size)
  (define (queen-cols k)
    (if (= 0 k)
        (list empty-board)
        (filter
         (lambda (positions) (safe? k positions))
         (flatmap
          (lambda (rest-of-queens)
            (map (lambda (new-row)
                   (adjoin-position
                    new-row k rest-of-queens))
                 (enumerate-interval 1 board-size)))
          (queen-cols (- k 1))))))
  (queen-cols board-size))

(define empty-board nil)

(define (adjoin-position new-row k rest-of-queens)
  (cons new-row rest-of-queens))

(define (abs x)
  (if (< 0 x)
      (- x)
      x))

(define (safe? k positions)
  (let ((row-in-kth-col (car positions)))
    (define (iter col positions)
      (if (= col 0)
        #t
        (and (iter (- col 1) (cdr positions))
             (not (= (abs (- col k)) (abs (- (car positions) row-in-kth-col))));不在对角线
             (not (= (car positions) row-in-kth-col));不在同一行
             )
        ))
    (iter (- k 1) (cdr positions))));check k-1 cols
```

## Exercise 2.43

Exercise 2.42 的 `queens` 函数对于每个棋盘 `(queen-cols k)`，使用 `enumerate-interval` 产生 `board-size` 个棋盘。而 Louis 的 `queens` 函数对于 `(enumerate-interval 1 board-size)` 的每个 `k`，都要产生 `(queen-cols (- k 1))` 个棋盘。因此，Louis 的 `queens` 函数的运行速度大约是原来 `queens` 函数的 `board-size` 倍，也即是 `T * board-size`。

参考：https://sicp.readthedocs.io/en/latest/chp2/43.html

## Exercise 2.44

```scheme
(define (up-split painter n)
  (if (= n 0)
      painter
      (let ((smaller (up-split painter (- n 1))))
        (below painter (beside smaller smaller)))))
```

## Exercise 2.45

```scheme
(define (split big-combiner small-combiner)
  (define (helper painter n)
    (if (= n 0)
      painter
      (let ((smaller (helper painter (- n 1))))
        (big-combiner painter (small-combiner smaller smaller)))))
  helper)
```

## Exercise 2.46

```scheme
(define (make-vect xcor ycor)
  (cons xcor ycor))

(define (xcor-vect vect)
  (car vect))

(define (ycor-vect vect)
  (cdr vect))

(define (add-vect vect1 vect2)
  (make-vect (+ (xcor-vect vect1) (xcor-vect vect2))
             (+ (ycor-vect vect1) (ycor-vect vect2))))

(define (sub-vect vect1 vect2)
    (make-vect (- (xcor-vect vect1) (xcor-vect vect2))
               (- (ycor-vect vect1) (ycor-vect vect2))))

(define (scale-vect s vect)
  (make-vect (* s (xcor-vect vect))
             (* s (ycor-vect vect))))
```

## Exercise 2.47

```scheme
(define (make-frame origin edge1 edge2)
  (list origin edge1 edge2))

(define (origin-frame frame)
  (car frame))

(define (edge1-frame frame)
  (cadr frame))

(define (edge2-frame frame)
  (caddr frame))
```

## Exercise 2.48

```scheme
(define (make-segement start end)
  (cons start end))

(define (start-segement segement)
  (car segement))

(define (end-segement segement)
  (cdr segement))
```

## Exercise 2.49

这题要运行的话需要使用 `sicp-pict` 包里的 `segments->painter`，和 `make-vect` 以及 `make-segment`。不要使用前面几个练习中自己定义的，以及书本中的 `segments->painter`。

```scheme
(define left-bottom (make-vect 0 0))
(define left-top (make-vect 0 1))
(define right-bottom (make-vect 1 0))
(define right-top (make-vect 1 1))

(define mid-bottom (make-vect 0 0))
(define mid-top (make-vect 0 1))
(define mid-left (make-vect 1 0))
(define mid-right (make-vect 1 1))

;Q(a)

(paint (segments->painter (list (make-segment left-bottom left-top)
                                 (make-segment left-top right-top)
                                 (make-segment right-top right-bottom)
                                 (make-segment right-bottom left-bottom))))

(newline)

;Q(b)
(paint (segments->painter (list (make-segment left-bottom right-top)
                         (make-segment left-top right-bottom))))

(newline)

;Q(c)
(paint (segments->painter (list (make-segment mid-left mid-top)
                         (make-segment mid-top mid-right)
                         (make-segment mid-right mid-bottom)
                         (make-segment mid-bottom mid-left))))


;Q(d)
;把wave的各个条线表示出来，太复杂了，不画了
;可以参考 https://sicp.readthedocs.io/en/latest/chp2/49.html
```

## Exercise 2.50

```scheme
(define (flip-horiz painter)
  (transform-painter painter
                     (make-vect 1.0 0)
                     (make-vect 0 0)
                     (make-vect 1 1)))


(define (rotate180 painter)
  (transform-painter painter
                     (make-vect 1 1)
                     (make-vect 0 1)
                     (make-vect 1 0)))

(define (rotate270 painter)
  (transform-painter painter
                     (make-vect 0 1)
                     (make-vect 0 0)
                     (make-vect 1 1)))
```

## Exercise 2.51

```scheme
;Def 1
(define (below painter1 painter2)
  (let ((split-point (make-vect 0 0.5)))
    (let ((paint-top
           (transform-painter
            painter1
            (make-vect 0 0)
            (make-vect 1 0)
            split-point))
          (paint-bottom
           (transform-painter
            painter2
            split-point
            (make-vect 1 0.5)
            (make-vect 0 1))))
      (lambda (frame)
        (paint-top frame)
        (paint-bottom frame)))))
;Def 2
(define (below painter1 painter2)
    (lambda (frame)
        ((flip-horiz
            (rotate90
                (beside
                    (rotate270
                        (flip-horiz painter1))
                    (rotate270
                        (flip-horiz painter2)))))
         frame)))
```

## Exercise 2.52

```scheme
;Q(b)
(define (corner-split painter n)
    (if (= n 0)
        painter
        (let ((up (up-split painter (- n 1)))
              (right (right-split painter (- n 1)))
              (corner (corner-split painter (- n 1))))
            (beside (below painter up)
                    (below right corner)))))
;Q(c)
(define (square-limit painter n)
    (let ((combine4 (square-of-four identity flip-horiz)
                                    flip-vect rotate180))
        (combine4 (corner-split painter n))))
```

## Exercise 2.53

```scheme
> (list 'a 'b 'c)
(a b c)
> (list (list 'geroge))
((geroge))
> (cdr '((x1 x2) (y1 y2)))
((y1 y2))
> (cadr '((x1 x2) (y1 y2)))
(y1 y2)
> (pair? (car '(a short list)))
#f
> (memq 'red '((red shoes) (blue socks)))
#f
> (memq 'red '(red shoes blue socks))
(red shoes blue socks)
```

## Exercise 2.54

```scheme
(define (equal? x y)
    (cond ((and (symbol? x) (symbol? y))
            (symbol-equal? x y))
          ((and (list? x) (list? y))
            (list-equal? x y))
          (else #f)))

(define (symbol-equal? x y)
    (eq? x y))

(define (list-equal? x y)
    (cond ((and (null? x) (null? y))
            #t)
          ((or (null? x) (null? y))
            #f)
          ((equal? (car x) (car y))
            (equal? (cdr x) (cdr y)))
          (else #f)))
```

## Exercise 2.55

根据 97 页的注释 100，符号 `'` 在求值时会被替换成 `quote` 特殊形式，因此，求值：

```scheme
(car ''abracadabra)
```

实际上就是求值：

```scheme
(car '(quote abracadabra))
```

因此 `car` 取出的是第一个 `quote` 的 `car` 部分，而这个 `car` 部分就是 `'quote`，所以返回值就是 `quote`

## Exercise 2.56

```scheme
(define (exponentiation? exp)
  (and (pair? exp)
       (eq? (car exp) '**)))


(define (base exp)
  (cadr exp))

(define (exponent exp)
  (caddr exp))

(define (make-exponentiation base exponent)
  (cond ((=number? exponent 0) 1)
        ((=number? exponent 1) base)
        (else (list '** base exponent))))


(define (deriv exp var)
  (cond ((number? exp) 0)
        ((variable? exp) (if (same-variable? exp var) 1 0))
        ((sum? exp) (make-sum (deriv (addend exp) var)
                              (deriv (augend exp) var)))
        ((product? exp) (make-sum
                         (make-product (multiplier exp)
                                       (deriv (multiplicand exp) var))
                         (make-product (deriv (multiplier exp) var)
                                       (multiplicand exp))))
        ((exponentiation? exp) (make-product
                                (make-product
                                (exponent exp)
                                 (make-exponentiation (base exp) (make-sum (exponent exp) -1)))
                                 (deriv (base exp) var)))))
```

## Exercise 2.57

```scheme
#lang sicp
(define (deriv exp var)
  (cond ((number? exp) 0)
        ((variable? exp) (if (same-variable? exp var) 1 0))
        ((sum? exp) (make-sum (deriv (addend exp) var)
                              (deriv (augend exp) var)))
        ((product? exp) (make-sum
                         (make-product (multiplier exp)
                                       (deriv (multiplicand exp) var))
                         (make-product (deriv (multiplier exp) var)
                                       (multiplicand exp))))
        ((exponentiation? exp) (make-product
                                (make-product
                                (exponent exp)
                                 (make-exponentiation (base exp) (make-sum (exponent exp) -1)))
                                 (deriv (base exp) var)))))


(define (variable? x)
  (symbol? x))

(define (same-variable? v1 v2)
  (and (variable? v1)
       (variable? v2)
       (eq? v1 v2)))


(define (exponentiation? exp)
  (and (pair? exp)
       (eq? (car exp) '**)))


(define (base exp)
  (cadr exp))

(define (exponent exp)
  (caddr exp))

(define (make-exponentiation base exponent)
  (cond ((=number? exponent 0) 1)
        ((=number? exponent 1) base)
        (else (list '** base exponent))))

(define (sum? x) (and (pair? x) (eq? (car x) '+)))

(define (addend s) (cadr s))

(define (augend s)
    (let ((tail-operand (cddr s)))
        (if (single-operand? tail-operand)
            (car tail-operand)
            (apply make-sum tail-operand))))


(define (make-sum a1 . a2)
    (if (single-operand? a2)
        (let ((a2 (car a2)))
            (cond ((=number? a1 0)
                    a2)
                  ((=number? a2 0)
                    a1)
                  ((and (number? a1) (number? a2))
                    (+ a1 a2))
                  (else
                    (list '+ a1 a2))))
        (cons '+ (cons a1 a2))))


(define (single-operand? x) (= 1 (length x)))

(define (=number? exp num)
  (and (number? exp) (= exp num)))

(define (make-product m1 . m2)
    (if (single-operand? m2)
        (let ((m2 (car m2)))
            (cond ((or (=number? m1 0) (=number? m2 0))
                    0)
                  ((=number? m1 1)
                    m2)
                  ((=number? m2 1)
                    m1)
                  ((and (number? m1) (number? m2))
                    (* m1 m2))
                  (else
                    (list '* m1 m2))))
        (cons '* (cons m1 m2))))

(define (product? x)
    (and (pair? x)
         (eq? (car x) '*)))

(define (multiplier p)
    (cadr p))

(define (multiplicand p)
    (let ((tail-operand (cddr p)))
        (if (single-operand? tail-operand)
            (car tail-operand)
            (apply make-product tail-operand))))


(deriv '(* x y (+ x 3)) 'x)
```

## Exercise 2.58

Q(a)：

```scheme
(define (sum? x)
  (and (pair? x) (eq? (cadr x) '+)))

(define (addend s) (car s))

(define (augend s) (caddr s))

(define (make-sum a1 a2)
  (cond ((=number? a1 0) a2)
        ((=number? a2 0) a1)
        ((and (number? a1) (number? a2))
         (+ a1 a2))
        (else
         (list a1 '+ a2))))

(define (make-product m1 m2)
  (cond ((or (=number? m1 0) (=number? m2 0)) 0)
        ((=number? m1 1) m2)
        ((=number? m2 1) m1)
        ((and (number? m1) (number? m2)) (* m1 m2))
        (else (list m1 '* m2))))

(define (product? x)
    (and (pair? x)
         (eq? (cadr x) '*)))

(define (multiplier p) (car p))

(define (multiplicand p) (caddr p))
```

Q(b)：

如果允许使用标准代数写法的话，那么我们就没办法只是通过修改谓词、选择函数和构造函数来达到正确计算求导的目的，因为这必须要修改 `deriv` 函数，提供符号的优先级处理功能。

比如说，对于输入 `x + y * z`，有两种可能的求导顺序会产生 (称之为二义性文法)，一种是 `(x + y) * z`，另一种是 `x + (y * z)`；对于求导计算来说，后一种顺序才是正确的，但是这种顺序必须通过修改 `deriv` 来提供，只是修改谓词、选择函数和构造函数是没办法达到调整求导顺序的目的的。

## Exercise 2.59

```scheme
(define (union-set set1 set2)
  (cond ((null? set1) set2)
        ((null? set2) set1)
        ((element-of-set? (car set1) set2) (union-set (cdr set1) set2))
        (else (cons (car set1) (union-set (cdr set1) set2)))))
```

## Exercise 2.60

```scheme
(define (element-of-set? x set)
  (cond ((null? set) #f)
        ((equal? x (car set)) #t)
        (else (element-of-set? x (cdr set)))))

(define (adjoin-set x set)
  (cons x set))

(define (union-set set1 set2)
  (append set1 set2))

(define (intersection-set set1 set2)
  (cond ((or (null? set1) (null? set2)) '())
        ((element-of-set? (car set1) set2) (cons (car set1) (intersection-set (cdr set1) set2)))
        (else (intersection-set (cdr set1) set2))))
```

空间换时间，在取交集上没有优势，但如果数据本身就很少有重复，那么用这种方式较好。

## Exercise 2.61

```scheme
(define (adjoin-set x set)
  (cond ((null? set) (list x))
        ((= x (car set)) set)
        ((< x (car set)) (cons x set))
        ((> x (car set)) (cons (car set) (adjoin-set x (cdr set))))))
```

## Exercise 2.62

```scheme
(define (union-set set1 set2)
  (cond ((null? set1) set2)
        ((null? set2) set1)
        ((let ((x1 (car set1))
               (x2 (car set2)))
           (cond ((= x1 x2)
                  (cons x1 (union-set (cdr set1) (cdr set2))))
                 ((< x1 x2)
                  (cons x1 (union-set (cdr set1) set2)))
                 ((< x2 x1)
                  (cons x2 (union-set set1 (cdr set2)))))))))
```

## Exercise 2.63

两者都是中序遍历，二叉排序树的中序遍历就是递增顺序的，故而尽管树的形状不同，但是中序遍历的结果都相同。

`tree->list-1` 是树形递归，`tree->list-2` 也是树形递归，它们都需要访问每个节点一次，所以说它们的时间复杂度是一样的。但是由于 `tree->list-1` 中用了 append 过程，append 过程本身是 `O(n)`，所以 `tree->list-1` 的时间要长一些。

## Exercise 2.64

Q(a)：

`partial-tree` 的工作原理为，先找出中间的元素来做根节点，然后依次递归调用求出左子树与右子树，最后调用 `make-tree` 把这三者组合起来。

Q(b)：

这个问题问的是 `list->tree` 的时间复杂度。`partial-tree` 这里其实只是对每个节点访问了一次，`make-tree` 本身是 `O(1)` 的所以 `list->tree` 是 `O(n)` 的。

## Exercise 2.65

1. 用 `tree->list-2` 把树转为有序列表
2. 用有序列表的 `union-set` 与 `intersection-set` 方法
3. 用 `list->tree` 把上面两个方法的结果再转为平衡树

## Exercise 2.66

```scheme
(define (look-up given-key tree)
  (cond
    ((null? tree) #f)
    ((= given-key (car tree)) #t)
    ((> given-key (car tree))
      (look-up given-key (right-branch tree)))
    (else
      (look-up given-key (left-branch tree)))))
```

## Exercise 2.67

```scheme
(define sample-tree (make-code-tree (make-leaf 'A 4)
                  (make-code-tree
                   (make-leaf 'B 2)
                   (make-code-tree
                    (make-leaf 'D 1)
                    (make-leaf 'C 1)))))


(define sample-message '(0 1 1 0 0 1 0 1 0 1 1 1 0))

(decode sample-message sample-tree)
;Value: (A D A B B C A)
```

## Exercise 2.68

```scheme
(define (encode message tree)
  (if (null? message)
      '()
      (append (encode-symbol (car message) tree)
              (encode (cdr message) tree))))


(define (exist? symbol symbols)
      (if (null? symbols)
          #f
          (or (eq? symbol (car symbols))
              (exist? symbol (cdr symbols)))))


(define (encode-symbol symbol tree)
  (cond ((null? tree) (error "bad symbol :ENCODE-SYMBOL" symbol))
        ((leaf? tree) nil)
        (else (cond ((exist? symbol (symbols (left-branch tree)))
                     (cons 0 (encode-symbol symbol (left-branch tree))))
                    ((exist? symbol (symbols (right-branch tree)))
                     (cons 1 (encode-symbol symbol (right-branch tree))))
                    (else (error "bad symbol :ENCODE-SYMBOL" symbol))))))
        
              
(encode '(A D A B B C A) sample-tree)
;Value: (0 1 1 0 0 1 0 1 0 1 1 1 0)
```

## Exercise 2.69

```scheme
(define (generate-huffman-tree pairs)
  (successive-merge (make-leaf-set pairs)))

(define (successive-merge set)
  (if (= 1 (length set))
      (car set)
      (successive-merge (adjoin-set (make-code-tree (car set)
                                                    (cadr set))
                                    (cddr set)))))

(define test-tree (generate-huffman-tree (list (cons 'A 4)
                                               (cons 'B 2)
                                               (cons 'C 1)
                                               (cons 'D 1))))

(decode (encode '(A D A B B C A) test-tree) test-tree)
;Value: (A D A B B C A)
```

## Exercise 2.70

```scheme
(define alphabet (list (cons 'A 2)
                       (cons 'GET 2)
                       (cons 'SHA 3)
                       (cons 'WAH 1)
                       (cons 'BOOM 1)
                       (cons 'JOB 2)
                       (cons 'NA 16)
                       (cons 'YIP 9)))


(define songs-tree (generate-huffman-tree alphabet))

(define code (encode '(GET A JOB 
          SHA NA NA NA NA NA NA NA NA
          GET A JOB
          SHA NA NA NA NA NA NA NA NA
          WAH YIP YIP YIP YIP YIP YIP YIP YIP YIP
          SHA BOOM) songs-tree))

(decode code songs-tree)

(length code)
;Value 84
```

变长 huffman 编码为 84 位二进制，定长编码 8 个字符需要用 3 位二进制，故需要 3*36 = 108 二进制。

## Exercise 2.71

For the most frequent symbol need just `1` bit，for the least frequent symbol need `n-1` bit。

## Exercise 2.72

编码字符的次数为 nn，那么对最频繁出现的字符进行编码的复杂度为 `Θ(n)`，而对最不频繁出现的字符进行编码的复杂度为 `Θ(n^2)`。

## Exercise 2.73

b)

```scheme
(define (install-sum-package)
    ;;; internal procedures 
    (define (addend s)
        (car s))

    (define (augend s)
        (cadr s))

    (define (make-sum x y)
        (cond ((=number? x 0)
                y)
              ((=number? y 0)
                x)
              ((and (number? x) (number? y))
                (+ x y))
              (else
                (attach-tag '+ x y))))

    ;;; interface to the rest of the system
    (put 'addend '+ addend)
    (put 'augend '+ augend)
    (put 'make-sum '+ make-sum)

    (put 'deriv '+
        (lambda (exp var)
            (make-sum (deriv (addend exp) var)
                      (deriv (augend exp) var))))
'done)

(define (make-sum x y)
    ((get 'make-sum '+) x y))

(define (addend sum)
    ((get 'addend '+) (contents sum)))

(define (augend sum)
    ((get 'augend '+) (contents sum)))
```

## Exercise 2.74

略

## Exercise 2.75
```scheme
(define (make-from-mag-ang x y)
  (define (dispatch op)
    (cond ((eq? op 'magnitude) x)
          ((eq? op 'angle) y)
          ((eq? op 'real-part) (* x (cos y)))
          ((eq? op 'imag-part) (* x (sin y)))
          (else (error "Unknown op: MAKE-FROM-MAG-ANG" op))))
  dispatch)
```

## Exercise 2.76

`explicit dispatch` 在增加新操作时需要使用者避免命名冲突，而且每当增加新类型时，所有通用操作都需要做相应的改动，这种策略不具有可加性，因此无论是增加新操作还是增加新类型，这种策略都不适合。

`data-directed style` 可以很方便地通过包机制增加新类型和新的通用操作，因此无论是增加新类型还是增加新操作，这种策略都很适合。

`message- passing-style` 将数据对象和数据对象所需的操作整合在一起，因此它可以很方便地增加新类型，但是这种策略不适合增加新操作，因为每次为某个数据对象增加新操作之后，这个数据对象已有的实例全部都要重新实例化才能使用新操作。

## Exercise 2.78
之后的例题代码需要完成第三章，等到时候在做。
