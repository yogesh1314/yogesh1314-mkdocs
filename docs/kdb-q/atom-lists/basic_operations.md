# Basic Operations

## Mathematical Operations

### Add
```py
q)2+6
8
```
### Subtract
```q
q)10-5
5
```
### Divide
`%` is used instead of `/`, as `/` is used for comment
```q
q)10%2
5f        / f means float
```
### Multiply
```q
q)2*7
14
```
### Order of operation
q follows **right to left** order instead of _BODMAS_
```q
q)5*2+3   / 3+2=5*5=25
25


```
Order can be forced using brackets
```q
q)(5*2)+3
13
```

## Assignment
any entity(atom/function/list/table) in q can be assigned to a variable using operator `:`
```q
q)a:5
q)b:6     / values are not displayed in next line as before, they are stored(assigned)
q)c:7
q)d:a+b+c
q)d       / to display value, enter variable name
18
```

## Comparison or Logical operators
Two numbers/entities can be compared using :

* `<` less than
* `>` greater than
* `<=` less than or equal to
* `>=` greater than or equal to
* `=` equal to - only checks underlying value
* `<>` not equal to
* `~` match - checks underlying value and type
```q
q)a<b
1b
q)d<>a
1b
q)c<b
0b         / 1b stand for true, 0b for false
q)5.0=5
1b
q)5.0~5
0b
```

## Compound and Multiple Assignment
### Compound Assignment
```q
q)x:10
q)x:x+2     / add 2 to x and store the result in x
q)x
12
q)x+:2      / add 2 to x
q)x
14
```

### Multiple Assignment
```q
q)a:2
q)c:7*b:2+a   / multiple variables are assigned in same statement
q)a
2
q)b
4
q)c
28
q)\v          / to view all variables defined till now \v
`a`b`c`x
```

## Exit q
```q
q)\\          / double black slash
```
