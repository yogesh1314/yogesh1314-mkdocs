# The `by` clause
* used to group data
* tables are set of lists
```
q)/setup
q)makedb[1000000;200000]
tables[]
q)`depth`quotes`trades
```
* we can apply group data using `by` without aggregation - aggregation function `last` is applied by default
```
q)select by sym from trades
sym | time                          src price size
----| --------------------------------------------
AAPL| 2020.12.25D16:29:55.722000000 N   29.89 210
CSCO| 2020.12.25D16:29:56.120000000 O   35.29 2803
DELL| 2020.12.25D16:29:57.739000000 N   28.66 1889
GOOG| 2020.12.25D16:29:56.958000000 N   43.87 1254
IBM | 2020.12.25D16:29:58.656000000 N   37.92 4379
MSFT| 2020.12.25D16:29:59.677000000 L   38.88 6493
NOK | 2020.12.25D16:29:57.448000000 N   34.55 1092
ORCL| 2020.12.25D16:29:57.487000000 L   31.47 2685
YHOO| 2020.12.25D16:29:59.662000000 N   37.01 5305
```
* grouping with aggregation also works

```
q)select max price, sum size by sym from trades
sym | price size
----| --------------
AAPL| 30.39 59274749
CSCO| 38.11 58247173
DELL| 31.11 58330222
GOOG| 44.02 57776533
IBM | 43.6  58945930
MSFT| 41.2  58150838
NOK | 35.56 57899767
ORCL| 33.46 58335324
YHOO| 38.11 59446175
```
* in case of grouping by 2 columns, they are grouped in order they are queried
* in example below, it's grouped first by `sym` and then again by `src`
```
q)select max price, sum size by sym,src from trades
sym  src| price size
--------| --------------
AAPL L  | 30.36 20012403
AAPL N  | 30.39 19782620
AAPL O  | 30.33 19479726
CSCO L  | 38.08 19537950
CSCO N  | 38.11 19351658
CSCO O  | 38.1  19357565
DELL L  | 31.09 19188543
DELL N  | 31.09 19677846
DELL O  | 31.11 19463833
GOOG L  | 44.02 19686298
GOOG N  | 43.95 19884630
GOOG O  | 44.02 18205605
IBM  L  | 43.55 19806381
IBM  N  | 43.54 19809111
IBM  O  | 43.6  19330438
MSFT L  | 41.2  19272596
MSFT N  | 41.17 19105562
MSFT O  | 41.19 19772680
NOK  L  | 35.49 19366132
NOK  N  | 35.56 19586497
```
* we can also get values without aggregation
```
q)select max price, size by sym,src from trades
sym  src| price size                                                         ..
--------| -------------------------------------------------------------------..
AAPL L  | 30.36 1344 922 1223 9988 5087 4250 384 6328 3506 4901 1216 6507 236..
AAPL N  | 30.39 119 7205 125 137 2385 6201 5277 781 434 778 5221 165 3665 697..
AAPL O  | 30.33 1132 4958 4875 2746 1006 686 636 3929 4680 7888 8422 4246 268..
CSCO L  | 38.08 36 3243 1037 2923 994 2832 1613 1530 452 1381 1334 9846 1676 ..
CSCO N  | 38.11 2144 1363 7463 5020 2068 1329 2075 4074 549 5471 812 7790 251..
CSCO O  | 38.1  154 2171 797 1579 1868 2098 2910 1525 921 744 2132 6843 3296 ..
DELL L  | 31.09 4317 4648 27 5416 1379 3476 2331 154 1246 5158 558 5662 4337 ..
DELL N  | 31.09 2356 357 170 5101 460 2933 751 4031 1950 4837 5847 5596 244 1..
DELL O  | 31.11 22 7472 376 440 2789 1247 520 3012 272 5445 6697 581 1201 108..
```
* data is sorted by columns in ascending order of `by` clause
* in example below trades are first sorted ascending by `time` then by `sym`
```
q)select max price by time, sym from trades
time                          sym | price
----------------------------------| -----
2020.12.25D08:00:00.029000000 CSCO| 35.46
2020.12.25D08:00:00.229000000 NOK | 31.82
2020.12.25D08:00:00.250000000 YHOO| 35.5
2020.12.25D08:00:00.861000000 GOOG| 41.41
2020.12.25D08:00:00.946000000 IBM | 43.54
2020.12.25D08:00:00.958000000 MSFT| 36.09
2020.12.25D08:00:01.156000000 CSCO| 35.46
2020.12.25D08:00:01.255000000 NOK | 31.78
2020.12.25D08:00:01.272000000 MSFT| 36.09
2020.12.25D08:00:01.527000000 NOK | 31.76
2020.12.25D08:00:01.546000000 AAPL| 25.36
2020.12.25D08:00:01.629000000 MSFT| 36.08
2020.12.25D08:00:01.680000000 YHOO| 35.54
2020.12.25D08:00:01.836000000 DELL| 29.07
2020.12.25D08:00:01.889000000 CSCO| 35.45
2020.12.25D08:00:02.127000000 DELL| 29.07
2020.12.25D08:00:02.161000000 YHOO| 35.5
2020.12.25D08:00:02.824000000 YHOO| 35.5
2020.12.25D08:00:03.191000000 YHOO| 35.52
2020.12.25D08:00:03.309000000 GOOG| 41.3
```

## HLOC/OHLC
* high, low, open and close / open, high, low and close
```
q)select high:max price, low:min price, open:first price, close:last price by sym from trades
sym | high  low   open  close
----| -----------------------
AAPL| 30.39 24.61 25.36 29.89
CSCO| 38.11 34.25 35.46 35.29
DELL| 31.11 28.48 29.07 28.66
GOOG| 44.02 39.9  41.41 43.87
IBM | 43.6  37.66 43.54 37.92
MSFT| 41.2  35.9  36.09 38.88
NOK | 35.56 29.09 31.82 34.55
ORCL| 33.46 30.55 32.22 31.47
YHOO| 38.11 32.83 35.5  37.01

```
## Function in `by` clause

### `xbar`
* frequently used to split data into time buckets
* sample use case : select maximum price for each sym for every hour/every half hour
* `n xbar list` is used for bucketing `list` of any q entity into `n` number of buckets
 ```
 q)select max price by 1 xbar time.hh from trades
hh| price
--| -----
8 | 43.6
9 | 42.62
10| 43.06
11| 42.61
12| 42.57
13| 42.27
14| 42.03
15| 43.75
16| 44.02
q)select max price by 0.5 xbar time.hh from trades
hh | price
---| -----
4  | 43.6
4.5| 42.62
5  | 43.06
5.5| 42.61
6  | 42.57
6.5| 42.27
7  | 42.03
7.5| 43.75
8  | 44.02
q)select max price, sum size by sym, 5 xbar time.minute from trades
sym  minute| price size
-----------| ------------
AAPL 08:00 | 25.44 559983
AAPL 08:05 | 25.41 658484
AAPL 08:10 | 25.45 568858
AAPL 08:15 | 25.05 590919
AAPL 08:20 | 24.97 592187
AAPL 08:25 | 25.18 618660
AAPL 08:30 | 24.94 536795
q) // not just for temporal data, xbar works on all types of q datatypes
q)select sum size by sym, 5 xbar price from trades
sym  price| size
----------| --------
AAPL 20   | 2778333
AAPL 25   | 55483203
AAPL 30   | 1013213
CSCO 30   | 4698809
CSCO 35   | 53548364
DELL 25   | 39015640
DELL 30   | 19314582
GOOG 35   | 78225
GOOG 40   | 57698308
IBM  35   | 18314146
IBM  40   | 40631784
MSFT 35   | 52387686
MSFT 40   | 5763152
NOK  25   | 11650507
NOK  30   | 42933232
NOK  35   | 3316028
ORCL 30   | 58335324
YHOO 30   | 20237630
YHOO 35   | 39208545
 ```

### tick direction example

* we want to add new column `dir` showing direction of price : +1 = increase in price, -1 = decrease in price, 0 = no change
* we define a function `tradedirection`
```
q)tradedirection:{signum deltas x}
q)tradedirection:{signum deltas x}
q)update dir:tradedirection price by sym from trades
time                          sym  src price size dir
-----------------------------------------------------
2020.12.25D08:00:00.029000000 CSCO O   35.46 154  0
2020.12.25D08:00:00.229000000 NOK  L   31.82 690  0
2020.12.25D08:00:00.250000000 YHOO L   35.5  1583 0
2020.12.25D08:00:00.861000000 GOOG N   41.41 3183 0
2020.12.25D08:00:00.946000000 IBM  N   43.54 7605 0
2020.12.25D08:00:00.958000000 MSFT N   36.09 1592 0
2020.12.25D08:00:01.156000000 CSCO N   35.46 2144 0
2020.12.25D08:00:01.255000000 NOK  O   31.78 508  -1
2020.12.25D08:00:01.272000000 MSFT L   36.09 4960 0
2020.12.25D08:00:01.527000000 NOK  L   31.76 7832 -1
..
```
* here we see for first row dir is 1 which is incorrect and should be 0
* hence, we re-define our `tradedirection` function
* now we are using diadic form of `deltas` function
```
q)tradedirection:{signum deltas[first x;x]}
q)update dir:tradedirection price by sym from trades
time                          sym  src price size dir
-----------------------------------------------------
2020.12.25D08:00:00.029000000 CSCO O   35.46 154  0
2020.12.25D08:00:00.229000000 NOK  L   31.82 690  0
2020.12.25D08:00:00.250000000 YHOO L   35.5  1583 0
2020.12.25D08:00:00.861000000 GOOG N   41.41 3183 0
2020.12.25D08:00:00.946000000 IBM  N   43.54 7605 0
2020.12.25D08:00:00.958000000 MSFT N   36.09 1592 0
2020.12.25D08:00:01.156000000 CSCO N   35.46 2144 0
2020.12.25D08:00:01.255000000 NOK  O   31.78 508  -1
2020.12.25D08:00:01.272000000 MSFT L   36.09 4960 0
2020.12.25D08:00:01.527000000 NOK  L   31.76 7832 -1
..
```
* now if we want to use `dir` column in grouping, to group by sym and upticks(dir=+1)/downticks(dir=-1)/no movement(dir=0)
* we do this by first updating `dir` in trades then group them by `sym` and `dir`
```
q)select sum size by sym, dir from update dir:tradedirection price by sym from trades
sym  dir| size
--------| --------
AAPL -1 | 23669600
AAPL 0  | 10796074
AAPL 1  | 24809075
CSCO -1 | 24273006
CSCO 0  | 10492199
CSCO 1  | 23481968
DELL -1 | 23765475
DELL 0  | 11081873
DELL 1  | 23482874
GOOG -1 | 23767777
..
```
* we can also use `fby` in order to get same results in single statement
```
q)select sum size by sym, dir:(tradedirection;price) fby sym from trades
sym  dir| size
--------| --------
AAPL -1 | 23669600
AAPL 0  | 10796074
AAPL 1  | 24809075
CSCO -1 | 24273006
CSCO 0  | 10492199
CSCO 1  | 23481968
DELL -1 | 23765475
DELL 0  | 11081873
DELL 1  | 23482874
GOOG -1 | 23767777
..
```
* **same can be used for calculating a value which is dependent on another value in the group**
* **example** : we want to know total size of trades for each sym where price is more than 10 tick moving average
```
q)select sum size by sym, price>(mavg[10];price) fby sym from trades
sym  sym1| size
---------| --------
AAPL 0   | 29193756
AAPL 1   | 30080993
CSCO 0   | 29465592
CSCO 1   | 28781581
DELL 0   | 29500402
DELL 1   | 28829820
GOOG 0   | 28792193
GOOG 1   | 28984340
IBM  0   | 30403277
IBM  1   | 28542653
..
```

### trades in size group
* Suppose we want to classify trades into 3 groups:
1. size<1000 - small
2. 1000<size<9000 - medium
3. size>9000 - big
* we can do it using
1. multiple selects
2. have a function and apply `fby` in `by` clause
```
q) // my attempt
q)sizegroup:{?[x<1000;`small;?[x within 1000 8999; `medium;`big]]}
q)select by sym, sg:(sizegroup;size) fby sym from trades
sym  sg    | time                          src price size
-----------| --------------------------------------------
AAPL big   | 2020.12.25D16:22:11.344000000 N   30.05 9746
AAPL medium| 2020.12.25D16:29:54.842000000 O   29.87 1568
AAPL small | 2020.12.25D16:29:55.722000000 N   29.89 210
CSCO big   | 2020.12.25D16:28:52.974000000 N   35.27 9805
CSCO medium| 2020.12.25D16:29:56.120000000 O   35.29 2803
CSCO small | 2020.12.25D16:29:53.908000000 L   35.38 906
DELL big   | 2020.12.25D16:29:01.330000000 O   29.02 9338
DELL medium| 2020.12.25D16:29:57.739000000 N   28.66 1889
DELL small | 2020.12.25D16:29:41.032000000 O   28.69 386
GOOG big   | 2020.12.25D16:28:03.979000000 O   44.02 9368
..
```
```
q) // instructor's attempt using bin
q)sizegroup2:{`small`medium`big 0 1000 9000 bin x}
q)select by sym, (sizegroup2;size) fby sym from trades
sym  sym1  | time                          src price size
-----------| --------------------------------------------
AAPL big   | 2020.12.25D12:47:22.215000000 L   25.27 9008
AAPL medium| 2020.12.25D16:29:14.600000000 L   25.48 1226
AAPL small | 2020.12.25D16:29:52.700000000 L   25.48 167
CSCO big   | 2020.12.25D14:04:40.319000000 N   36.22 9027
CSCO medium| 2020.12.25D16:29:17.739000000 N   36.4  4333
CSCO small | 2020.12.25D16:28:08.296000000 N   36.46 815
DELL medium| 2020.12.25D16:27:11.848000000 O   29.81 4049
DELL small | 2020.12.25D16:22:44.943000000 N   29.82 39
GOOG medium| 2020.12.25D16:26:30.366000000 N   41.84 4956
GOOG small | 2020.12.25D16:23:11.310000000 N   41.8  264
IBM  big   | 2020.12.25D14:35:04.732000000 N   46.19 9051
IBM  medium| 2020.12.25D16:28:57.502000000 O   46.89 1322
IBM  small | 2020.12.25D16:22:03.114000000 N   47.04 796
MSFT big   | 2020.12.25D16:22:42.020000000 N   35.59 9636
MSFT medium| 2020.12.25D16:14:50.382000000 N   35.59 1055
MSFT small | 2020.12.25D16:21:07.118000000 L   35.55 937
NOK  big   | 2020.12.25D10:29:07.054000000 L   31.16 9045
NOK  medium| 2020.12.25D16:22:24.261000000 L   31.04 2457
NOK  small | 2020.12.25D16:27:15.083000000 O   31.14 357
ORCL big   | 2020.12.25D11:32:42.294000000 O   33.26 9812
..
```
```
q)// solution with bin needs size list to be sorted, solution with ? would work otherwise as well
q)// performance comparison
q) count trades`size
2000
q)sizegroup
{?[x<1000;`small;?[x within 1000 8999; `medium;`big]]}
q)\t:10000 sizegroup trades`size
2069
q)\t:10000 sizegroup trades`size
2083
q)sizegroup2
{`small`medium`big 0 1000 9000 bin x}
q)\t:10000 sizegroup2 trades`size
1675
q)\t:10000 sizegroup2 trades`size
1677
q) // solution with bin is more optimized, faster by ~300-400ms, only restriction is list needs to be sorted, otherwise solution with ? would work in both cases sorted or otherwise
```
