# Where Clause

* used to filter data from in-memory and on-disk tables
```
q)tab:([]price:1000000?100f;qty:1000000?100)
q)tab
price    qty
------------
68.78151 92
1.897575 93
56.70071 84
48.33407 20
82.02713 22
..
```
* where clause cascades the data - filter conditions are applied in order they are written and only subset of data is passed to next one
* we need to be careful about the order of filter conditions
* we should be cutting the dataset into **as small as possible as fast as possible**
* filter conditions are applied left to right - thus if we apply `price` filter first less number of rows are left for applying `qty` filter, similarly if we apply `qty` filter first then `price` filter is applied to more number of rows thus taking more time
```
q)count select from tab where qty<90
899773
q)count select from tab where price>50
500309
q)\t select from tab where price>50, qty<90
64
q)\t select from tab where qty<90, price>50
77
```

## Historic database
* we should apply `where` first on partition
* if partitioned by date, first filter condition should be on date
* there is a huge difference in fetching data from HDB if first filter is not partition(date/month/year)
```
q)\l fakedb.q
q)makehdb[`:hdb;10;100000;10]
2020.12.24T10:09:26.303 saving data for date 2014.04.21 to :hdb
2020.12.24T10:09:27.947 saving data for date 2014.04.22 to :hdb
2020.12.24T10:09:29.396 saving data for date 2014.04.23 to :hdb
2020.12.24T10:09:31.115 saving data for date 2014.04.24 to :hdb
2020.12.24T10:09:33.085 saving data for date 2014.04.25 to :hdb
2020.12.24T10:09:34.825 saving data for date 2014.04.28 to :hdb
2020.12.24T10:09:36.320 saving data for date 2014.04.29 to :hdb
2020.12.24T10:09:38.063 saving data for date 2014.04.30 to :hdb
2020.12.24T10:09:40.130 saving data for date 2014.05.01 to :hdb
2020.12.24T10:09:42.148 saving data for date 2014.05.02 to :hdb
q)\l hdb
q)q)count quotes
1027292
q)select distinct date from quotes
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
q) // HDB created and loaded
```
* query without date as first filter
```
q)\t:100 select from quotes where sym=`AAPL, bsize>5000, date=2014.04.21
2849
```
* query with date as first filter
```
q)\t:100 select from quotes where date=2014.04.21, sym=`AAPL, bsize>5000
328
```
* difference is almost **7 times** which is quite huge


## Attributes
* if attributes are present in table - can be checked using `meta table_name`
* in `where` we should use attributes columns just after partition columns, thus for on-disk table `where` filter order would be:
1. partition column
2. columns with attributes
3. other columns
and for in-memory tables `where` order would be:
1. columns with attributes
2. other columns

```
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
q)\t:100 select from quotes where date=2014.04.23, sym=`AAPL,bid>10
370
q)\t:100 select from quotes where date=2014.04.23, bid>10, sym=`AAPL
1035
q) // difference is almost 3 times
```


## tuple matches
* allows us to apply multiple filters on same columns in single query
* eg: we want IBM trades from src L, CSCO trades from src L or N and MSFT trades from src O
* **method 1:** apply where individually and join results
```
q)(select from trades where date=2014.04.21, sym=`IBM, src=`L),(select from trades where date=2014.04.21, sym=`MSFT, src in `O),(select from trades where date=2014.04.21, sym=`CSCO, src in `L`N)
date       sym  time                          src price size
------------------------------------------------------------
2014.04.21 IBM  2014.04.21D09:35:14.517000000 L   43.37 6771
2014.04.21 IBM  2014.04.21D12:56:35.834000000 L   42.36 3389
2014.04.21 MSFT 2014.04.21D08:30:37.642000000 O   36.4  855
2014.04.21 CSCO 2014.04.21D08:20:47.172000000 N   35.74 1615
```
* **method 2:** use tuple matches
* we define our conditions in a table
* apply where with condition table
```
q)toget:([]sym:`IBM`MSFT`CSCO`CSCO;src:`L`O`L`N)
q)toget
sym  src
--------
IBM  L
MSFT O
CSCO L
CSCO N
q)select from trades where date=2014.04.21, ([]sym; src) in toget
date       sym  time                          src price size
------------------------------------------------------------
2014.04.21 CSCO 2014.04.21D08:20:47.172000000 N   35.74 1615
2014.04.21 IBM  2014.04.21D09:35:14.517000000 L   43.37 6771
2014.04.21 IBM  2014.04.21D12:56:35.834000000 L   42.36 3389
2014.04.21 MSFT 2014.04.21D08:30:37.642000000 O   36.4  855
q)\t:1000 select from trades where date=2014.04.21, ([]sym; src) in toget
1613
```
* we can also attributes to bring down query time further down
```
q)\t:1000 select from trades where date=2014.04.21, sym in toget`sym, ([]sym; src) in toget
1397
q) // around 200ms difference
```

## Where clause ordering
* while applying where clause ordering is most important part
* eg: in a sample table trades
```
q)trades:([]price:50 51 70; size: 600 800 300)
q)trades
price size
----------
50    600
51    800
70    300
```
* we want to search for trades which occurred at max price also has size more than 500
* from looking at table trades we see there is no such trade
* but if we are not careful in the order of where conditions we can get incorrect results
```
q)select from trades where size>500, price=max price
price size
----------
51    800
q) // this result is incorrect
```
* we get incorrect result - because `price` filter should have been applied on whole trades table, instead it is applied result of `size` filter
* query with correct order would be
```
q)select from trades where price=max price, size>500
price size
----------
```
* which results into 0 such trades, which is correct


## `fby` - filter by
* used to apply aggregations within the where clause
* eg: get trades with max price for each sym for a date
```
q)// first attempt
q)select from trades where date=2014.04.21, price=max price
date       sym time                          src price size
-----------------------------------------------------------
2014.04.21 IBM 2014.04.21D09:35:14.517000000 L   43.37 6771
```
* but this will only give us trades with max price across all syms not for each syms
```
q)// second attempt
q)select by sym from trades where date=2014.04.21, price=max price
sym| date       time                          src price size
---| -------------------------------------------------------
IBM| 2014.04.21 2014.04.21D09:35:14.517000000 L   43.37 6771
```
* this also fails as `where` clause is applied before `by` clause
```
q) // using fby - apply aggregation of max price and filter by sym
q)select from trades where date=2014.04.21, price=(max;price) fby sym
date       sym  time                          src price size
------------------------------------------------------------
2014.04.21 CSCO 2014.04.21D08:20:47.172000000 N   35.74 1615
2014.04.21 DELL 2014.04.21D13:15:14.289000000 L   29.63 1490
2014.04.21 GOOG 2014.04.21D13:57:48.315000000 L   41.4  989
2014.04.21 IBM  2014.04.21D09:35:14.517000000 L   43.37 6771
2014.04.21 MSFT 2014.04.21D08:30:37.642000000 O   36.4  855
2014.04.21 NOK  2014.04.21D15:06:51.099000000 N   31.64 4452
2014.04.21 ORCL 2014.04.21D12:28:32.455000000 L   32.6  15
q) // now this seems, right !!
```
* we can also aggregate by more than 1 field (sym, src, etc..)
```
q)select from trades where date=2014.04.21, price=(max;price) fby ([]sym;src)
date       sym  time                          src price size
------------------------------------------------------------
2014.04.21 CSCO 2014.04.21D08:20:47.172000000 N   35.74 1615
2014.04.21 DELL 2014.04.21D13:15:14.289000000 L   29.63 1490
2014.04.21 GOOG 2014.04.21D13:57:48.315000000 L   41.4  989
2014.04.21 IBM  2014.04.21D09:35:14.517000000 L   43.37 6771
2014.04.21 IBM  2014.04.21D16:09:21.276000000 O   41.89 4567
2014.04.21 MSFT 2014.04.21D08:30:37.642000000 O   36.4  855
2014.04.21 NOK  2014.04.21D15:06:51.099000000 N   31.64 4452
2014.04.21 ORCL 2014.04.21D12:28:32.455000000 L   32.6  15
2014.04.21 ORCL 2014.04.21D16:19:28.873000000 N   31.33 2269
```
