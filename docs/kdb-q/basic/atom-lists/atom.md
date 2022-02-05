# Atoms

q has 3 main entities: **atom**, **list** and **dictionary**

Atom is smallest level of entity all others are combinations of atoms

## Basic data types

* **boolean**
* **short**
* **int**
* **long**
* **float**
* **character**
* **symbol**

We can specify types while defining variables in q

```q
q) a:1         
q) a:1i                               / here we are specifying type of variable a to be int
q) c:1                                / if we don't specify, q automatically assigns, here c is of type long
q) d:2.0
q) b:2f                               / b and d are float type
q) t:1b                               / boolean type is specified by b, its 1b or 0b
q) d:"m"                              / d is now character
q) str:"mogli"                        / str is not atom, its a list of 5 characters
q) sym: `mogli                        / sym is symbol, its a data type in q where we can store list of characters as atom
```

**Type** - we can check type of a variable/constant using function `type`, `-` means atom, `+`/no sign means list

[Reference](https://code.kx.com/q/ref/#datatypes)

```q
q) type d                             / d is atom of type character, -ve sign in type
-10h
q) type str                           / str is a list of 5 characters, +ve sign in type
10h
q) type sym                           / symbol is an atom
-11h
```
## Temporal data types
* **date** - `yyyy.mm.dd`
* **month** - `yyyy.mm`m
* **time** - `hh:mm:ss.12345689` - precision till nanoseconds
* **timestamp** - `yyyy.mm.ddDhh:mm:ss.123456789` - precision till nanoseconds
	* we need not specify full precision for timestamp
	* date `+` time `=` timestamp
* date and time are seprated by **D**
* **datetime** - `yyyy.mm.ddThh:mm:ss.123` - precision till milliseconds (used previously, avoid using now)
	* date and time are seprated by **T**
* Arithmetic operations can be performed with temporal data types
* Dates and time are based at `0` and saved as numerals in q
	* date `0` is `2000.01.01`, before `-`, later `+`
	* time `0` is midnight - `00:00:00.000000000`
	* timestamp `0` is midnight of `2000.01.01` - `2000.01.01D00:00:00.000000000`, before `-`, later `+`
* Current date: `.z.d`, current time: `.z.t`, current timestamp: `.z.p`


```q
q) time: 09:00:00.000000000
q) type time
-16h
q) date: 2020.05.30
q) type date
-14h
q) month: 2020.05m                    / month is defined by putting 'm' in the end,'m' helps to differentiate between float and month
q) timestamp: 2019.05.30D17:15:00.000000000
q) t:2019.05.30D                      / time stamp precision can be relaxed while defining
q) t
2020.05.30D00:00:00.000000000
q) date+time
2020.05.30D09:00:00.000000000
q) date-2000.01.01
7455i
```
## Match `~` vs. equal `=`
**match** compare whole object including type, **equal** compare underlying values

```q
q) 3i=3i
1b
q) 3i=3f
1b
q) 3i~3i
1b
q) 3i~3f
0b
```

## Type promotion
* Arithmetic operations can result into type promotion
* Default type for integer is long(64 bit integer)
* Boolean are promoted to another type when needed
	* this is useful in `if` or other conditional statements : `if[test_condition; do...];`
	* non-zero(+/-) is true`1b`, `0` is false`0b`

```q
q) 2i+3h
5i
q) type 1i+4.3f
-9h
q) a:1                                / default type of variable 'a' would be long
q) type a
-7h
q) a=1
1b
q) if[a=1; show "a equals 1"]         / here it is evaluating a=1 and proceed further if true
"a equals 1"
q) if[a; show "statement not false"]  / here it is checking a is non zero
"statement not false"
q) if[0; show "false"]
q) if[count (1 2 3); show "list not empty"]
"list not empty"
q) if[count (); show "list is empty"]
q)
```

## Casting
* new_type`$`old_type
* from string to other types, we need to use corresponding character for new_type
* from other types to string, we use function `string`
* numerals to symbol casting dont work directly, first we need to cast to string
* casting temporal values to numerals we can see their underlying values

```q
q) `int$3.1f
3i
q) `int$3.6f
4i                                    / rounded off
q) `$"name"
`name                                 / from string to symbol use empty symbol `
q) string `test
"test"
q) string 23i
"23"
q) "I"$"23"
23i
q) "F"$"23"
23f
q) `$2020.05.30
'type
q) `$string 2020.05.30
`2020.05.30
q) date-2000.01.01
7455i
q) `int$date
7455i
q) 0=2000.01.01
1b
```