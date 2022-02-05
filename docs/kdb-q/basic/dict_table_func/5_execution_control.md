# Execution control

## `if` statement

* syntax `if[condition; expression1; expression2; .. ; expressionN]`
* if `condition` is true execute all expressions till `expressionN`.
* `0` is **false**, non zero values positive or negative are **true**.

```q
q)a:b:c:0
q)if[a>0;b:10;c:20]
q)b
0
q)c
0
q)a:2
q)if[a>0;b:10;c:20]
q)b
10
q)c
20
q)if[-20; show "executed"]
"executed"
q)if[20; show "executed"]
"executed"
q)if[0; show "executed"]
q)
```

## `$` if-else statement

* `$` is used for if-else
* syntax: `$[condition; trueExpr; falseExpr]`
* nested syntax: `$[condition; trueExpr; [condition2; trueExpr2; ...] ... falseExprN]`
** here `falseExprN` will be executed when every `condition1..N` are false
* once a true condition is found, no execution is followed

```q
q)$[a<2; 1; a<4; 2; a<6; 3; a<8; 4; 20]
2
q)a
2
q)r:$[a=2;10;20]
q)r
10
q)$[a<10; [b:99;c:100]; [b:100;c:101]]
100
q)b
99
q)c
100
```
## `?` vector conditional

* syntax `?[vector condition; trueExpr; falseExpr]`
* `trueExpr` and `falseExpr` need to either atom of vector of same length as `vector condition`
* where `vector condition` is true `trueExpr` is returned else `falseExpr` is returned
* we need to be careful with type promotion:
** if `trueExpr` or `falseExpr` are of different type than original list, whole returned list will be converted to `trueExpr` or `falseExpr` type

```q
q)p:10?10
q)p
4 5 4 2 7 8 5 6 4 1                   
q)?[p<5;p;0]                         / where true, return p, else return 0
4 0 4 2 0 0 0 0 4 1                 
q)?[p<5;p;-1f]
4 -1 4 2 -1 -1 -1 -1 4 1f            / whole list converted to float type
q)?[p<5;p;0Nd]
2000.01.05 0N 2000.01.05 2000.01.03 0N 0N 0N 0N 2000.01.05 2000.01.02
```
## `while` loop

* this is not generally used, same operation can be done using adverbs
* try to think of solution in vectorised form to use adverbs than `while`
* syntax: `while[condition; expression/s]`

```q
q)a
2
q)while[a>0; show a; a-:1]
2
1
```

## `do` loop

* syntax: `do[numberOfIterations; expression/s]`
* execute `expression/s` `numberOfIterations` times
* do is useful when we want to time an expression
** `\t do[N; expr]`
** though same can be done using `\t:N expr` - this is not supported in 2.8 version
** useful when we want to see how our expression would scale for thousand or million rows

```q
q)i:0
q)do[10;show i; i+:1]
0
1
2
3
4
5
6
7
8
9
q)a:1000?100f
q)b:1000?100f
q)\t a xexp b
0
q)\t do[1000; a xexp b]
64
q)\t:1000 a xexp b
62
```
