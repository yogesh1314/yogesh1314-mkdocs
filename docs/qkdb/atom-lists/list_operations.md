# List Operations

## join `,`

* we can add atom/atoms/lists to list using `,` operator

```q
q) l: til 5
q) l
0 1 2 3 4                   
q) a:10
q) b:200
q) a,b
10 200
q) type a,b
7h
q) l,a
0 1 2 3 4 10
q) l,a,b
0 1 2 3 4 10 200
q) l,:a
q) l
0 1 2 3 4 10
q) l,4.35                   / we can join items of different types to the list
0
1
2
3
4
10
4.35

```

## drop `_`

* (-/+)(n)number_of_elements_to_drop`_`list
	* if -ve, drop from end of list
	* if +ve, drop from start of list
	* if n is greater than length of list; return empty list of type same as first element

```q
q) l
0 1 2 3 4 10
q) 2_l
2 3 4 10
q) -2_l
0 1 2 3
q) -99_l
`long$()
```

* if list is at left side of function `_` then operand at right specifies index (list of indexes wont work)

```q
q) l
0 1 2 3 4 10
q) l _ 2
0 1 3 4 10
q) l _ 2 3
'domain
```

## cut

* atom/list`cut`list
* if left item is atom: cut list in that many equal parts
* if left item is list: cut list at those indexes

```q
q) l
0 1 2 3 4 10 99
q) 2 cut l                  / cut in 2 equal parts, if 1 item is left enlist it, return is always list
0 1
2 3
4 10
,99
q) 3 cut l
0 1 2
3 4 10
,99
q) 2 5 cut l                / first cut at index 2, then at index 5, values shown are from index 2 to 5 then 5 till end of list
2 3 4
10 99
q) 0 2 5 cut l             / in previous expression values from index 0 to 2 were not shown, we need to make cut at index 0 to show values before first non 0 index cut
0 1
2 3 4
10 99
```

## take `#`

* its partner of drop
* (-/+)(n)number_of_elements_to_take`#`list
	* if -ve, take from end of list
	* if +ve, take from start of list
	* if n is greater than length of list; return n elements by repeating list till required, i.e., list if wrapped around

```q
q) l
0 1 2 3 4 10 99
q) 2#l
0 1
q) -2#l
10 99
q) 2#l
0 1
q) 20#l
0 1 2 3 4 10 99 0 1 2 3 4 10 99 0 1 2 3 4 10
```

## sublist

* x `sublist` list
* x can be atom or list
* if x is atom - return x number of elements starting from index 0
	* if x is greater than length of list, return list dont wrap around
* if x is list - return x[1] number of elements starting from x[0]
```q
q) l
0 1 2 3 4 10 99
q) 2 sublist l              / return 2 elements from index 0
0 1
q) 2 5 sublist l            / return 5 elements from index 2
2 3 4 10 99
q) 10 sublist l             / x > count l , return list, no wrap around
0 1 2 3 4 10 99
q) 2 4 5 6 99 100 sublist l / x is list, 2<count x, x[1]=4 return 4 element, starting x[0]=2, ignore rest
2 3 4 10
q) 2 3 5 6 99 100 sublist l
2 3 4
```

## Question mark `?`

### 1. randomise

#### -atom`?`y
atom is x
* y is atom - return x distinct random element from `til y`
* y is list - return x distinct random elements from list y
* y is `n (backtick n), return x distinct symbols of size n
* if x > y  or x > count y, `'length` error

```q
q) -3?5                     / x is -ve, y is +ve, distinct elements
0 2 1
q) -6?5                     / x is -ve, y is +ve, x>y
'length
q) -5?5                     / x is -ve, y is +ve, x=y, randomized til y
1 3 4 2 0
q) -5?til 5                 / x is -ve, x=count y, randomized list y
2 0 3 4 1
q) -6?til 5                 / x is -ve, x>count y
'length
q) -20?`2
`bc`nc`km`fd`hk`ef`dl`mh`pg`pn`cd`dj`hl`pb`go`jl`ib`np`ok`ll
```

#### atom`?`y
* y is atom or list - return random elements from `til y` or list(y)

```q
q) 6?til 5                  / x is +ve, x>count y
4 2 0 1 4 0
q) 5?til 5                  / x is +ve, x=count y
3 4 4 1 3
q) 20?`2
`lb`kj`cd`ok`le`ce`ii`ji`ac`id`kp`ci`ie`kf`en`he`fp`bb`cj`mk
```
### 2. find `?`

#### list`?`element
* return index of element from list
* if not found, return index where it might add i.e., `1+count list`
* if more than 1 element exist in list, `?` returns index of first occurence

```q
q) l
0 1 2 3 4 10 99
q) l?99
6
q) l?101                    / element not in list, return 1+count l
7
q) l?101 102
7 7                         / if more than 1 element not found, return same index i.e.,  1+count l
q) l,:10
q) l
0 1 2 3 4 10 99 10
q) l?10
5
```


## first
* returns first element from list

## last
* returns last element from list

## raze
* flatten out 1 level of nesting from nested list

## distinct
* return distinct element from list

## count
* return count of number of elements in list

## union
* list1 `union` list2 - distinct elements from join of list1 and list2

## inter
* return elements which are in both of them

## except
* list1 `except` list2
* left which are not in right

```q
q) l
0 1 2 3 4 10 99 10
q) l2:5?l
q) l2
99 2 3 2 2
q) l
0 1 2 3 4 10 99 10
q) first l2
99
q) last l
10
q) l3:(1 2 3; 4 5 6; (7;8))
q) l3
1 2 3
4 5 6
7 8
q) raze l3
1 2 3 4 5 6 7 8
q) distinct l
0 1 2 3 4 10 99
q) distinct l2
99 2 3
q) count l
8
q) count l2
5
q) l except l2
0 1 4 10 10
q) l union l2
0 1 2 3 4 10 99
q) l in l2
00110010b
q) where l in l2
2 3 6
q) l where l in l2
2 3 99
```
