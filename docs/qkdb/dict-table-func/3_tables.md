# Tables

* list of dictionaries or collection of named columns

## Simple table

* can be created by flipping a dictionary
* can be created using format `([] column_name: values; col...)`
* data type is 98

```q
q)t:([]a:1 2 3; b:4 5 6)
q)t
a b
---
1 4
2 5
3 6
```

## Keyed table

* dictionary with mapping of table to table
* can be created using table format `([key_columns] column_name: values; col...)`
* data type is 99

```q
q)kt:([a:`a`b`c] b: 1 2 3; c:4 5 6)
q)kt
a| b c
-| ---
a| 1 4
b| 2 5
c| 3 6
q)type kt
99h
```
## Empty table
* used to specify schema of table before inserting anything in it
* we can force column type while defining, if defined, only values with matching type can be inserted

```q
q)et:([]a:(); b:())
q)et
a b
---
q)et2:([]a:`int$(); b:`float$())
q)et2
a b
---
```

## `meta`

* applied to a table will return table info
* c is for column names
* t is for type
	* if values are lists, type will be represented by capital letters
* f is for foreign key
* a is for attributes

```q
q)meta kt
c| t f a
-| -----
a| s
b| j
c| j
q)t2:([]a: ((1 2);(3 4)); b: 4 5f)
q)t2
a   b
-----
1 2 4
3 4 5
q)meta t2
c| t f a
-| -----
a| J
b| f
```
## Foreign keys
* table can be linked to another keyed table using foreign keys
* while defining foreign keys we use `keyed_table_name$values
* meta of new table will show `f` column to have linked to `kt` keyed_table_name

```q
q)kt
a| b c
-| ---
a| 1 4
b| 2 5
c| 3 6
q)t3:([]id: `kt$`a`b`c; prc: 10 20 30f)
q)t3
id prc
------
a  10
b  20
c  30
q)meta t3
c  | t f  a
---| ------
id | s kt
prc| f
```

## Extracting rows and columns
* we can extract rows using row index number, return type is dictionary
* we can extract column using column name
```q
q)t[0]
a| 1
b| 4
q)t[`a]
1 2 3
q)t[`a`b]
1 2 3
4 5 6
```
* take operator `#` can be used to pull first n or last n rows from table
* similarly drop operator `_` can be used

```q
q)2#t
a b
---
1 4
2 5
q)-2#t
a b
---
2 5
3 6
q)1 _ t
a b
---
2 5
3 6
q)-1 _ t
a b
---
1 4
2 5
```

## Table lookup

* we can lookup using find operator
* we can also find using list(row values) in a table

```q
q)t2:([]a: 1 2 3; b: 4 5 6; c: 7 8 9)
q)t2
a b c
-----
1 4 7
2 5 8
3 6 9
q)t2?`a`b`c!3 6 9
2
q)t2?3 6 9
2
q)t2?(2 5 8; 3 6 9)
1 2
q)t2?([]a:1 2; b:4 5; c:7 8)
0 1
```

## Table arithmetic

* we can perform arithmetic operations on keyed tables
* operation will be performed for matching keys and rest will remain unchanged

```q
q)kt1:([ks:`a`b`c] v1: 1 2 3; v2: 10 11 12)
q)kt2:([ks:`b`c`e] v1: 20 30 40; v2: 100 101 102)
q)kt1+kt2
ks| v1 v2
--| ------
a | 1  10
b | 22 111
c | 33 113
e | 40 102
```
