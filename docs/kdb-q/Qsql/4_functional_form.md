# Functional Form

* functional form is used for select, update, delete and exec queries
```
q)//setup
q)\l 2_where_clause_fakedb.q
USAGE: makedb[NUM QUOTES;NUM TRADES] eg makedb[100000;10000]

makedb1[NUM QUOTES;NUM TRADES;DATE;RANDOMISED COUNT FACTOR] eg makedb1[100000;10000;.z.d;.3]

makehdb[HDBDIR;NUM DAYS;APPROXIMATE NUM QUOTES PER DAY; APPROXIMATE NUM TRADES PER DAY] eg makehdb[`:hdb; 5; 100000; 10000]

makecsv[CSVDIR;NUM DAYS;NUM QUOTES;NUM TRADES]

q)makedb[100000;1000]
q)tables[]
`depth`quotes`trades
```

## `select`
* general format for select is `?[table name; where clause; groupings; aggregations]`
* we can also get each component of functional query using function `parse`
```
q)parse"select from trades"
?
`trades
()
0b
()
q)?[`trades;();0b;()]
time                          sym  src price size
-------------------------------------------------
2020.12.25D08:00:06.782000000 MSFT N   36.07 3996
2020.12.25D08:00:43.092000000 IBM  N   43.59 3001
2020.12.25D08:01:14.926000000 NOK  L   31.7  702
2020.12.25D08:01:26.622000000 AAPL N   25.3  1228
..
```
* if our `select` query is bit complex then understanding output of `parse` function is bit difficult, `,` means `enlist`
* keys of aggregation or groupings dictionary are names we want our columns to be and values are values we want columns to hold
```
q)select ssize:sum size by sym from trades where price<35
sym | ssize
----| ------
AAPL| 287593
CSCO| 9816
DELL| 296884
NOK | 325169
ORCL| 255399
YHOO| 91038
q)parse"select ssize:sum size by sym from trades where price<35"
?
`trades
,,(<;`price;35)
(,`sym)!,`sym
(,`ssize)!,(sum;`size)
q)?[trades;enlist(<;`price;35f);(enlist `sym)!enlist `sym;(enlist `ssize)!(enlist(sum;`size))]
sym | ssize
----| ------
AAPL| 287593
CSCO| 9816
DELL| 296884
NOK | 325169
ORCL| 255399
YHOO| 91038
```

* functional forms are generally used when we functions to be evaluate queries on dynamically passed column names
* functional forms are hard to read, difficult to write, needs atoms to be enlisted and thus should only be used when necessary

## `exec`
* it is simplified version of `select`
* for no group by we use empty list`()` than `0b`
* general form : `?[tablename; where clause; groupings; aggregations]`
* `exec sym from table` =  `?[table;();();`sym]`
```
q)exec sym from trades
`g#`MSFT`IBM`NOK`AAPL`GOOG`CSCO`NOK`AAPL`NOK`YHOO`AAPL`AAPL`AAPL`IBM`CSCO`YHO..
q)parse"exec sym from trades"
?
`trades
()
()
,`sym
q)?[trades;();();`sym]
`g#`MSFT`IBM`NOK`AAPL`GOOG`CSCO`NOK`AAPL`NOK`YHOO`AAPL`AAPL`AAPL`IBM`CSCO`YHO..
```
## `update`
* it is very much similar to select, we use `!` rather than `?`
* general form : `![tablename;where clause; groupings; aggregations]`
```
q)update pp:price*2 from trades
time                          sym  src price size pp
-------------------------------------------------------
2020.12.25D08:00:06.782000000 MSFT N   36.07 3996 72.14
2020.12.25D08:00:43.092000000 IBM  N   43.59 3001 87.18
2020.12.25D08:01:14.926000000 NOK  L   31.7  702  63.4
2020.12.25D08:01:26.622000000 AAPL N   25.3  1228 50.6
2020.12.25D08:01:27.019000000 GOOG L   41.38 4905 82.76
..
q)parse"update pp:price*2 from trades"
!
`trades
()
0b
(,`pp)!,(*;`price;2)
q)![trades;();0b;(enlist `pp)!enlist(*;`price;2)]
time                          sym  src price size pp
-------------------------------------------------------
2020.12.25D08:00:06.782000000 MSFT N   36.07 3996 72.14
2020.12.25D08:00:43.092000000 IBM  N   43.59 3001 87.18
2020.12.25D08:01:14.926000000 NOK  L   31.7  702  63.4
2020.12.25D08:01:26.622000000 AAPL N   25.3  1228 50.6
2020.12.25D08:01:27.019000000 GOOG L   41.38 4905 82.76
..
q)![trades;();0b;((),`pp)!enlist (*;`price;2)]
time                          sym  src price size pp
-------------------------------------------------------
2020.12.25D08:00:06.782000000 MSFT N   36.07 3996 72.14
2020.12.25D08:00:43.092000000 IBM  N   43.59 3001 87.18
2020.12.25D08:01:14.926000000 NOK  L   31.7  702  63.4
2020.12.25D08:01:26.622000000 AAPL N   25.3  1228 50.6
2020.12.25D08:01:27.019000000 GOOG L   41.38 4905 82.76
..
```
## `delete`
* it is very much similar to `exec`, here `!` is used instead of `?`
* general form : `![tablename;where clause; groupings; aggregations]`
```
q)delete price from trades
time                          sym  src size
-------------------------------------------
2020.12.25D08:00:06.782000000 MSFT N   3996
2020.12.25D08:00:43.092000000 IBM  N   3001
2020.12.25D08:01:14.926000000 NOK  L   702
2020.12.25D08:01:26.622000000 AAPL N   1228
2020.12.25D08:01:27.019000000 GOOG L   4905
..
q)parse"delete price from trades"
!
`trades
()
0b
,,`price
q)![trades;();0b;(),`price]
time                          sym  src size
-------------------------------------------
2020.12.25D08:00:06.782000000 MSFT N   3996
2020.12.25D08:00:43.092000000 IBM  N   3001
2020.12.25D08:01:14.926000000 NOK  L   702
2020.12.25D08:01:26.622000000 AAPL N   1228
2020.12.25D08:01:27.019000000 GOOG L   4905
..
```
* in update and delete if we want changes to happen in place(directly on table) - we pass tablename as symbol
```
q)![`trades;();0b;(),`price] // in-place change as tablename is passed as symbol
`trades
q)trades
time                          sym  src size
-------------------------------------------
2020.12.25D08:00:06.782000000 MSFT N   3996
2020.12.25D08:00:43.092000000 IBM  N   3001
2020.12.25D08:01:14.926000000 NOK  L   702
2020.12.25D08:01:26.622000000 AAPL N   1228
2020.12.25D08:01:27.019000000 GOOG L   4905
..
```
* in delete we can only specify `where` conditions or `aggregations` but not both at the same time
```
q)delete price from trades where sym=`AAPL
'nyi
q)parse"delete price from trades where sym=`AAPL"
!
`trades
,,(=;`sym;,`AAPL)
0b
,,`price
q)![`trades;enlist(=;`sym;enlist `AAPL);0b;(),`price]
'nyi
```
