# Operation on dictionaries

## Join and lookup assignments

* we can join values using `,`
* if we want to join 1 key/value pair, we need to enlist them

```q
q) d:`q`w`e!(1 2 3; 4 5 6; 7 8 9)
q) d
q| 1 2 3
w| 4 5 6
e| 7 8 9
q) d,:`r`t!(9 8 7; 10 10 10)
q) d
q| 1  2  3
w| 4  5  6
e| 7  8  9
r| 9  8  7
t| 10 10 10
q) d,:(enlist `y)!(enlist 100)
q) d
q| 1 2 3
w| 4 5 6
e| 7 8 9
r| 9 8 7
t| 10 10 10
y| 100
```
* another way of adding key/value pair is using indexing
	* we assign value to `dictionary[new_key]`
	* this means we can do upsert, update if exists otherwise insert
```q
q)d[`u]: 44 55
q)d
q| 1 2 3
w| 4 5 6
e| 7 8 9
r| 9 8 7
t| 10 10 10
y| 100
u| 44 55
q) d[`y]: 1 1 2 3 5
q) d
q| 1 2 3
w| 4 5 6
e| 7 8 9
r| 9 8 7
t| 10 10 10
y| 1 1 2 3 5
u| 44 55
```

## find `?`

* left argument is dictionary and right argument is value: `dictionary?value(s)`
* if value dont exist, returns null of type of first key
* also works with nested values
```q
q) d2:`a`b`c! 11 22 33
q) d2
a| 11
b| 22
c| 33
q) d2?22
`b
q) d2?22 33
`b`c
q) d2?44
`
q) d3:(`a`b`c)!(10 20; 30 40; 50 60)
q) d3
a| 10 20
b| 30 40
c| 50 60
q) d3?(10 20;50 60)
`a`c
```

## drop `_`

* left argument is key(s) and right argument is dictionary

```q
q) `a _ d3
b| 30 40
c| 50 60
q) `a`b _ d3
c| 50 60
```

## take `#`

* left argument is key(s) and right argument is dictionary
* if key doesn't exist in dictionary, null values are returned

```q
q) `a`c#d3
a| 10 20
c| 50 60
```
## Arithmetic operations

* function applied to dictionary is applied to its values
* if argument of function is dictionary, function is applied where keys match, otherwise values remain as is

```q
q) d4:`a`b`c`d`e!til 5
q) d4
a| 0
b| 1
c| 2
d| 3
e| 4
q) d5:`a`b`d! 10 20 30
q) d5
a| 10
b| 20
d| 30
q) d4*10                    / function applied to all values
a| 0
b| 10
c| 20
d| 30
e| 40
q) d4+d5                    / keys of result is union of both keys with function applied on common keys
a| 10
b| 21
c| 2
d| 33
e| 4
```

## Logical operations

* similar to Arithmetic operations Logical operations also performed on all values
* if arguments are dictionaries, function applied to common keys

```q
q) d4>2
a| 0
b| 0
c| 0
d| 1
e| 1
q) d4>d5                   
a| 0
b| 0
d| 0
c| 1
e| 1
```

## Mathematical operations

* **Average** `avg`
	* operation is performed column-wise
	* if we want average performed by keys we use function `each`

```q
q) d6:(`a`b`c)!(1 2 3; 40 50 60; 100 200 300)
q) d6
a| 1   2   3
b| 40  50  60
c| 100 200 300
q) avg d6                 
47 84 121f
q) avg each d6
a| 2
b| 50
c| 200
```

## Creating a table

* we can create table using function `flip`
* dictionary needs to be column dictionary - meaning count of each value should match and keys as list of symbols 
* value needs to be list not atom, we would need to enlist each of the values or enlist whole dictionary to get table

```q
q) d6
a| 1   2   3
b| 40  50  60
c| 100 200 300
q) flip d6
a b  c
--------
1 40 100
2 50 200
3 60 300
q) type flip d6
98h
q) d7:`a`b`c!til 3
q) d7
a| 0
b| 1
c| 2
q) flip d7
'rank
  [0]  flip d7
       ^
q) enlist d7
a b c
-----
0 1 2
q) flip enlist each d7
a b c
-----
0 1 2

```
