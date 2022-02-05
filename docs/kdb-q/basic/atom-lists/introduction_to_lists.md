# Introduction to lists

* list is 2nd data structure in q, 1st is atom
* can be defined for all data types

```q
q) l: 1 2 3 4               / default type is float
q) type l                   
7h                          / + means list, 7 means list of floats
q) s:`q`w`e`r`t`y           
q) type s
11h                         / list of symbols
```

* useful function to creating list is `til`
** `til n` returns list of long integers starting from `0` and ending with `n-1`

```q
q) l2: til 5
q) l2
0 1 2 3 4
```

## Arithmetic and logical operations on lists

* arithmetic and logical operation work atom by atom in a list

```q
q) l2+10
10 11 12 13 14              / 10 is added to all atoms of list
q) l2:+5                    / compound assignment to make changes in place
q) 5 6 7 8 9
q) l2<7
11000b                      / returns boolean list, 1b at indexes where expression is  true, this is also evaluated atom by atom
```

## Types of lists

### Simple list
* atoms in the list are of same types
* example: lists created by `til`

```q
q) l: til 4
q) l
0 1 2 3
q) type l
7h
q) type each l
-7 -7 -7 -7h
```

### Empty list

* can be created using `()` with no type
* can also be forced to typed

```q
q) l3:()
q) type l3
0h
q) l4:`int$()
q) type l4
6h
```

### Mixed list
* contains entities[atoms, lists, etc.] of different types

```q
q) l5: (`q;23i;9.8f;"hello")
q) l5
`q
23i
9.8000000000000007
"hello"
q) type l5
0h
q) type each l5
-11 -6 -9 10h
```

* equal `=` vs match `~`

```q
q) l6: 1 2 3 4i
q) l7: 1 2 3 4f
q) l6=l7                    / checks underlying values
1111b
q) l6~l7                    / checks types as well
0b
```

### Singleton list

* if we try to assign 1 item to a variable, it will be an atom
* we use function `enlist` to make singleton list: list with single item

```q
q) l8:99
q) type l8
-7h                         / -ve sign means atom
q) l8: enlist 99
q) type l8
7h                          / +ve sign means list
q) type each l8
,-7h
q) l8
,99
q)
````

## Indexing and index assignment

* indexing in list is done using `[]` - list`[`index/es`]`
* indexes starts from `0` and ends with length_of_list-1
* in `[]` we can provide 1 index or list of indexes

```q
q) l9: (1 2; 30 40 50; 77 88; -10 -11)
q) l9
1 2
30 40 50
77 88
-10 -11
q) l9[0]
1 2
q) l9[3]
-10 -11
q) l9[3 0 2]
-10 -11
1   2
77  88
```

* we can also **assign** values to list at index using `[]`

```q
q) l9[0]:99 98
q) l9
99 98
30 40 50
77 88
-10 -11
q) l9[3]:enlist 100
q) l9
99 98
30 40 50
77 88
,100
```

* **indexing** can be done at **depth** as well for nested lists, by providing indexes of each level seperated by `;`
* as we access value, similarly we assign as well
* if we dont specify index of a level that means we want all items of that level

```q
q) l9
99 98
30 40 50
77 88
,100
q) l9[0;1]
98
q) l9[0;1]:909              / type of value should match with type of list
q) l9
99 909
30 40 50
77 88
,100
q) l9[;0]
99 30 77 100                / get first item at all level
q) l9[0;]                   / get all items from first level
99 909
q) l9[0;99]                 / index out of range
0N
q) type l9[0;99]            / return null of type of first element of list
-7h
q) type first l9[0]
7h
```
