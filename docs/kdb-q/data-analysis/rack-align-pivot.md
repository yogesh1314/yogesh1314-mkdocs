# Rack Align Pivot

## Racking 
* common operation with timeseries data is grouping data by time bucket using function `xbar`
* However using `xbar` we don't get buckets for all symbols for all possible time buckets, rather buckets are created for which data is present
* Let's create timeseries table
```q
q)tab:([]sym:`TCS`SBI`TCS`SBI`SBI;time: 09:00 09:01 09:20 09:32 09:34; size: 200 300 5000 400 350; price: 39.99 45.3 34.56 22.34 99.89)
q)tab
sym time  size price
--------------------
TCS 09:00 200  39.99
SBI 09:01 300  45.3
TCS 09:20 5000 34.56
SBI 09:32 400  22.34
SBI 09:34 350  99.89
q)select sum price by sym, 15 xbar time.minute from tab
sym minute| price
----------| ------
SBI 09:00 | 45.3
SBI 09:30 | 122.23
TCS 09:00 | 39.99
TCS 09:15 | 34.56
// here we dont see 09:15 bucket for SBI
```
* we can resolve above issue with racking - creating bucket for each 15 minute interval and merging it with table `tab`
* first we define `start`, `end` and `bucket`
```q
q)start:09:00
q)end:09:59
q)bucket:15
q)(end-start)%bucket
3.933333
q)ceiling (end-start)%bucket
4
q)til ceiling (end-start)%bucket
0 1 2 3
q)bucket*til ceiling (end-start)%bucket
0 15 30 45
q)start+bucket*til ceiling (end-start)%bucket
09:00 09:15 09:30 09:45
q)times: start+bucket*til ceiling (end-start)%bucket
q)times
09:00 09:15 09:30 09:45
```
* to create a rack we need original symbols from tables
```q
q)select distinct sym from tab
sym
---
TCS
SBI
q)rack: (`sym xasc select distinct sym from tab) cross ([]minute: times)
q)rack
sym minute
----------
SBI 09:00
SBI 09:15
SBI 09:30
SBI 09:45
TCS 09:00
TCS 09:15
TCS 09:30
TCS 09:45
```
* now we can perform same operation but with value for each sym for each window
```q
q)select sum price by sym,bucket xbar time.minute from tab //earlier
sym minute| price
----------| ------
SBI 09:00 | 45.3
SBI 09:30 | 122.23
TCS 09:00 | 39.99
TCS 09:15 | 34.56
q)rack#select sum price by sym,bucket xbar time.minute from tab //after using rack
sym minute| price
----------| ------
SBI 09:00 | 45.3
SBI 09:15 |
SBI 09:30 | 122.23
SBI 09:45 |
TCS 09:00 | 39.99
TCS 09:15 | 34.56
TCS 09:30 |
TCS 09:45 |
```
* we can fill the blank data with `fills` or `^`
```q
q)update 0^price from rack#select sum price by sym,bucket xbar time.minute from tab
sym minute| price
----------| ------
SBI 09:00 | 45.3
SBI 09:15 | 0
SBI 09:30 | 122.23
SBI 09:45 | 0
TCS 09:00 | 39.99
TCS 09:15 | 34.56
TCS 09:30 | 0
TCS 09:45 | 0
q)update fills size by sym from update 0^price from rack#select sum sum price, last size by sym,bucket xbar time.minute from tab
sym minute| price  size
----------| -----------
SBI 09:00 | 45.3   300
SBI 09:15 | 0      300
SBI 09:30 | 122.23 350
SBI 09:45 | 0      350
TCS 09:00 | 39.99  200
TCS 09:15 | 34.56  5000
TCS 09:30 | 0      5000
TCS 09:45 | 0      5000
```
* we can create a function to do same on HDB
```q
q)\l fakedb.q
USAGE: makedb[NUM QUOTES;NUM TRADES] eg makedb[100000;10000]

makedb1[NUM QUOTES;NUM TRADES;DATE;RANDOMISED COUNT FACTOR] eg makedb1[100000;10000;.z.d;.3]

makehdb[HDBDIR;NUM DAYS;APPROXIMATE NUM QUOTES PER DAY; APPROXIMATE NUM TRADES PER DAY] eg makehdb[`:hdb; 5; 100000; 10000]

makecsv[CSVDIR;NUM DAYS;NUM QUOTES;NUM TRADES]

q)
q)makehdb[`:hdb;5;100000;10000]
2022.10.05T17:37:30.075 saving data for date 2014.04.21 to :hdb
2022.10.05T17:37:30.139 saving data for date 2014.04.22 to :hdb
2022.10.05T17:37:30.225 saving data for date 2014.04.23 to :hdb
2022.10.05T17:37:30.306 saving data for date 2014.04.24 to :hdb
2022.10.05T17:37:30.380 saving data for date 2014.04.25 to :hdb
q)\l hdb
q)\a
`depth`quotes`trades
q)// function to create rack
q)createrack:{[start;end;bucket;symbol] times:start+bucket*til ceiling (end-start:bucket xbar start)%bucket}
q)createrack[2022.10.05D09:00;2022.10.05D09:59;0D00:15;`SBI]
2022.10.05D09:00:00.000000000 2022.10.05D09:15:00.000000000 2022.10.05D09:30:..
q)//only create list of times
q)//lets add cross with sym logic
q)createrack:{[start;end;bucket;symbol] times:start+bucket*til ceiling (end-start:bucket xbar start)%bucket; ([]sym:symbol,()) cross ([]time: times)}
q)createrack[2022.10.05D09:00;2022.10.05D09:59;0D00:15;`SBI]
sym time
---------------------------------
SBI 2022.10.05D09:00:00.000000000
SBI 2022.10.05D09:15:00.000000000
SBI 2022.10.05D09:30:00.000000000
SBI 2022.10.05D09:45:00.000000000
```
* we need to create another function for using `createrack` function with hdb data
```q
q)jointables:{[start;end;bucket;symbol] createrack[start;end;bucket;symbol]#select sum price by sym, bucket xbar time from trades where date within `date$(start;end), sym in symbol, time within `timestamp$(start;end)}
q)jointables[2014.04.21D09:00;2014.04.21D12:59;0D00:15;`AAPL]
sym  time                         | price
----------------------------------| -------
AAPL 2014.04.21D09:00:00.000000000| 853.09
AAPL 2014.04.21D09:15:00.000000000| 823.7
AAPL 2014.04.21D09:30:00.000000000| 794.11
AAPL 2014.04.21D09:45:00.000000000| 799.56
AAPL 2014.04.21D10:00:00.000000000| 624.37
AAPL 2014.04.21D10:15:00.000000000| 650.2
AAPL 2014.04.21D10:30:00.000000000| 803.53
AAPL 2014.04.21D10:45:00.000000000| 933.05
AAPL 2014.04.21D11:00:00.000000000| 785.23
AAPL 2014.04.21D11:15:00.000000000| 1034.56
AAPL 2014.04.21D11:30:00.000000000| 779.19
AAPL 2014.04.21D11:45:00.000000000| 1205.52
AAPL 2014.04.21D12:00:00.000000000| 873.81
AAPL 2014.04.21D12:15:00.000000000| 991.83
AAPL 2014.04.21D12:30:00.000000000| 945.7
AAPL 2014.04.21D12:45:00.000000000| 823.28
```
## Align 

* Another common problem is to align two asynchronous timeseries data where the time rarely match 
* Alignment can be for the same dataset but different instruments for example aligning `GOOG` with `AAPL` 
* There are three main approached to aligning 
    1. we can use bucketing to align - have buckets of common time and then join 
    2. use `asof` join
    3. use `fill` join where we want all data at every timepoint
* we can load the data from HDB
```
q)trades1: select time, sym, price, size from trades where date=2014.04.21
q)quotes1: select time, sym, bid,ask from quotes where date=2014.04.21
```
* `aj` will join prevailing quote price to each trade i.e; the quote which occured just before will be joined on to that trade 
* `aj` makes exact match on the `sym` and as-of match on `time`
```
q)aj[`sym`time;trades1;quotes1]
time                          sym  price size bid   ask
---------------------------------------------------------
2014.04.21D08:00:12.155000000 AAPL 25.31 2450 25.31 25.33
2014.04.21D08:00:42.186000000 AAPL 25.32 289  25.32 25.34
2014.04.21D08:00:51.764000000 AAPL 25.34 3167 25.34 25.37
2014.04.21D08:00:54.526000000 AAPL 25.34 6474 25.34 25.37
2014.04.21D08:00:57.071000000 AAPL 25.34 1335 25.34 25.37
2014.04.21D08:01:19.773000000 AAPL 25.4  317  25.36 25.4
```  
* Now, what if we want to align prices and cumulative volumes of two different syms 
  * example: we want to align prevailing `GOOG` data to `AAPL` data 
* For this, first we create a table for price and total volume for `AAPL` trades and same for `GOOG` trades 
```
q)apple: select time, appleprice:price, applevol: sums size from trades1 where sym in `AAPL
q)apple
time                          appleprice applevol
-------------------------------------------------
2014.04.21D08:00:12.155000000 25.31      2450
2014.04.21D08:00:42.186000000 25.32      2739
2014.04.21D08:00:51.764000000 25.34      5906
2014.04.21D08:00:54.526000000 25.34      12380
2014.04.21D08:00:57.071000000 25.34      13715
q)google: select time, googprice:price, googvol: sums size from trades1 where sym in `GOOG
q)google
time                          googprice googvol
-----------------------------------------------
2014.04.21D08:00:01.184000000 41.3      2422
2014.04.21D08:00:23.103000000 41.3      4313
2014.04.21D08:00:49.515000000 41.29     5444
2014.04.21D08:00:57.435000000 41.26     12207
2014.04.21D08:01:02.314000000 41.29     18694
```
* now we can use `aj` to align the two tables
```
q)aj[`time;apple;google]
time                          appleprice applevol googprice googvol
-------------------------------------------------------------------
2014.04.21D08:00:12.155000000 25.31      2450     41.3      2422
2014.04.21D08:00:42.186000000 25.32      2739     41.3      4313
2014.04.21D08:00:51.764000000 25.34      5906     41.29     5444
2014.04.21D08:00:54.526000000 25.34      12380    41.29     5444
2014.04.21D08:00:57.071000000 25.34      13715    41.29     5444
```
## Offset Alignment
* it is shifting the time field of one table to calculate something either side of another
* using similar approach, calculate for every trade the VWAP for the 5 minutes following the trade (excluding the contribution from current trade)
* we can calculate VWAP using the running sum of the size and the running sum of the size*price
```
q)makedb[100000;10000]
q)tables`
`depth`quotes`trades
q)trades1: update sumps: sums price*size, sumsize: sums size by sym from trades
q)trades1
time                          sym  src price size sumps    sumsize
------------------------------------------------------------------
2022.12.25D08:00:04.569000000 CSCO N   35.48 3827 135782   3827
2022.12.25D08:00:22.311000000 MSFT L   36.11 354  12782.94 354
2022.12.25D08:00:27.438000000 IBM  N   43.54 2589 112725.1 2589
2022.12.25D08:00:27.511000000 YHOO L   35.54 6956 247216.2 6956
2022.12.25D08:00:27.906000000 IBM  N   43.54 2397 217090.4 4986
2022.12.25D08:00:30.278000000 NOK  L   31.76 353  11211.28 353
```
* Now we want to take just the running vwap from trades1 and shift it all by 5 mins, we can do this by creating new table 

```
q)shiftreades: select time-0D00:05,sym,sumplus5: sumps, sumsizeplus5: sumsize from trades1
q)shiftreades
time                          sym  sumplus5 sumsizeplus5
--------------------------------------------------------
2022.12.25D07:55:04.569000000 CSCO 135782   3827
2022.12.25D07:55:22.311000000 MSFT 12782.94 354
2022.12.25D07:55:27.438000000 IBM  112725.1 2589
2022.12.25D07:55:27.511000000 YHOO 247216.2 6956
2022.12.25D07:55:27.906000000 IBM  217090.4 4986
2022.12.25D07:55:30.278000000 NOK  11211.28 353
```
* Now we want to join this time shifted vwap info back onto the non-shifted vwap using `aj`
* we can have desired result by subtracting shifted from non-shifted 

```
q)aligntab: aj[`sym`time;delete src,size from trades1;shiftreades]
q)aligntab
time                          sym  price sumps    sumsize sumplus5 sumsizeplus5
-------------------------------------------------------------------------------
2022.12.25D08:00:04.569000000 CSCO 35.48 135782   3827    1492906  42000
2022.12.25D08:00:22.311000000 MSFT 36.11 12782.94 354     507052.6 14077
2022.12.25D08:00:27.438000000 IBM  43.54 112725.1 2589    1258692  28796
2022.12.25D08:00:27.511000000 YHOO 35.54 247216.2 6956    1170549  33014
2022.12.25D08:00:27.906000000 IBM  43.54 217090.4 4986    1258692  28796
2022.12.25D08:00:30.278000000 NOK  31.76 11211.28 353     498678.1 15746
2022.12.25D08:00:30.579000000 CSCO 35.45 161518.7 4553    1571526  44201
2022.12.25D08:00:32.203000000 DELL 29.1  30758.7  1057    734669.4 25218

q)update vwapplus5: (sumplus5-sumps)%sumsizeplus5 - sumsize from delete date, time from aligntab
sym  price sumps    sumsize sumplus5 sumsizeplus5 vwapplus5
-----------------------------------------------------------
CSCO 35.48 135782   3827    1492906  42000        35.55193
MSFT 36.11 12782.94 354     507052.6 14077        36.01761
IBM  43.54 112725.1 2589    1258692  28796        43.72753
YHOO 35.54 247216.2 6956    1170549  33014        35.43376
IBM  43.54 217090.4 4986    1258692  28796        43.74641
NOK  31.76 11211.28 353     498678.1 15746        31.66809
CSCO 35.45 161518.7 4553    1571526  44201        35.56313
DELL 29.1  30758.7  1057    734669.4 25218        29.13417
ORCL 32.25 181728.8 5635    1451326  45032        32.22575
ORCL 32.26 268250.1 8317    1451326  45032        32.22324
```

## Pivoting 

* can be used to move row values into column values