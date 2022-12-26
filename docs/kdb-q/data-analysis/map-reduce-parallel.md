* When running queries against on-disk, partitioned databases, kdb+ will automatically apply map-reduce algorithms to some operations
* These operations are executed in each partition seprately, then the results are aggregated to give the final answer
* Operations: `count` , `first` , `last`, `sum`, `prd`, `min`, `max`, `distinct`, `avg`, `wsum`, `wavg`, `var`, `dev`, `cov`, `cor`, `med`
* Some operations are simple
* Example: we want to calculate sum of a column across the date partition
* Sub-operation is simply to calculate the sum of each partition individually and single result is returned from each partition, once all of the sum have been calculated from required partitions then final result is produced by sum of all the individual sums 

# Without Slave processes 
* Let's take example of `avg`
* query is `select avg size from trades where date within 2014.04.03 2014.04.04`
* and available partitions in hdb are : 2014.04.01, 2014.04.02, 2014.04.03, 2014.04.04 and 2014.04.05
* Now sub-query which kdb+ send to each partition is something like : `select (sum size;count size) from trades` with result returned as `(220;6)`, `(600;30)`, .. 
* Once  all required results are returned, then final answer is calculated as sum of all size divided by sum of all counts
* result: `(220+600)%(6+30) = 22.78`

# with slave processes 
* KDB+ can also do sub-operations in parallel 
* sample query : `select avg size from trades where date within 2014.04.03 2014.04.04` 
* then each query will be executed parallely on each partitions by using slave processes - using `-s` flag
* once all the result are returned aggregation is done at the main thread 
* Sometimes this can result in slow peformance - in case all partitions are written on same disk then each slave will try to read disk at the same time which results in race conditions 
* the data can be spread across different file systems by having `par.txt` in root of HDB directory 

# with slave and par.txt 
* Now for above example assuming HDB is parted in 2 disks - disk1 having partitions `2014.04.01`, `2014.04.02` and `2014.04.03` and disk2 having partitions `2014.04.04` and `2014.04.05`
* Now queries can run parallely on disk1 and disk2 by each slave processes 
* this will result in less contention and speed up of query 

* let's look at an example 
```
q)\l fakedb.q
USAGE: makedb[NUM QUOTES;NUM TRADES] eg makedb[100000;10000]

makedb1[NUM QUOTES;NUM TRADES;DATE;RANDOMISED COUNT FACTOR] eg makedb1[100000;10000;.z.d;.3]

makehdb[HDBDIR;NUM DAYS;APPROXIMATE NUM QUOTES PER DAY; APPROXIMATE NUM TRADES PER DAY] eg makehdb[`:hdb; 5; 100000; 10000]

makecsv[CSVDIR;NUM DAYS;NUM QUOTES;NUM TRADES]

q)makehdb[`:hdb;15;100000;10000000]
2022.12.26T16:51:16.614 saving data for date 2014.04.21 to :hdb
2022.12.26T16:51:17.077 saving data for date 2014.04.22 to :hdb
2022.12.26T16:51:17.431 saving data for date 2014.04.23 to :hdb
2022.12.26T16:51:17.720 saving data for date 2014.04.24 to :hdb
2022.12.26T16:51:18.132 saving data for date 2014.04.25 to :hdb
2022.12.26T16:51:18.389 saving data for date 2014.04.28 to :hdb
2022.12.26T16:51:18.733 saving data for date 2014.04.29 to :hdb
2022.12.26T16:51:19.082 saving data for date 2014.04.30 to :hdb
2022.12.26T16:51:19.407 saving data for date 2014.05.01 to :hdb
2022.12.26T16:51:19.763 saving data for date 2014.05.02 to :hdb
2022.12.26T16:51:20.110 saving data for date 2014.05.05 to :hdb
2022.12.26T16:51:20.422 saving data for date 2014.05.06 to :hdb
2022.12.26T16:51:20.804 saving data for date 2014.05.07 to :hdb
2022.12.26T16:51:21.183 saving data for date 2014.05.08 to :hdb
2022.12.26T16:51:21.569 saving data for date 2014.05.09 to :hdb
q)\l hdb
q)tables`
`depth`quotes`trades
q)select distinct date from trades
date
----------
2014.04.21
2014.04.22
2014.04.23
2014.04.24
2014.04.25
2014.04.28
2014.04.29
2014.04.30
2014.05.01
2014.05.02
2014.05.05
2014.05.06
2014.05.07
2014.05.08
2014.05.09
```
* first let's take a sample query

```
q)q)select sum `long$size, max price, vwap:size wavg price by sym, 15 xbar time.minute from trades where date within 2014.04.21 2014.04.22
sym  minute| size     price vwap
-----------| -----------------------
AAPL 08:00 | 21303696 25.43 25.12335
AAPL 08:15 | 19856487 25.39 25.13909
AAPL 08:30 | 20348613 25.33 25.1434
```
* all the above functions can be map-reduce by kdb+ 
* now if we time this query, we get: 
```
q)\t select sum `long$size, max price, vwap:size wavg price by sym, 15 xbar time.minute from trades where date within 2014.04.21 2014.04.22
1791
```
* now let's start another process with slaves 

```
% q -s 4
KDB+ 4.0 2021.04.26 Copyright (C) 1993-2021 Kx Systems
m64/ 8(24)core 8192MB ranayk ykr-mbam1.local 127.0.0.1 EXPIRE 2023.07.24 1314.yogesh@gmail.com KXCE #73525

q)
q) // with slaves
q)\ts select sum `long$size, max price, vwap:size wavg price by sym, 15 xbar time.minute from trades where date within 2014.04.21 2014.04.22
1511 133312
q)\ts select sum `long$size, max price, vwap:size wavg price by sym, 15 xbar time.minute from trades where date within 2014.04.21 2014.04.22
468 127088
q)\ts select sum `long$size, max price, vwap:size wavg price by sym, 15 xbar time.minute from trades where date within 2014.04.21 2014.04.22
441 128400
q)\ts select sum `long$size, max price, vwap:size wavg price by sym, 15 xbar time.minute from trades where date within 2014.04.21 2014.04.22
443 128400
```
* we can't really mix functions that q knows how to map-reduce with others
* let's define a function which kdb+ doesn't know how to map-reduce and include it with one kdb+ knows how to map-reduce
```
q)f:{max x}
q)select fprice: f[price], max price by sym from trades
sym | fprice                                                                                    price                ..
----| ---------------------------------------------------------------------------------------------------------------..
AAPL| 25.45 25.38 24.26 24.3  25.12 24.63 24.14 25.61 24.05 23.46 24.26 24.01 23.69 24.57 24.44 25.45 25.38 24.26 24...
CSCO| 36.64 36.51 37.44 37.98 37.84 37.04 36.75 36.38 36.46 35.54 35.83 35.81 36.13 37.28 37.74 36.64 36.51 37.44 37...
DELL| 29.95 29.77 29.98 30.56 30.92 31.48 30.38 30.69 30.54 29.72 28.43 28.97 27.86 28.37 26.48 29.95 29.77 29.98 30...
GOOG| 42.11 42.36 42.23 41.44 40.6  40.23 39.76 39.71 38.59 39.84 39.25 37.17 34.88 35.98 37.08 42.11 42.36 42.23 41...
IBM | 43.7  42.9  43.84 43.88 43.68 43.81 43.55 46.7  49.42 50.49 50.33 50.47 50.47 49.56 48.42 43.7  42.9  43.84 43...
MSFT| 36.49 35.84 35.97 36.12 35.01 33.75 32.44 32.76 35.34 35.87 34.56 34.77 35.51 36.93 37.81 36.49 35.84 35.97 36...
NOK | 31.95 31.91 33.69 33.88 33.66 33.15 33.57 36.02 35.91 36.58 35.33 36.01 36.32 36.35 36.66 31.95 31.91 33.69 33...
ORCL| 32.73 32.23 31.57 31.43 30.48 30.53 31.51 31.59 31.96 31.91 30.95 30.93 30.56 31.27 32.16 32.73 32.23 31.57 31...
YHOO| 35.61 35.22 34.3  34.24 34.89 33.31 33.69 33.65 31.51 30.41 29.28 29.79 30.45 30.28 30.19 35.61 35.22 34.3  34...
q)\ts select fprice: f[price], max price by sym from trades
512 151006784
```
* no aggregation on the results from each partition 
* this  is because kdb+ parser doen't know how to aggregate the data
* we can do the aggregation manually 
```
q)update fprice: max each fprice, mprice: max each mprice from select fprice: f[price], mprice: max price by sym from trades
sym | fprice mprice
----| -------------
AAPL| 25.61  25.61
CSCO| 37.98  37.98
DELL| 31.48  31.48
GOOG| 42.36  42.36
IBM | 50.49  50.49
MSFT| 37.81  37.81
NOK | 36.66  36.66
ORCL| 32.73  32.73
YHOO| 35.61  35.61
```
* Now it returns the desired result
* but it is not efficient or ideal


* Now if we put user-defined function second, then..
```
q)select mprice: max price, fprice: f[price] by sym from trades
'price
  [0]  select mprice: max price, fprice: f[price] by sym from trades
       ^
q))
```
* query fails 
* reason for this: kdb+ only looks at first clause when aggreagating - here it sees it is map-reduce compatible 
* but it fails when it goes to 2nd clause which it doesn't know how to re-aggregate 
* few exceptions to this - if we query for single date, then..
```
q)select mprice: max price, fprice: f[price] by sym from trades where date=2014.04.21
sym | mprice fprice
----| -------------
AAPL| 25.45  25.45
CSCO| 36.64  36.64
DELL| 29.95  29.95
GOOG| 42.11  42.11
IBM | 43.7   43.7
MSFT| 36.49  36.49
NOK | 31.95  31.95
ORCL| 32.73  32.73
YHOO| 35.61  35.61
```
* it works fine ! 
* this is because there is no need to aggregate as querying only 1 partition 
* or we group by partition type - then..
```
q)select mprice: max price, fprice: f[price] by date from trades
date      | mprice fprice
----------| -------------
2014.04.21| 43.7   43.7
2014.04.22| 42.9   42.9
2014.04.23| 43.84  43.84
2014.04.24| 43.88  43.88
2014.04.25| 43.68  43.68
2014.04.28| 43.81  43.81
2014.04.29| 43.55  43.55
2014.04.30| 46.7   46.7
2014.05.01| 49.42  49.42
2014.05.02| 50.49  50.49
2014.05.05| 50.33  50.33
2014.05.06| 50.47  50.47
2014.05.07| 50.47  50.47
2014.05.08| 49.56  49.56
2014.05.09| 48.42  48.42
```
* also it works ! 
* but be careful on using the partition column first in group clause otherwise it fails again 
```
q)select mprice: max price, fprice: f[price] by date,sym from trades
date       sym | mprice fprice
---------------| -------------
2014.04.21 AAPL| 25.45  25.45
2014.04.21 CSCO| 36.64  36.64
2014.04.21 DELL| 29.95  29.95
2014.04.21 GOOG| 42.11  42.11
2014.04.21 IBM | 43.7   43.7
2014.04.21 MSFT| 36.49  36.49
q)select mprice: max price, fprice: f[price] by sym,date  from trades
'price
  [0]  select mprice: max price, fprice: f[price] by sym,date  from trades
       ^
q))
```
# Query Re-structure for slaves
* we  can re-structure queries to make use of slaves 
## no slave 
```
q)// price movement
q)\ts select 1 _ deltas price by sym, date, 15 xbar time.minute from trades where date within 2014.04.21 2014.04.22
769 1099976496
q)\ts t1: select maxmove: max each price, avgmove: avg each price, avgabs: avg each abs each price, medmove: med each price by date, sym, minute from select 1 _ deltas price by sym, date, 15 xbar time.minute from trades where date within 2014.04.21 2014.04.22
3157 1099976544
```

## with slaves
```
q)\ts select 1 _ deltas price by sym, date, 15 xbar time.minute from trades where date within 2014.04.21 2014.04.22
591 125488
q)\ts t1: select maxmove: max each price, avgmove: avg each price, avgabs: avg each abs each price, medmove: med each price by date, sym, minute from select 1 _ deltas price by sym, date, 15 xbar time.minute from trades where date within 2014.04.21 2014.04.22
2077 1795984
```
* little speed up but we can further optimise it 
* in above query - slave thread were only processing the first part of the query and second part of calculating max, avg, med were done in main thread 
* we can move whole calculation to slave as well 

```
q)\ts select (max;avg;{avg abs x};med)@\:1 _ deltas price by sym, date, 15 xbar time.minute from trades where date within 2014.04.21 2014.04.22
1327 120048
```
* now whole operation is done at slave side and we see significant speed up - about twice as fast 

* now we can try optimised query in process with no slave, then..
* it is taking approx same time 
```
q)\ts select (max;avg;{avg abs x};med)@\:1 _ deltas price by sym, date, 15 xbar time.minute from trades where date within 2014.04.21 2014.04.22
2450 939564736
```