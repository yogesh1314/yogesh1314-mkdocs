# Joins

```
q)// setup
q)\l 2_where_clause_fakedb.q
USAGE: makedb[NUM QUOTES;NUM TRADES] eg makedb[100000;10000]

makedb1[NUM QUOTES;NUM TRADES;DATE;RANDOMISED COUNT FACTOR] eg makedb1[100000;10000;.z.d;.3]

makehdb[HDBDIR;NUM DAYS;APPROXIMATE NUM QUOTES PER DAY; APPROXIMATE NUM TRADES PER DAY] eg makehdb[`:hdb; 5; 100000; 10000]

makecsv[CSVDIR;NUM DAYS;NUM QUOTES;NUM TRADES]

q)makedb[1000;1000]
q)tables[]
`depth`quotes`trades
```
## Vertical Join `,`
* used to append 1 table at the end of another table
```
q)t1:5#trades
q)t2:-5#trades
q)t1,t2
time                          sym  src price size
-------------------------------------------------
2020.12.26D08:00:13.099000000 IBM  N   43.52 326
2020.12.26D08:00:39.491000000 CSCO L   35.45 2382
2020.12.26D08:01:46.279000000 AAPL O   25.34 211
2020.12.26D08:02:09.670000000 AAPL L   25.37 8077
2020.12.26D08:02:31.911000000 ORCL L   32.25 5730
2020.12.26D16:28:13.574000000 GOOG L   41.42 746
2020.12.26D16:28:51.196000000 NOK  N   31.77 147
2020.12.26D16:28:56.993000000 CSCO O   35.51 6233
2020.12.26D16:29:05.901000000 AAPL L   25.32 64
2020.12.26D16:29:38.453000000 NOK  N   31.81 307
```
* columns of the 2 tables must match
```
q)t1,5#quotes
'mismatch
  [0]  t1,5#quotes
         ^
```
* if we want to append subset of columns to table we must use `,:` (join in-place) which will fill missing values with null
```
q)t1,5#select time, sym, price from trades
'mismatch
  [0]  t1,5#select time, sym, price from trades
         ^
q)t1,:5#select time, sym, price from trades
q)t1
time                          sym  src price size
-------------------------------------------------
2020.12.26D08:00:13.099000000 IBM  N   43.52 326
2020.12.26D08:00:39.491000000 CSCO L   35.45 2382
2020.12.26D08:01:46.279000000 AAPL O   25.34 211
2020.12.26D08:02:09.670000000 AAPL L   25.37 8077
2020.12.26D08:02:31.911000000 ORCL L   32.25 5730
2020.12.26D08:00:13.099000000 IBM      43.52
2020.12.26D08:00:39.491000000 CSCO     35.45
2020.12.26D08:01:46.279000000 AAPL     25.34
2020.12.26D08:02:09.670000000 AAPL     25.37
2020.12.26D08:02:31.911000000 ORCL     32.25
```
* if we want to add single row of data, then also we must use join in-place(`,:`)
```
q)t1,(.z.p;`GOOG;`N;45.55;098)
`time`sym`src`price`size!(2020.12.26D08:00:13.099000000;`IBM;`N;43.52;326i)
`time`sym`src`price`size!(2020.12.26D08:00:39.491000000;`CSCO;`L;35.45;2382i)
`time`sym`src`price`size!(2020.12.26D08:01:46.279000000;`AAPL;`O;25.34;211i)
`time`sym`src`price`size!(2020.12.26D08:02:09.670000000;`AAPL;`L;25.37;8077i)
`time`sym`src`price`size!(2020.12.26D08:02:31.911000000;`ORCL;`L;32.25;5730i)
`time`sym`src`price`size!(2020.12.26D08:00:13.099000000;`IBM;`;43.52;0Ni)
`time`sym`src`price`size!(2020.12.26D08:00:39.491000000;`CSCO;`;35.45;0Ni)
`time`sym`src`price`size!(2020.12.26D08:01:46.279000000;`AAPL;`;25.34;0Ni)
`time`sym`src`price`size!(2020.12.26D08:02:09.670000000;`AAPL;`;25.37;0Ni)
`time`sym`src`price`size!(2020.12.26D08:02:31.911000000;`ORCL;`;32.25;0Ni)
2020.12.26D09:12:37.087587000
`GOOG
`N
45.55
98
q)t1,:(.z.p;`GOOG;`N;45.55;098)
q)t1
time                          sym  src price size
-------------------------------------------------
2020.12.26D08:00:13.099000000 IBM  N   43.52 326
2020.12.26D08:00:39.491000000 CSCO L   35.45 2382
2020.12.26D08:01:46.279000000 AAPL O   25.34 211
2020.12.26D08:02:09.670000000 AAPL L   25.37 8077
2020.12.26D08:02:31.911000000 ORCL L   32.25 5730
2020.12.26D08:00:13.099000000 IBM      43.52
2020.12.26D08:00:39.491000000 CSCO     35.45
2020.12.26D08:01:46.279000000 AAPL     25.34
2020.12.26D08:02:09.670000000 AAPL     25.37
2020.12.26D08:02:31.911000000 ORCL     32.25
2020.12.26D09:12:41.729632000 GOOG N   45.55 98
```
## Horizontal Join `,'`
* join each
* it adds extra columns to the right of existing table
```
q)flip (enlist `side)!enlist 1000?`buy`sell  // created new column side
side
----
sell
buy
sell
sell
sell
..
q)trades,'flip (enlist `side)!enlist 1000?`buy`sell
time                          sym  src price size side
------------------------------------------------------
2020.12.26D08:00:13.099000000 IBM  N   43.52 326  sell
2020.12.26D08:00:39.491000000 CSCO L   35.45 2382 buy
2020.12.26D08:01:46.279000000 AAPL O   25.34 211  buy
2020.12.26D08:02:09.670000000 AAPL L   25.37 8077 buy
2020.12.26D08:02:31.911000000 ORCL L   32.25 5730 sell
..
```
* two table must have same number of rows, otherwise we would get `'length` error
```
q)count flip (enlist `side)!enlist 50?`buy`sell
50
q)trades,'flip (enlist `side)!enlist 50?`buy`sell
'length
  [0]  trades,'flip (enlist `side)!enlist 50?`buy`sell

```

## Left Join `lj`
* joins 2 tables on the keyed columns of the right hand table
* right hand table must be a keyed table
* this can be used to add static data to a table
```
q)voltrades:select vol:sum size by sym from trades
q)voltrades
sym | vol
----| ------
AAPL| 272375
CSCO| 295679
DELL| 274055
GOOG| 207724
IBM | 400192
..
q)trades lj voltrades
time                          sym  src price size vol
--------------------------------------------------------
2020.12.26D08:00:13.099000000 IBM  N   43.52 326  400192
2020.12.26D08:00:39.491000000 CSCO L   35.45 2382 295679
2020.12.26D08:01:46.279000000 AAPL O   25.34 211  272375
2020.12.26D08:02:09.670000000 AAPL L   25.37 8077 272375
2020.12.26D08:02:31.911000000 ORCL L   32.25 5730 337317
..
```
* if keyed table doesnt have values for each keyed column in left hand side table, then null data will be added for missing keys
```
q)1_voltrades
sym | vol
----| ------
CSCO| 295679
DELL| 274055
GOOG| 207724
IBM | 400192
MSFT| 330837
..
q)trades lj 1_voltrades
time                          sym  src price size vol
--------------------------------------------------------
2020.12.26D08:00:13.099000000 IBM  N   43.52 326  400192
2020.12.26D08:00:39.491000000 CSCO L   35.45 2382 295679
2020.12.26D08:01:46.279000000 AAPL O   25.34 211
2020.12.26D08:02:09.670000000 AAPL L   25.37 8077
2020.12.26D08:02:31.911000000 ORCL L   32.25 5730 337317
..
```
* joins can also be made for more than 1 columns
```
q)voltrades2: select sum size by sym,src from trades
q)voltrades2
sym  src| size
--------| ------
AAPL L  | 110676
AAPL N  | 56876
AAPL O  | 104823
CSCO L  | 122109
CSCO N  | 75468
..
q)trades lj voltrades2
time                          sym  src price size
---------------------------------------------------
2020.12.26D08:00:13.099000000 IBM  N   43.52 134448
2020.12.26D08:00:39.491000000 CSCO L   35.45 122109
2020.12.26D08:01:46.279000000 AAPL O   25.34 104823
2020.12.26D08:02:09.670000000 AAPL L   25.37 110676
2020.12.26D08:02:31.911000000 ORCL L   32.25 186353
..
```
## Plus Join `pj`
* similar to a `left join`
* it will add the values in keyed table to the corresponding values in left-hand table
* if the value in the left hand table has no matching value in the keyed table, it will add `0`
```
q)trades
time                          sym  src price size
-------------------------------------------------
2020.12.26D08:00:13.099000000 IBM  N   43.52 326
2020.12.26D08:00:39.491000000 CSCO L   35.45 2382
2020.12.26D08:01:46.279000000 AAPL O   25.34 211
2020.12.26D08:02:09.670000000 AAPL L   25.37 8077
2020.12.26D08:02:31.911000000 ORCL L   32.25 5730
..
q)correction:([sym:`AAPL`GOOG] price:.5 .5; changed:1 1)
q)correction
sym | price changed
----| -------------
AAPL| 0.5   1
GOOG| 0.5   1
q)
q)trades pj correction
time                          sym  src price size changed
---------------------------------------------------------
2020.12.26D08:00:13.099000000 IBM  N   43.52 326  0
2020.12.26D08:00:39.491000000 CSCO L   35.45 2382 0
2020.12.26D08:01:46.279000000 AAPL O   25.84 211  1
2020.12.26D08:02:09.670000000 AAPL L   25.87 8077 1
2020.12.26D08:02:31.911000000 ORCL L   32.25 5730 0
..
```
* values in the keyed table must be compatible with addition
```
q)correction:([sym:`AAPL`GOOG] price:.5 .5; changed:`y`y)
q)correction
sym | price changed
----| -------------
AAPL| 0.5   y
GOOG| 0.5   y
q)trades pj correction
'type
  [0]  trades pj correction
              ^
q))\
```


## Inner Join `ij`
* inner join is similar to left join, however it will only return a value from left hand table if there is corresponding value in the keyed table
* only values present in both the table for each key are returned
* in left join, it would return all the rows from left table with null data joined even if there is no data in keyed table
```
q)voltrades
sym | vol
----| ------
AAPL| 272375
CSCO| 295679
DELL| 274055
GOOG| 207724
IBM | 400192
..
q)voltrades2:2_voltrades
q)voltrades2
sym | vol
----| ------
DELL| 274055
GOOG| 207724
IBM | 400192
MSFT| 330837
NOK | 215976
..
q)count trades lj voltrades2
1000
q)count trades ij voltrades2
788
q)trades ij voltrades2
time                          sym  src price size vol
--------------------------------------------------------
2020.12.26D08:00:13.099000000 IBM  N   43.52 326  400192
2020.12.26D08:02:31.911000000 ORCL L   32.25 5730 337317
2020.12.26D08:02:40.066000000 ORCL N   32.22 4295 337317
2020.12.26D08:03:00.934000000 YHOO N   35.49 5339 320482
2020.12.26D08:03:29.198000000 ORCL L   32.25 2629 337317
..
q)trades lj voltrades
time                          sym  src price size vol
--------------------------------------------------------
2020.12.26D08:00:13.099000000 IBM  N   43.52 326  400192
2020.12.26D08:00:39.491000000 CSCO L   35.45 2382 295679
2020.12.26D08:01:46.279000000 AAPL O   25.34 211  272375
2020.12.26D08:02:09.670000000 AAPL L   25.37 8077 272375
2020.12.26D08:02:31.911000000 ORCL L   32.25 5730 337317
..
```
## Union Join `uj`
```
q) // setup
q)\l 2_where_clause_fakedb.q
q)makedb[10000;1000]
q)tables[]
`depth`quotes`trades
q)count trades
1000
q)count quotes
10000
```
* can be keyed and unkeyed as well
* **unkeyed** - it will return a super set of all rows and all columns of both tables - missing values are filled with nulls
```
q)t1:5#trades
q)t2:5#quotes
q)t1
time                          sym  src price size
-------------------------------------------------
2020.12.26D08:00:17.350000000 CSCO N   35.46 2367
2020.12.26D08:00:30.245000000 DELL N   29.05 27
2020.12.26D08:01:40.842000000 AAPL L   25.37 2797
2020.12.26D08:02:11.579000000 CSCO L   35.46 1714
2020.12.26D08:02:12.246000000 YHOO N   35.5  166
q)t2
time                          sym  src bid   ask   bsize asize
--------------------------------------------------------------
2020.12.26D08:00:02.809000000 MSFT L   36.08 36.09 8000  7500
2020.12.26D08:00:09.282000000 MSFT O   36.06 36.09 5500  8000
2020.12.26D08:00:10.075000000 IBM  N   43.52 43.53 2500  4000
2020.12.26D08:00:10.646000000 YHOO L   35.49 35.5  7500  4500
2020.12.26D08:00:11.107000000 DELL N   29.05 29.09 2000  6000
q)t1 uj t2
time                          sym  src price size bid   ask   bsize asize
-------------------------------------------------------------------------
2020.12.26D08:00:17.350000000 CSCO N   35.46 2367
2020.12.26D08:00:30.245000000 DELL N   29.05 27
2020.12.26D08:01:40.842000000 AAPL L   25.37 2797
2020.12.26D08:02:11.579000000 CSCO L   35.46 1714
2020.12.26D08:02:12.246000000 YHOO N   35.5  166
2020.12.26D08:00:02.809000000 MSFT L              36.08 36.09 8000  7500
2020.12.26D08:00:09.282000000 MSFT O              36.06 36.09 5500  8000
2020.12.26D08:00:10.075000000 IBM  N              43.52 43.53 2500  4000
2020.12.26D08:00:10.646000000 YHOO L              35.49 35.5  7500  4500
2020.12.26D08:00:11.107000000 DELL N              29.05 29.09 2000  6000
```
* **keyed** - rows from the left keyed table are updated with matches from right keyed table - if no match is found, then null data is added
* can be useful which are keyed by time
* example - we have table which has average mid price bucketed by 5 minute buckets
```
q)tq: select avg .5*bid+ask by 5 xbar time.minute from quotes
q)tt: select avg price by 5 xbar time.minute from trades where sym=`IBM
q)tq
minute| x
------| --------
08:00 | 33.93856
08:05 | 35.3704
08:10 | 35.56701
08:15 | 34.212
08:20 | 33.9723
08:25 | 35.0075
08:30 | 34.20413
08:35 | 34.81067
08:40 | 35.19892
08:45 | 33.94478
08:50 | 35.65367
08:55 | 33.19094
09:00 | 35.1866
09:05 | 34.21341
09:10 | 34.51582
09:15 | 34.58091
09:20 | 34.29382
09:25 | 33.81271
09:30 | 33.77884
09:35 | 34.51159
..
q)tt
minute| price
------| --------
08:00 | 43.56333
08:10 | 43.55
08:15 | 43.54
08:20 | 43.56
08:30 | 43.7
08:35 | 43.62
08:40 | 43.63
08:50 | 43.56
09:05 | 43.515
09:15 | 43.43
09:25 | 43.42
09:30 | 43.49333
09:35 | 43.5
09:40 | 43.495
09:45 | 43.49
09:50 | 43.52
09:55 | 43.585
10:10 | 43.57
10:15 | 43.57
10:25 | 43.515
..
q)tq uj tt
minute| x        price
------| -----------------
08:00 | 33.93856 43.56333
08:05 | 35.3704
08:10 | 35.56701 43.55
08:15 | 34.212   43.54
08:20 | 33.9723  43.56
08:25 | 35.0075
08:30 | 34.20413 43.7
08:35 | 34.81067 43.62
08:40 | 35.19892 43.63
08:45 | 33.94478
08:50 | 35.65367 43.56
08:55 | 33.19094
09:00 | 35.1866
09:05 | 34.21341 43.515
09:10 | 34.51582
09:15 | 34.58091 43.43
09:20 | 34.29382
09:25 | 33.81271 43.42
09:30 | 33.77884 43.49333
09:35 | 34.51159 43.5
..
```
* we can fill the nulls with prev avg price if there are not trades during the bucket window
```
q)fills tq uj tt
minute| x        price
------| -----------------
08:00 | 33.93856 43.56333
08:05 | 35.3704  43.56333
08:10 | 35.56701 43.55
08:15 | 34.212   43.54
08:20 | 33.9723  43.56
08:25 | 35.0075  43.56
08:30 | 34.20413 43.7
08:35 | 34.81067 43.62
08:40 | 35.19892 43.63
08:45 | 33.94478 43.63
08:50 | 35.65367 43.56
08:55 | 33.19094 43.56
09:00 | 35.1866  43.56
09:05 | 34.21341 43.515
09:10 | 34.51582 43.515
09:15 | 34.58091 43.43
09:20 | 34.29382 43.43
09:25 | 33.81271 43.42
09:30 | 33.77884 43.49333
09:35 | 34.51159 43.5
..
```
