#  `@` and `.`

* `@` and `.` build on the functionality provided by square bracket notation

* If x is a data structure, `x[y]` means 'Index':

```q
q)x
8 1 9 5 4 6 6 1 8 5
q)x[3]
5
```
* If x is a function, `x[y]` means 'Apply':

```q
q)x
{(x*2)+5}
q)x[3]
11
```
* `@` and `.` provides more powerful, flexible and alternatives to square bracket notation
* `@` is used for single-dimensional lists 
* `.` is used for nested lists 
  
# Index using `@` and `.`

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

?[list: (1 2 3;4 5 6; 7 8) - Which of these will return a value of 3]
-[ ] .[list;0 2]
-[ ] list[0][2]
- list 0 2
- @[list;0 2]