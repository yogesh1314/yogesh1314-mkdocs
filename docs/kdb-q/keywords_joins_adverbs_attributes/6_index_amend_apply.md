# Index Amend and Apply 

## Operators `@` and `.`

* `@` and `.` build on the functionality provided by square bracket notation

* If `x` is a data structure, `x[y]` means 'Index' 

```q
q)x
8 1 9 5 4 6 6 1 8 5
q)x[3]
5
```
* If `f` is a function, `f[y]` means 'Apply':

```q
q)f
{(x*2)+5}
q)f[3]
11
```
* `@` and `.` provides more powerful, flexible and alternatives to square bracket notation
* `@` is used for single-dimensional lists 
* `.` is used for nested lists 
  
## Index using `@` and `.`

* If list is single-dimensional list and index is 2:
```q
q)list
4 9 2 7 0 1 9 2 1 8
q)index: 2
q)list[index]
2
```
then same can be acheived using `@` operator as:
```q
q)@[list;index]
2
```
* If list is nested list and row is 1 and column is 2:
```q
q)list
1 2 3
4 5
6 7
q)row:0
q)column:1
q)list[row][column]
2
q)list[row;column]
2
```
then same can be acheived using `.` operator as: 
```q
q).[list;(row;column)]
2
q).[list;0 1]
2
```
## Amend/Apply using `@`

* Previously we used `:` operator to amend items in a single-dimensional list

```q
q)list
"ABCDE"
q)list[2]:"Z"
q)list
"ABZDE"
```
* Same can be acheived using `@` for single-dimensional list, however we need to give list name as symbol for changes to be permanent
```q
q)list
"ABZDE"
q)@[list;2;:;"K"]
"ABKDE"
q)list
"ABZDE"
q)@[`list;2;:;"K"]
`list
q)list
"ABKDE"
```
* we can also apply mathematical operator to list
```q
q)list
4 9 2 7 0 1 9 2 1 8
q)@[list;3;+;66]
4 9 2 73 0 1 9 2 1 8
q)list:(8 4 3;9 8 76;7 8)
q)list
8 4 3
9 8 76
7 8
q).[list;1 2;+;100]
8 4 3
9 8 176
7 8

```
* we can also update more than 1 item in the list
```q
q)list
4 9 2 7 0 1 9 2 1 8
q)@[list;0 1 2;:;44 55 66]
44 55 66 7 0 1 9 2 1 8
```

## `@` vs `[]`
* square brackets are easy to read for simple use cases, but there are cases where `@` can be used where `[]` cannot
1.  `[]` can only be used with named variables but `@` can be used without named variables, it can also be used with result of previous operation
```
q)@[0000b;2;:;1b]
0010b
q)@[10#.Q.A;2;:;"H"]
"ABHDEFGHIJ"
``` 
2. `@` has a wider range of operations - square bracket allows us to append to a dictionary item
```
q)d:(`a`b`c)!(1 2;3 4 5;6 7 8 9)
q)d
a| 1 2
b| 3 4 5
c| 6 7 8 9
q)d[`c],:10
q)d
a| 1 2
b| 3 4 5
c| 6 7 8 9 10
```
but `@` also allows us to prepend
```
q)d
a| 1 2
b| 3 4 5
c| 6 7 8 9 10
q)@[d;`c]
6 7 8 9 10
q)@[d;`c;{y,x};55]
a| 1 2
b| 3 4 5
c| 55 6 7 8 9 10
```
3. we can use `@`  with non-dyadic operators
```
q)@["avcdf";4;upper]
"avcdF"
```

* to use a function taking more than 2 arguments, we create a monadic projection by fixing the other arguments inside the 3rd part of `@` expression
```
q)applyInterest:{[x;nperiods;rate] nperiods{x*1+y%100}[;rate]/x}
q)account:`ram`bheem!12.33 2033.56
q)account
ram  | 12.33
bheem| 2033.56
q)@[account;`bheem;applyInterest[;2;5.5]] // fix nperiods to 2 and rate to 5.5
ram  | 12.33
bheem| 2263.403
```

## Advanced use of `@` index

* Setup 2 structures:
  1.  l is a mixed list of symbols and integers
  2.  m is a mapping of symbol to integers
* we can apply the mapping at the indexes of the symbols
```
q)l: (`a;1;2;`b;5;90)
q)l
`a
1
2
`b
5
90
q)m:`a`b!500 600
q)m
a| 500
b| 600
q)@[l; where -11=type each l;m]
500 1 2 600 5 90
```

## Using `.` with database tables

* we can use `.` to index into a table
```q
q)t:([sym:`MSFT`IBM] prc: 10.29 24.66; size: 100 200)
q)t
sym | prc   size
----| ----------
MSFT| 10.29 100
IBM | 24.66 200
q).[t;(`MSFT;`prc)]
10.29
```

* `.` also allows us to update entire object at once by passing an empty set of indexes - this allows us to amend each element of the list
```
q)list:(1 2;3 4 5;6 7 8 9)
q)list
1 2
3 4 5
6 7 8 9
q).[list;();+;90]
91 92
93 94 95
96 97 98 99
```
## Convert `@` to `.`
* we can convert an `@` statement to `.` statement as `@` is a speacial case of `.`
```
q)list
1 2
3 4 5
6 7 8 9
q)f
{2*x}
q)@[list;1;f]
1 2
6 8 10
6 7 8 9
q).[list;enlist 1;f]
1 2
6 8 10
6 7 8 9
```
## Apply using `@` and `.`
* we have seen that when `f` is a function then `f[x]` means apply
* `[]` can be replaced with `@` (for single argument)and `.` (for multiple arguments)
```
q)f
{2*x}
q)f[2]
4
q)@[f;2]
4
```
* Let's say we have list of lists with varying length and we want to flip it into a table - we can't do it straight away as lenghts are not same
```
q)list:(1 2 3;4 5 ; 7 8 9 0)
q)list
1 2 3
4 5
7 8 9 0
q)flip `a`b`c!list
'length
  [0]  flip `a`b`c!list
       ^
```
* instead we can use `@` to fill the lists with `0N` to make them of same lenght and then flip it into a table 
```
q)@[max[c]#0N;;:;]'[til each c:count each list;list]
1 2 3
4 5
7 8 9 0
q)flip `a`b`c!@[max[c]#0N;;:;]'[til each c:count each list;list]
a b c
-----
1 4 7
2 5 8
3   9
    0
```

## Using `@` on database tables

* we can use `@` as simple version of functional select - here column name is used as symbol so it can be passed programatically
```
q) @[select from trades where sym=`AAPL;`size;sums]
```

## Error trapping

```
q)f
{2*x}
q)g
{x*y}
q)@[f;"ABC";{-2 "Error: ",x;}]
Error: type
q).[g;("ABC";5);{-2 "Error: ",x;}]
Error: type
```