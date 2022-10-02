# Adverbs

* used generally to apply a function over a list

## `each` `'`
* used to apply a function to each individual element of a list
```
q) //function to count number of divisors of a number
q) divisors:{count where 0=x mod 1+til x}
q) //works fine with atomic values
q)divisors 5
2
q)divisors 10
4
q)divisors 20
6
q) //doesn't work with list
q)l
12 10 1 90 73 90 43 90 84 63 93 54 38 97 88 58 68 45 2 39
q)divisors l
'type
  [2]  (.q.til)

  [1]  divisors:{count where 0=x mod 1+til x}
q) //we use each to execute divisors function over each element of list
q)divisors each l
6 4 1 12 2 12 2 12 12 6 4 8 4 2 8 4 6 6 2 4
```
* also used for nested lists
```
q)(2 3 4 5; 9 8; 9 8 3 4 56)
2 3 4 5
9 8
9 8 3 4 56
q)count (2 3 4 5; 9 8; 9 8 3 4 56)
3
q)count each (2 3 4 5; 9 8; 9 8 3 4 56)
4 2 5
q)reverse  each (2 3 4 5; 9 8; 9 8 3 4 56)
5 4 3 2
8 9
56 4 3 8 9
q)reverse  (2 3 4 5; 9 8; 9 8 3 4 56)
9 8 3 4 56
9 8
2 3 4 5
```

## each-both `,'`
* it is polyadic form of `each`
* often used for applying diadic function element wise to a pair of lists
```
q)l1
45 84 97 45 38
q)l2
12 45 88 63 2
q)l1,'l2
45 12
84 45
97 88
45 63
38 2
q)l1,l2
45 84 97 45 38 12 45 88 63 2
```
* each-both can also be applied to higher valency functions
```
q){x,y,z}'[1 2 3;"abc";1.1 2.2 3.3]
1 "a" 1.1
2 "b" 2.2
3 "c" 3.3
```

## each-right `/:`
* used with diadic function where we want to use a list as right-hand argument
* join each right hand arguments to left hand argument
* iterate over right
```
q)l:1 2 3
q)l,/:10 20 30
1 2 3 10
1 2 3 20
1 2 3 30
```
## each-left `\:`
* used with diadic function where we want to use a list as right-hand argument
* join each left hand argument to right hand argument
* iterate over left
```
q)l:1 2 3
q)l,\:10 20 30
1 10 20 30
2 10 20 30
3 10 20 30
```
* many inbuilt operator already works on list and atom in same manner - thus if we get desired result with such operators we shouldn't be using adverbs with them
```
q)10 + 1 2 3
11 12 13
q)10 +/: 1 2 3
11 12 13
```

## each-prior `':`
* used to apply a diadic function to a list element and its predecessor in the list
```
q)// sum each prior - running sum
q)l
1 2 3
q)+':[l]
1 3 5
```
* can also be used to show each element of list alongside its predecessor
```
q){x,y}':[l]
1
2 1
3 2
```
* we can also use optional first parameter - which will be used as first `y` argument in the function
```
q){x,y}':[0;l]
1 0
2 1
3 2
q){x,y}':[10;l]
1 10
2 1
3 2
```
* `deltas` is `-':` minus each prior
```
q)deltas
-':
q)deltas l
1 1 1
q)-':[l]
1 1 1
q)
```
## scan `\` , over `/`
* little complicated
* their behavior depends on valency of function also on data type of arguments
* scan returns results after each iteration of `f`, over returns result only once after result is converged

### monadic function - `f`
* `f\[n;x]` - returns `f{x}`, `f{f{x}}` and so on `n` times
```
q){x+7}\[10;0] // add 7, 10 times, starting with 0
0 7 14 21 28 35 42 49 56 63 70
q){x+7}\[10;5] // add 7, 10 times, starting with 5
5 12 19 26 33 40 47 54 61 68 75
q){x+7}/[10;5] // add 7, 10 times, starting with 5, over-only shows end result
75
```
* we can also use while clause instead of integer `n`
```
q){x+7}\[{x<100};5] // add 7, while result <100, starting with 5
5 12 19 26 33 40 47 54 61 68 75 82 89 96 103
```
* `f/[x]` - if `n` is not provided, iterate till output is same as previous result, or till last 2 results are same
* `f\[x]` - iterate(scan) and `f/[x]` - converge(over)
```
q){x%10}/[5]
0f
q)
q){x%10}\[5]
5
0.5
0.05
0.005
0.0005
5e-005
5e-006
5e-007
5e-008
```
### dyadic function - `f`
* function takes 2 arguments
* `f\[a b c d] = a; f[a;b] f[f[a;b];c] ...`
* or we can also provide initial number x - `f\[x;a b c d] = 100+a; 100+f[a;b]; 100+f[f[a;b];c] ..`
```
q)til 20
0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19
q)+\[til 20]
0 1 3 6 10 15 21 28 36 45 55 66 78 91 105 120 136 153 171 190
q)+\[100;til 20]
100 101 103 106 110 115 121 128 136 145 155 166 178 191 205 220 236 253 271 290
```
### Exercise
* calculate depreciation of an asset over time
* function : `depr:{[c;r] c*1-r%100}`
1. depreciation in 1 year
```
q)depr[1000;10] // starting cost=1000, rate=10%
900f
```
2. depreciation in 5 years or `x` years
```
q)depr[;10]\[5;1000] // starting cost=1000, rate=10%, over 5 years, using monadic version by keeping rate constant
1000
900f
810f
729f
656.1
590.49
```
3. check with different rate
```
q)depr\[1000;3 5 6 8 10] // cost=1000, rates=3 5 6 8 10%
970 921.5 866.21 796.9132 717.2219
```
4. different rate, different cost, 1 year
```
q)depr\[1000 2000 5000 10000;3 5 6 8 10] // cost=1000 2000 5000 10000, rates=3 5 6 8 10%
970      1940     4850     9700
921.5    1843     4607.5   9215
866.21   1732.42  4331.05  8662.1
796.9132 1593.826 3984.566 7969.132
717.2219 1434.444 3586.109 7172.219
```

## Exercise:
* Question:
```
Suppose we are given a vector of numbers (xx) and a step size (s). Write a function that will take x and s as arguments, and output a result vector. The result vector is constructed using the following procedure:
   1. The xx value sets the mid-point of the step (e.g. if xx is 1 and s is 3, then the values -1,0,1,2 and 3 are included in that step range)
   2. If the next xx value is within the current step range, then the step mid-point does not change (e.g. if xx is -1, then the step mid-point is still 1)
   3. If the next xx value falls outside the current step range, the step mid-point is reset to that xx value (e.g. if xx is 4, then the new step mid-point is 4)
   4. The current step mid-point is stored each time in the result vector
Here are two examples to clarify the procedure:
     xx:1 -1 3 0 1 1 5 6 2 10
     s:3
     result:1 1 1 1 1 1 5 5 2 10
     and
     xx:2 3 4 3 20 7 8 31
     s:6
     result:2 2 2 2 20 7 7 31
```
* Answer:
```
We will need to introduce 3 parameters:
   1. x - the step size (this will take the value s)
   2. y - the current step mid-point
   3. z - the next value in the xx vector (we will see later how this may be achieved with an adverb)
If z is within the current step range, it will satisfy: y - x < z < y + x ; this can be rearranged to give: x > abs ( z - y)
So, if x>abs(z-y) is true, then the step mid-point does not change, and the value of y should be saved to the result vector.
Alternatively, if x>abs(z-y) is false, then the step mid-point will change to the value of z, and so z should be saved to the result vector.
The previous two comments can be expressed as an if-else statement:
 $[x>abs z - y; y; z]   /the value returned, will be the step mid-point
Let's digress back to the scan function e.g.
  q){x+y}\[1 2 3 4]
  1 3 6 10
The value returned by the above function is used as the next x value, and the next y value is taken from the next item in the list. This is exactly what we need to do in our problem. Thus, the solution is given by:
 {$[x>abs z - y; y; z]}[s]\[xx]
```
