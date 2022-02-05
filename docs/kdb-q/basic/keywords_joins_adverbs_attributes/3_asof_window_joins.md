# asof and window join

```
q) // setup
q)\l 2_where_clause_fakedb.q
q)makedb[50000;10000]
q)tables[]
`depth`quotes`trades
q)count quotes
50000
q)count trades
10000
```
* there are many more quotes than trades
* we want to find quotes just before the trade took place for each sym
```
```

## As of Join `aj` `aj0`
* format : `aj[c1..cn;table1;table2]`
* `cn` : column on which as-of-join is performed, most recent time from t2 which is asof(<=) time in table1
* `c1..cn-1` : columns are matched exactly in table1 and table2
* `aj` will have all columns from `table1` and values of columns `c1..cn` are from `table1` and value from `table2` will have matching values
```
q)select from aj[`sym`time;trades;quotes] where sym=`AAPL
time                          sym  src price size bid   ask   bsize asize
-------------------------------------------------------------------------
2020.12.26D08:00:03.057000000 AAPL L   25.33 4527 25.33 25.37 9000  3000
2020.12.26D08:00:12.190000000 AAPL O   25.38 1889 25.35 25.38 4000  7000
2020.12.26D08:01:22.084000000 AAPL L   25.37 5413 25.35 25.37 6000  5500
2020.12.26D08:02:12.686000000 AAPL O   25.33 1841 25.32 25.33 4000  5000
2020.12.26D08:02:59.207000000 AAPL N   25.32 957  25.32 25.36 1500  6500
2020.12.26D08:03:04.870000000 AAPL N   25.34 991  25.32 25.34 3000  1000
2020.12.26D08:03:05.194000000 AAPL N   25.34 454  25.32 25.34 3000  1000
2020.12.26D08:03:50.205000000 AAPL N   25.34 176  25.31 25.34 1500  2000
2020.12.26D08:03:52.988000000 AAPL N   25.34 491  25.31 25.34 1500  2000
2020.12.26D08:04:00.136000000 AAPL L   25.35 4392 25.32 25.35 1000  5500
2020.12.26D08:04:14.266000000 AAPL N   25.32 530  25.32 25.33 5500  9000
2020.12.26D08:05:16.333000000 AAPL L   25.35 1118 25.33 25.35 7000  6000
2020.12.26D08:06:36.068000000 AAPL L   25.33 1321 25.33 25.33 5500  6000
2020.12.26D08:06:46.472000000 AAPL L   25.33 1973 25.33 25.35 2500  8500
2020.12.26D08:07:40.059000000 AAPL O   25.36 4447 25.36 25.4  10000 500
2020.12.26D08:08:01.875000000 AAPL N   25.37 4330 25.37 25.42 7500  10000
2020.12.26D08:08:07.323000000 AAPL N   25.42 3511 25.37 25.42 7500  10000
2020.12.26D08:08:11.894000000 AAPL N   25.37 4588 25.37 25.42 7500  10000
2020.12.26D08:08:16.016000000 AAPL N   25.37 3810 25.37 25.42 7500  10000
2020.12.26D08:08:25.522000000 AAPL N   25.37 2264 25.37 25.42 7500  10000
..
q)select from quotes where sym=`AAPL
time                          sym  src bid   ask   bsize asize
--------------------------------------------------------------
2020.12.26D08:00:02.705000000 AAPL N   25.32 25.37 6500  1000
2020.12.26D08:00:02.882000000 AAPL L   25.33 25.37 9000  3000
2020.12.26D08:00:04.111000000 AAPL L   25.33 25.37 9500  7000
2020.12.26D08:00:05.040000000 AAPL L   25.35 25.37 2500  9500
2020.12.26D08:00:09.235000000 AAPL N   25.35 25.37 4000  6500
2020.12.26D08:00:10.576000000 AAPL O   25.35 25.38 4000  7000
2020.12.26D08:00:16.432000000 AAPL O   25.35 25.39 3000  10000
2020.12.26D08:00:25.528000000 AAPL N   25.35 25.37 3000  4000
2020.12.26D08:00:26.237000000 AAPL N   25.34 25.37 6000  9000
2020.12.26D08:00:32.058000000 AAPL L   25.35 25.38 3000  4000
2020.12.26D08:00:47.294000000 AAPL L   25.34 25.36 9500  2500
2020.12.26D08:01:06.551000000 AAPL N   25.35 25.38 4500  2500
2020.12.26D08:01:08.708000000 AAPL N   25.32 25.37 9000  7500
2020.12.26D08:01:16.411000000 AAPL L   25.35 25.37 6000  5500
2020.12.26D08:01:25.050000000 AAPL L   25.34 25.37 500   5500
2020.12.26D08:01:33.147000000 AAPL N   25.35 25.36 6500  5000
2020.12.26D08:01:46.838000000 AAPL L   25.34 25.37 2500  4500
2020.12.26D08:01:47.149000000 AAPL L   25.32 25.35 3000  5500
2020.12.26D08:01:48.825000000 AAPL N   25.33 25.36 500   7000
2020.12.26D08:01:59.163000000 AAPL L   25.35 25.36 5000  500
..
```
* query performance will matter a lot if join columns`c1..cn` has grouped(`g#`) attribute set for in-memory tables and parted(`p#`) attribute set for on-disk tables
```
q)\t aj[`sym`time;trades;quotes]
12
q)meta quotes
c    | t f a
-----| -----
time | p
sym  | s   g
src  | s   g
bid  | f
ask  | f
bsize| i
asize| i
q)meta trades
c    | t f a
-----| -----
time | p
sym  | s   g
src  | s   g
price| f
size | i
q)update `#sym from `quotes
`quotes
q)\t aj[`sym`time;trades;quotes]
5805
```
* `aj0` will used if in result table we want to show values for column `cn` from `table2`
### `aj` in **hdb**
* main difference with on-disk tables is we would have to load them in-memory first
* while loading parted attribute `p#` is lost - workaroun :  load data for each date and raze them into 1 table - which will preserve parted attributes

## Window Join `wj` `wj0`
* in `aj` we would get last value/most-recent value
* in `wj` we can specify our window and also aggregation function which would be applied on joined data
* format: `wj[w; c; t; (q; (f0;c0); (f1;c1))]`
* `t` and `q` are tables
* `w` is pair of lists of times/timestamps, begin and end
* `c` are the common columns
* `f0`, `f1` are aggregate function applied to values in q columns `c0` and `c1` over the intervals
* returns for each record in `t`, a record with additional columns `c0` and `c1`, which are the results of the aggregation functions applied to values over the matching intervals in `w`.
* `aj` is a special case of `wj` or `wj` is generalized form of `aj`
* exact match columns `c` needs to have attribute `#p` or `#g` applied to them, other wise query would be much slower and also might give wrong results as well
```
q)w: -1000 1000+\:trades.time
q)w
08:00:02.057 08:00:03.230 08:00:10.150 08:00:11.190 08:00:11.531 08:00:14.312..
08:00:04.057 08:00:05.230 08:00:12.150 08:00:13.190 08:00:13.531 08:00:16.312..
q)\t wj[w; `sym`time;trades;(quotes; (max;`bid);(min;`ask))]
61
q)wj[w; `sym`time;trades;(quotes; (max;`bid);(min;`ask))]
time         sym  src price size bid   ask
--------------------------------------------
08:00:03.057 AAPL L   25.33 4527 25.33 25.37
08:00:04.230 YHOO N   35.49 1457 35.49 35.52
08:00:11.150 DELL L   29.08 6788 43.53 25.38
08:00:12.190 AAPL O   25.38 1889 25.35 25.38
08:00:12.531 MSFT L   36.08 4974 36.08 36.11
08:00:15.312 ORCL O   32.2  668  43.53 25.37
08:00:24.269 ORCL L   32.23 1116 32.19 32.23
08:00:29.856 MSFT L   36.13 766  36.1  36.13
08:00:33.033 ORCL L   32.23 1749 43.54 25.37
08:00:33.126 ORCL L   32.19 1394 43.54 25.37
08:00:40.441 MSFT L   36.12 1151 36.12 36.13
08:00:46.699 IBM  N   43.57 1958 43.55 43.57
08:00:49.597 CSCO L   35.46 2164 35.46 35.49
08:00:51.534 CSCO L   35.46 2762 35.46 35.49
08:00:56.880 YHOO N   35.52 227  35.49 35.52
08:00:57.649 MSFT L   36.13 2028 36.08 36.13
08:00:58.363 CSCO O   35.49 2273 35.45 35.49
08:01:00.037 DELL L   29.06 3260 29.04 29.06
08:01:04.382 NOK  O   31.79 3765 31.79 31.83
08:01:06.810 IBM  N   43.55 4224 43.55 25.36
..
```
