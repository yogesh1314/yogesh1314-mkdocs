In-depth Profit and Loss(PnL) trade analysis

Suppose we have a set of trades, we will calculate PnL of each trades at different timepoint, either before or after current trade time

We are assuming every trade is a bi-trade and we are marketing trade price rather than quoted price 

* load in fakedb.q and load in 100000 trades
* create a table with shifted time of 5 minutes
* join this to original table using `aj` - this will give current trade price and trade price before 5 minutes

```
q)\l fakedb.q
USAGE: makedb[NUM QUOTES;NUM TRADES] eg makedb[100000;10000]

makedb1[NUM QUOTES;NUM TRADES;DATE;RANDOMISED COUNT FACTOR] eg makedb1[100000;10000;.z.d;.3]

makehdb[HDBDIR;NUM DAYS;APPROXIMATE NUM QUOTES PER DAY; APPROXIMATE NUM TRADES PER DAY] eg makehdb[`:hdb; 5; 100000; 10000]

makecsv[CSVDIR;NUM DAYS;NUM QUOTES;NUM TRADES]

q)makedb[10000;100000]
q)100#select from trades
time                          sym  src price size
-------------------------------------------------
2022.12.26D08:00:00.493000000 MSFT L   36.08 2624
2022.12.26D08:00:01.025000000 MSFT L   36.08 7619
2022.12.26D08:00:01.242000000 DELL N   29.09 1887
2022.12.26D08:00:02.316000000 CSCO L   35.48 1920
2022.12.26D08:00:02.404000000 DELL L   29.09 2845
2022.12.26D08:00:02.547000000 MSFT L   36.09 175
2022.12.26D08:00:02.667000000 ORCL N   32.2  3540
q)show trades1: select sym, time: time-0D00:05, priceplus5: price from trades
q)trades1
sym  time                          priceplus5
---------------------------------------------
MSFT 2022.12.26D07:55:00.493000000 36.08
MSFT 2022.12.26D07:55:01.025000000 36.08
DELL 2022.12.26D07:55:01.242000000 29.09
CSCO 2022.12.26D07:55:02.316000000 35.48
DELL 2022.12.26D07:55:02.404000000 29.09
MSFT 2022.12.26D07:55:02.547000000 36.09
q)show tab1: aj[`sym`time;trades;trades1]
time                          sym  src price size priceplus5
------------------------------------------------------------
2022.12.26D08:00:00.493000000 MSFT L   36.08 2624 36.09
2022.12.26D08:00:01.025000000 MSFT L   36.08 7619 36.06
2022.12.26D08:00:01.242000000 DELL N   29.09 1887 29.09
2022.12.26D08:00:02.316000000 CSCO L   35.48 1920 35.41
2022.12.26D08:00:02.404000000 DELL L   29.09 2845 29.09
2022.12.26D08:00:02.547000000 MSFT L   36.09 175  36.06
2022.12.26D08:00:02.667000000 ORCL N   32.2  3540 32.18
```
* now we can calculate pnl
```
q)update pnl: size*priceplus5-price from tab1
time                          sym  src price size priceplus5 pnl
--------------------------------------------------------------------
2022.12.26D08:00:00.493000000 MSFT L   36.08 2624 36.09      26.24
2022.12.26D08:00:01.025000000 MSFT L   36.08 7619 36.06      -152.38
2022.12.26D08:00:01.242000000 DELL N   29.09 1887 29.09      0
2022.12.26D08:00:02.316000000 CSCO L   35.48 1920 35.41      -134.4
2022.12.26D08:00:02.404000000 DELL L   29.09 2845 29.09      0
2022.12.26D08:00:02.547000000 MSFT L   36.09 175  36.06      -5.25
2022.12.26D08:00:02.667000000 ORCL N   32.2  3540 32.18      -70.8
2022.12.26D08:00:02.796000000 IBM  O   43.52 2054 43.55      61.62
2022.12.26D08:00:02.887000000 DELL N   29.05 1567 29.09      62.68
```
* let's generalise by creating a function which takes argument of number of seconds shifted
```
q)sel:{[x] select sym, time.second-x,price from trades }
q)sel[10]
sym  second   price
-------------------
MSFT 07:59:50 36.08
MSFT 07:59:51 36.08
DELL 07:59:51 29.09
CSCO 07:59:52 35.48
DELL 07:59:52 29.09
MSFT 07:59:52 36.09
```
* let's rename the price column using `xcol`

```
q)sel:{[x] (`sym`second,`$"pricePlus",string[x]) xcol select sym, time.second-x,price from trades }
q)sel[10]
sym  second   pricePlus10
-------------------------
MSFT 07:59:50 36.08
MSFT 07:59:51 36.08
DELL 07:59:51 29.09
CSCO 07:59:52 35.48
DELL 07:59:52 29.09
```
* but it doesn't work with negative values, let's modify it
```
q)sel:{[x] (`sym`second,`$($[x<0;"Minus";"Plus"]),string[abs[x]]) xcol select sym, time.second-x,price from trades }
q)sel[10]
sym  second   Plus10
--------------------
MSFT 07:59:50 36.08
MSFT 07:59:51 36.08
DELL 07:59:51 29.09
CSCO 07:59:52 35.48
q)sel[-10]
sym  second   Minus10
---------------------
MSFT 08:00:10 36.08
MSFT 08:00:11 36.08
DELL 08:00:11 29.09
CSCO 08:00:12 35.48
```
* now we can use it in `aj`

```
q)aj[`sym`second;select sym, time.second,price,size from trades;sel[300]]
sym  second   price size Plus300
--------------------------------
MSFT 08:00:00 36.08 2624 36.06
MSFT 08:00:01 36.08 7619 36.06
DELL 08:00:01 29.09 1887 29.09
CSCO 08:00:02 35.48 1920 35.41
```
* we can combine it with adverb over `/` and get the values at number of different times

```
q){aj[`sym`second;x;sel[y]]}/[select sym, time.second,price,size from trades;10 20]
sym  second   price size Plus10 Plus20
--------------------------------------
MSFT 08:00:00 36.08 2624 36.09  36.06
MSFT 08:00:01 36.08 7619 36.09  36.06
DELL 08:00:01 29.09 1887 29.09  29.05
CSCO 08:00:02 35.48 1920 35.52  35.48
DELL 08:00:02 29.09 2845 29.09  29.09
```
* let's write this as a function called `priceat`

```
q)priceat:{{aj[`sym`second;x;sel[y]]}/[select sym, time.second,price,size from x;y]}
q)priceat[trades;5 10 20]
sym  second   price size Plus5 Plus10 Plus20
--------------------------------------------
MSFT 08:00:00 36.08 2624 36.09 36.09  36.06
MSFT 08:00:01 36.08 7619 36.08 36.09  36.06
DELL 08:00:01 29.09 1887 29.05 29.09  29.05
CSCO 08:00:02 35.48 1920 35.52 35.52  35.48
```

* now to calculate the PnL at each price
* first we drop the static column and flip it to a dictionary

```
q)t: priceat[trades;-120 -1 0 5 60 300]
q)t
sym  second   price size Minus120 Minus1 Plus0 Plus5 Plus60 Plus300
-------------------------------------------------------------------
MSFT 08:00:00 36.08 2624                 36.08 36.09 36.06  36.06
MSFT 08:00:01 36.08 7619          36.08  36.08 36.08 36.06  36.06
DELL 08:00:01 29.09 1887                 29.09 29.05 29.05  29.09
CSCO 08:00:02 35.48 1920                 35.48 35.52 35.52  35.41
DELL 08:00:02 29.09 2845          29.09  29.05 29.05 29.05  29.09
q)flip `sym`second`price`size _ t
Minus120|                                                                    ..
Minus1  |       36.08             29.09 36.08             29.09              ..
Plus0   | 36.08 36.08 29.09 35.48 29.05 36.09 32.2  43.52 29.05 35.48 25.36 2..
Plus5   | 36.09 36.08 29.05 35.52 29.05 36.08 32.24 43.52 29.05 35.52 25.36 2..
Plus60  | 36.06 36.06 29.05 35.52 29.05 36.06 32.2  43.53 29.05 35.52 25.34 2..
Plus300 | 36.06 36.06 29.09 35.41 29.09 36.06 32.18 43.55 29.09 35.41 25.29 2..
```
* then we use each-left `\:` adverb to subtract the original trade price 
* and we multiply it with size using each-right `/:`
```
q)(flip `sym`second`price`size _ t)-\:t`price
Minus120|                                                                    ..
Minus1  |       0                 0     -0.01            0.04                ..
Plus0   | 0     0     0     0     -0.04 0     0     0    0    0     0.02  0  ..
Plus5   | 0.01  0     -0.04 0.04  -0.04 -0.01 0.04  0    0    0.04  0.02  0  ..
Plus60  | -0.02 -0.02 -0.04 0.04  -0.04 -0.03 0     0.01 0    0.04  0     -0...
Plus300 | -0.02 -0.02 0     -0.07 0     -0.03 -0.02 0.03 0.04 -0.07 -0.05 -0...
q)t[`size]*/:(flip `sym`second`price`size _ t)-\:t`price
Minus120|                                                                    ..
Minus1  |        0                     0      -1.75             62.68        ..
Plus0   | 0      0       0      0      -113.8 0     0     0     0     0      ..
Plus5   | 26.24  0       -75.48 76.8   -113.8 -1.75 141.6 0     0     89.04  ..
Plus60  | -52.48 -152.38 -75.48 76.8   -113.8 -5.25 0     20.54 0     89.04  ..
Plus300 | -52.48 -152.38 0      -134.4 0      -5.25 -70.8 61.62 62.68 -155.82..
```
* now we flip the dictionary back to table and join the static columns using join-each `,'`

```
q)(`sym`second`price`size#t),'flip t[`size]*/:(flip `sym`second`price`size _ t)-\:t`price
sym  second   price size Minus120 Minus1 Plus0  Plus5  Plus60  Plus300
----------------------------------------------------------------------
MSFT 08:00:00 36.08 2624                 0      26.24  -52.48  -52.48
MSFT 08:00:01 36.08 7619          0      0      0      -152.38 -152.38
DELL 08:00:01 29.09 1887                 0      -75.48 -75.48  0
CSCO 08:00:02 35.48 1920                 0      76.8   76.8    -134.4
DELL 08:00:02 29.09 2845          0      -113.8 -113.8 -113.8  0
```
* we can turn this into a table by replacing table `t` with variable `x`

```
q)pnl:{(`sym`second`price`size#x),'flip x[`size]*/:(flip `sym`second`price`size _ x)-\:x`price}
q)q)pnl priceat[trades;-10 5 5 10 20]
sym  second   price size Minus10 Plus5  Plus10 Plus20
------------------------------------------------------
MSFT 08:00:00 36.08 2624         26.24  26.24  -52.48
MSFT 08:00:01 36.08 7619         0      76.19  -152.38
DELL 08:00:01 29.09 1887         -75.48 0      -75.48
CSCO 08:00:02 35.48 1920         76.8   76.8   0
DELL 08:00:02 29.09 2845         -113.8 0      0
MSFT 08:00:02 36.09 175          -1.75  0      -5.25
```
* we can run further analysis on it and get performance as well
```
q)select sum Plus120, sum Plus300 by sym from pnl priceat[trades;120 300]
sym | Plus120   Plus300
----| -------------------
AAPL| 22717.91  55355.6
CSCO| -47906.97 -78350.84
DELL| -334.31   28214.05
GOOG| -104945.9 -276816.1
IBM | 47372.67  62615.09
MSFT| -39582.02 -72533.59
NOK | 8341.78   -2454.73
ORCL| 8724.23   -36906.04
YHOO| -39038.82 -108447.7
q)\ts select sum Plus120, sum Plus300 by sym from pnl priceat[trades;120 300]
60 7865472
```
# Recap
* we used 3 functions to perform this analysis
* `sel` which selects the price at a specified number of seconds before or after the current trade 
* `priceat` which uses the adverb over `/` to select price at number of different times 
* `pnl` which calcules the PnL at each time selected in the query


* we will do further analysis by calculating different statistics but that is grouped

* grouping columns would be selected dynamically so we need to use functional form of queries 

* let's see sample using `parse`
```
q)parse"select sum Plus120, sum Plus300 by sym from trades"
?
`trades
()
(,`sym)!,`sym
`Plus120`Plus300!((sum;`Plus120);(sum;`Plus300))
```
* now we first want to re-create last argument of functional form

```
q)p: pnl priceat[trades;120 300 -10 -30]
sym  second   price size Plus120 Plus300 Minus10 Minus30
--------------------------------------------------------
MSFT 08:00:00 36.08 2624 -157.44 -52.48
MSFT 08:00:01 36.08 7619 -152.38 -152.38
DELL 08:00:01 29.09 1887 -132.09 0
CSCO 08:00:02 35.48 1920 38.4    -134.4
q)c: cols p: pnl priceat[trades;120 300 -10 -30]
q)c like/:("Plus*";"Minus*")
00001100b
00000011b
q)any c like/:("Plus*";"Minus*")
00001111b
q)c where any c like/:("Plus*";"Minus*")
`Plus120`Plus300`Minus10`Minus30
q)c:c where any c like/:("Plus*";"Minus*")
q)sum,/:c:c where any c like/:("Plus*";"Minus*")
sum `Plus120
sum `Plus300
sum `Minus10
sum `Minus30
q)c!sum,/:c:c where any c like/:("Plus*";"Minus*")
Plus120| sum `Plus120
Plus300| sum `Plus300
Minus10| sum `Minus10
Minus30| sum `Minus30
```
* now forming full functional query
```
q)?[p;();g!g:`sym,();c!sum,/:c:c where any (c:cols p) like/:("Plus*";"Minus*")]

q)p: pnl priceat[trades;120 300 -10 -30]

q)?[p;();g!g:`sym,();c!sum,/:c:c where any (c:cols p) like/:("Plus*";"Minus*")]
sym | Plus120   Plus300   Minus10  Minus30
----| --------------------------------------
AAPL| 22717.91  55355.6   20165.13 13798.57
CSCO| -47906.97 -78350.84 4750.23  -19347.92
DELL| -334.31   28214.05  6217.01  -3688.24
GOOG| -104945.9 -276816.1 1487.34  19773.96
IBM | 47372.67  62615.09  17820.44 14116.69
MSFT| -39582.02 -72533.59 20459.3  15433.05
NOK | 8341.78   -2454.73  312.61   3813.57
ORCL| 8724.23   -36906.04 12894.14 7253.68
YHOO| -39038.82 -108447.7 4230.39  20577.22
```
* we can put this in a function to allow dynamic grouping
* `g` is grouping parameter and `x` is table parameter 

```
q)aggpnl: {[g;x] ?[x;();g!g;c!sum,/:c:c where any (c:cols x) like/:("Plus*";"Minus*")]}
q)aggpnl[(),`sym;pnl priceat[trades;-100 -200 120 300]]
sym | Minus100  Minus200  Plus120   Plus300
----| ---------------------------------------
AAPL| 18067.25  -7424.27  22717.91  55355.6
CSCO| 1788.14   19085.81  -47906.97 -78350.84
DELL| -8037.82  -15200.43 -334.31   28214.05
GOOG| 101814.2  162998.4  -104945.9 -276816.1
IBM | -15371.78 -41147.4  47372.67  62615.09
MSFT| 32295.39  52877.07  -39582.02 -72533.59
NOK | 6487.44   2694.26   8341.78   -2454.73
ORCL| 15190.38  20106.71  8724.23   -36906.04
YHOO| 50511.97  87474.26  -39038.82 -108447.7
q)// further optimised - spot the difference
q)aggpnl: {[g;x] ?[x;();g!g,:();c!sum,/:c:c where any (c:cols x) like/:("Plus*";"Minus*")]}
q)aggpnl[`sym;pnl priceat[trades;-100 -200 120 300]]
sym | Minus100  Minus200  Plus120   Plus300
----| ---------------------------------------
AAPL| 18067.25  -7424.27  22717.91  55355.6
CSCO| 1788.14   19085.81  -47906.97 -78350.84
DELL| -8037.82  -15200.43 -334.31   28214.05
GOOG| 101814.2  162998.4  -104945.9 -276816.1
IBM | -15371.78 -41147.4  47372.67  62615.09
MSFT| 32295.39  52877.07  -39582.02 -72533.59
NOK | 6487.44   2694.26   8341.78   -2454.73
ORCL| 15190.38  20106.71  8724.23   -36906.04
YHOO| 50511.97  87474.26  -39038.82 -108447.7
```

# RECAP
* we are able to calculate PnL for each trade and different time points 
```
q)show p: pnl priceat[trades;120 300 -10 -30]
sym  second   price size Plus120 Plus300 Minus10 Minus30
--------------------------------------------------------
MSFT 08:00:00 36.08 2624 -157.44 -52.48
MSFT 08:00:01 36.08 7619 -152.38 -152.38
DELL 08:00:01 29.09 1887 -132.09 0
CSCO 08:00:02 35.48 1920 38.4    -134.4
DELL 08:00:02 29.09 2845 -199.15 0
MSFT 08:00:02 36.09 175  -7      -5.25
```
* then we created a function to aggregate this data by different columns
```
q)aggpnl
{[g;x] ?[x;();g!g,:();c!sum,/:c:c where any (c:cols x) like/:("Plus*";"Minus*")]}
```
* Now, for example we add a new column hour to split the data in 1 hour time bucket 
```
q)update hour: 3600 xbar second from p
sym  second   price size Plus120 Plus300 Minus10 Minus30 hour
-----------------------------------------------------------------
MSFT 08:00:00 36.08 2624 -157.44 -52.48                  08:00:00
MSFT 08:00:01 36.08 7619 -152.38 -152.38                 08:00:00
DELL 08:00:01 29.09 1887 -132.09 0                       08:00:00
CSCO 08:00:02 35.48 1920 38.4    -134.4                  08:00:00
DELL 08:00:02 29.09 2845 -199.15 0                       08:00:00
MSFT 08:00:02 36.09 175  -7      -5.25                   08:00:00
ORCL 08:00:02 32.2  3540 177     -70.8                   08:00:00
IBM  08:00:02 43.52 2054 143.78  61.62                   08:00:00
```
* then we can aggregate PnL by this new column hour as well 
```
q)aggpnl[`sym`hour;update hour: 3600 xbar second from pnl priceat[trades;-100 -200 120 300]]
sym  hour    | Minus100  Minus200  Plus120   Plus300
-------------| ---------------------------------------
AAPL 08:00:00| 19270.56  31587.66  -26519.89 -41863.08
AAPL 09:00:00| -1994.83  -16828.77 1078.57   5073.86
AAPL 10:00:00| -2244.5   -11181.54 12990.95  33201.2
AAPL 11:00:00| -10400.1  -15510.17 10905.64  31212.47
AAPL 12:00:00| 19347.66  24419.62  -2687.42  -20299.06
```
* we can also bucket the trade volume and aggregate by this column 
```
q)aggpnl[`sym`size;update size : 3000 xbar size from pnl priceat[trades;-100 -200 120 300]]
sym  size| Minus100 Minus200  Plus120   Plus300
---------| --------------------------------------
AAPL 0   | -7115.16 -14194.51 3609.58   12080.66
AAPL 3000| 13486.6  8903.18   9650.91   33716.12
AAPL 6000| 13933.78 413.07    5114.52   2928.46
AAPL 9000| -2237.97 -2546.01  4342.9    6630.36
CSCO 0   | 1513.52  5812.96   -9580.98  -12700.74
```
These queries and functions are quite powerful