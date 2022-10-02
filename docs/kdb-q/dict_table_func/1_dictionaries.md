# Dictionaries

* It is 3rd data structure in q after atom and lists
* Its mapping of lists: list of keys and list of values separated by `!`
* Count of keys must be equal to count of values
* Data type is 99

```q
q) l1:`q`w`e
q) l2:`a`s`d
q) l3:`z`x`c
q) d1:(`A`B`C)!(l1;l2;l3)
q) d1
A| q w e
B| a s d
C| z x c
q) type dict
99h
```

## Accessing values in a dictionary

* access is done using key or list of keys with dictionary name

```q
q) d1`A
`q`w`e
q) d1`A`B
q w e
a s d
```

* function `key` or `cols` can be used to view keys of dictionary
* function `value` can be used to view values of dictionary

```q
q) key d1
`A`B`C
q) cols d1
`A`B`C
q) value d1
q w e
a s d
z x c
```

* keys in dictionary **need not be unique**
* trying to get value for repeating key will return value of first match for the key

```q
q) d3:(`q`q`w`e`q)!til 5
q) d3
q| 0
q| 1
w| 2
e| 3
q| 4
q) d3`q
0
```

* dictionary can have nested lists or any q entity as values

```q
q) d4: (`q`w`e)!(((1 2 3);4 5); 1010b; 1 2 3f)
q) d4
q| (1 2 3;4 5)
w| 1010b
e| 1 2 3f
q) d4`q
1 2 3
4 5
```

## Adding or modifying values to dictionary

* we can add new values using join `,` operator
* we can modify values using key as index and assign new value

```q
q)d1,(`P`Q)!(23 56;78 97f)  / changes wont reflect in dictionary
A| `q`w`e
B| `a`s`d
C| `z`x`c
P| 23 56
Q| 78 97f
q)d1,:(`P`Q)!(23 56;78 97f) / compounded assignment to make change in place
q)d1
A| `q`w`e
B| `a`s`d
C| `z`x`c
P| 23 56
Q| 78 97f
q)d1[`P]: 100 102           / new value assigned
q)d1
A| `q`w`e
B| `a`s`d
C| `z`x`c
P| 100 102
Q| 78 97f
```
