# Attributes
kdb has 4 attributes that can be applied to datastructures (generally lists)

| Attr      | Symbol | Description                                                     |
| :-------: | :----: | :-------------------------------------------------------------: |
| sorted    | `s#    | Items are sorted - **binary search**                            |
| grouped   | `g#    | A dictionary maps **each** occurence to position in array       |
| unique    | `u#    | no duplicates                                                   |
| parted    | `p#    | stored same items together; dictionary maps **first** occurence |

* Each attribute provides different performance benefits
* Attributes may be lost during modification

* Applying attributes
```
q)list: 1 2 3 4 5
q)list
1 2 3 4 5
q)list: `s#1 2 3 4 5
q)list
`s#1 2 3 4 5
```

* we can check which attributes have been applied by checking meta data of the table
```
q)\l 2_where_clause_fakedb.q
USAGE: makedb[NUM QUOTES;NUM TRADES] eg makedb[100000;10000]

makedb1[NUM QUOTES;NUM TRADES;DATE;RANDOMISED COUNT FACTOR] eg makedb1[100000;10000;.z.d;.3]

makehdb[HDBDIR;NUM DAYS;APPROXIMATE NUM QUOTES PER DAY; APPROXIMATE NUM TRADES PER DAY] eg makehdb[`:hdb; 5; 100000; 10000]

makecsv[CSVDIR;NUM DAYS;NUM QUOTES;NUM TRADES]

q)makedb[10000;50000]
q)meta trades
c    | t f a
-----| -----
time | p    
sym  | s   g
src  | s   g
price| f    
size | i    
```

* we can also update a table by applying an attribute to a specific column

```
q)update time:`s#time from `trades
`trades
q)meta trades
c    | t f a
-----| -----
time | p   s
sym  | s   g
src  | s   g
price| f    
size | i    
```

* applying attribute to a column in table may improve query speed
* they are commonly applied to column with many repeating values

## 1. Sorted `s#

* tells q that list is sorted in ascending order
* if we use function `asc` to sorted a list, then q will automatically add `s# attribute
* Just having a list in ascending order wont apply `s# attribute, we need to explicitly tell q to apply it
```
q)list: 1 2 3 4 5
q)list
1 2 3 4 5
q)asc list
`s#1 2 3 4 5
q)list: `s#list
q)list
`s#1 2 3 4 5
```
* `s# attribute optimises the searching on list by performing binary search on the list 
* with `s# attribute search takes approx same time for element at start, mid and end
* without `s# attribute search takes more time as we search for element at end

```
q)a: til 100000
q)a
0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 ..
q)b:`s#til 100000
q)b
`s#0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 ..
q)\t do[1000;a?2]
0
q)\t do[10000;a?2]
4
q)\t do[10000;a?1000]
20
q)\t do[10000;a?10000-4]
87
q)\t do[10000;a?100000-4]
505
q)\t do[10000;b?100000-4]
11
q)\t do[10000;b?10000-4]
10
q)\t do[10000;b?1000]
7
q)\t do[10000;b?2]
8
```

## 2. Unique `u#

* tells q that list has unique elements and behind the scenes q creates a hash map, so it can query the elements in constant time
```
q)n:1000000
q)list: til n
q)listu: `u#til n
q)\t do[10000;list?`int$n%2]
2364
q)\t do[10000;listu?`int$n%2]
24

```
* `u# attribute is preserved when appending to list provided list still has unique elements after appending 

## 3. Grouped `g#

* when `g# attribute is applied to a list, q forms a lookup table of elements in list against list of their indexes

```
q)52?`1
`o`d`d`h`e`d`l`e`m`p`a`l`k`k`p`p`f`d`m`h`f`g`o`o`i`n`d`m`e`l`f`n`a`b`g`f`f`o`..
// lookup table looks something like this:
q)group 52?`1
a| 0 13 17 42 44
i| 1 4
l| ,2
f| 3 9 29 32 38
h| 5 18 21 25 26
b| 6 49
d| 7 14 23 27 41
c| 8 15 35 45
g| 10 28
m| 11 16 31 47
k| 12 19 22 30 33 34
j| 20 46
p| 24 51
e| 36 48
o| 37 39 40
n| 43 50
```

* it causes q to take more time while applying `g# attribute and it takes more space(to store lookup table) 
```
q)n
1000000
q)syms: -1000?`4
q)t1:([]a:n#syms;b:n?1000 )
q)tg:([]a:`g#n#syms;b:n?1000 )
q)\t do[10000;select from t1 where a=`lghi]
4930
q)\t do[10000;select from tg where a=`lghi]
19
q)
```

## 4. Parted `p#

* similar to grouped but here q only stores index of first occurence of an element 
* `p# is not preserved when appending to list even if elements added maintains list partitions
* This is mostly used for on-disk data
```
q)list
`a`a`a`a`b`b`b`c`c
q)`p#list
`p#`a`a`a`a`b`b`b`c`c
q)pl: `p#list
q)pl
`p#`a`a`a`a`b`b`b`c`c
// underlying lookup table looks something like this:
q)first each group pl
a| 0
b| 4
c| 7
q)pl,`c // appending `c at end causes pl to lose `p# 
`a`a`a`a`b`b`b`c`c`c
``` 

* when querying from HDB we should use partition column first(date/month/year) and then use column with attribute, if we use any other column after date then attribute is lost 
```
q)\l .
q)quotes
date       sym  time                          src bid ask bsize asize
---------------------------------------------------------------------
2014.04.21 aacj 2014.04.21D08:52:45.114000000 O           1500  5000
2014.04.21 aacj 2014.04.21D09:56:58.529000000 L           9500  8000
2014.04.21 aacj 2014.04.21D10:11:02.073000000 O           500   5500
2014.04.21 aacj 2014.04.21D11:05:48.110000000 O           5000  10000
2014.04.21 aacj 2014.04.21D11:57:34.256000000 N           6500  7000
2014.04.21 aacj 2014.04.21D13:13:43.109000000 O           6500  4000
2014.04.21 aacj 2014.04.21D14:53:19.787000000 N           2500  7500
2014.04.21 aacj 2014.04.21D15:42:19.164000000 L           4000  1000
2014.04.21 aacj 2014.04.21D16:14:48.307000000 L           9500  1500
2014.04.21 aacj 2014.04.21D16:24:17.107000000 O           8000  7500
2014.04.21 aakk 2014.04.21D08:33:13.751000000 N           6500  6500
2014.04.21 aakk 2014.04.21D09:00:10.954000000 N           5500  4000
2014.04.21 aakk 2014.04.21D09:10:39.732000000 N           9000  6000
2014.04.21 aakk 2014.04.21D10:31:16.183000000 L           7500  9500
2014.04.21 aakk 2014.04.21D11:05:31.205000000 L           3000  2500
2014.04.21 aakk 2014.04.21D11:06:45.876000000 O           8000  6500
2014.04.21 aakk 2014.04.21D11:44:05.643000000 O           2000  8500
2014.04.21 aakk 2014.04.21D11:47:34.308000000 N           8000  1000
2014.04.21 aakk 2014.04.21D12:25:59.515000000 L           10000 5000
2014.04.21 aakk 2014.04.21D12:26:02.080000000 N           1000  9500
..
q)meta quotes
c    | t f a
-----| -----
date | d
sym  | s   p
time | p
src  | s
bid  | f
ask  | f
bsize| i
asize| i
q)meta select from quotes where date=2014.04.21
c    | t f a
-----| -----
date | d
sym  | s   p
time | p
src  | s
bid  | f
ask  | f
bsize| i
asize| i
q)meta select from quotes where date=2014.04.21,bid>50
c    | t f a
-----| -----
date | d
sym  | s
time | p
src  | s
bid  | f
ask  | f
bsize| i
asize| i
```

## Speacial Case - step function using `s#

* we can use `s# to create step function 
* if `s# is not applied we cant search for values in between
* if `s# is applied we get values which is prevaling the argument value
```
q)d:1 1.6 1.9 2.1!`a`b`c`d
q)d
1  | a
1.6| b
1.9| c
2.1| d
q)d[1.0]
`a
q)d[1.1]
`
q)d:`s#1 1.6 1.9 2.1!`a`b`c`d
q)d[1.0]
`a
q)d[1.1]
`a
q)d[1.6]
`b
q)d[1.85]
`b
```

* same can be applied to tables as well
```
q)t:`s#([sym:`A`A`B;date:2014.12.05 2014.12.22 2015.01.10] px: 10 12 15)
q)t
sym date      | px
--------------| --
A   2014.12.05| 10
A   2014.12.22| 12
B   2015.01.10| 15
q)meta t
c   | t f a
----| -----
sym | s   p
date| d
px  | j
q)t(`A;2014.12.05)
px| 10
q)t(`A;2014.12.21)
px| 10
q)t(`A;2015.12.21)
px| 12
q)t(`B;2015.12.21)
px| 15
```