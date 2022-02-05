# Keywords

## Table modification Keywords

```
q)//setup
q)\l 2_where_clause_fakedb.q
USAGE: makedb[NUM QUOTES;NUM TRADES] eg makedb[100000;10000]

makedb1[NUM QUOTES;NUM TRADES;DATE;RANDOMISED COUNT FACTOR] eg makedb1[100000;10000;.z.d;.3]

makehdb[HDBDIR;NUM DAYS;APPROXIMATE NUM QUOTES PER DAY; APPROXIMATE NUM TRADES PER DAY] eg makehdb[`:hdb; 5; 100000; 10000]

makecsv[CSVDIR;NUM DAYS;NUM QUOTES;NUM TRADES]

q)makedb[10000;1000]
q)tables[]
`depth`quotes`trades
```
### `cols`

* used to view columns of the table and also to view keys of the dictionary
```
q)cols trades
`time`sym`src`price`size
```
### `xasc` / `xdesc`
* used to sort the table by columns specified
```
q)`price xasc  trades
time                          sym  src price size
-------------------------------------------------
2020.12.26D08:34:34.754000000 AAPL O   25.12 1535
2020.12.26D08:34:01.151000000 AAPL L   25.14 5290
2020.12.26D08:35:52.188000000 AAPL N   25.15 3413
2020.12.26D08:42:18.951000000 AAPL L   25.16 1902
2020.12.26D08:33:35.987000000 AAPL L   25.18 4579
..
q)`price xdesc trades
time                          sym src price size
------------------------------------------------
2020.12.26D16:17:13.172000000 IBM L   43.88 2560
2020.12.26D15:29:24.817000000 IBM L   43.84 369
2020.12.26D16:12:52.607000000 IBM N   43.83 4454
2020.12.26D16:21:07.666000000 IBM O   43.82 3918
2020.12.26D16:13:22.181000000 IBM L   43.81 4869
..
```
* more than 1 column can also be specified - it will first sort by first column, and if values are same for first then sort by next column
```
q)`price`size xdesc trades
time                          sym src price size
------------------------------------------------
2020.12.26D16:17:13.172000000 IBM L   43.88 2560
2020.12.26D15:29:24.817000000 IBM L   43.84 369
2020.12.26D16:12:52.607000000 IBM N   43.83 4454
2020.12.26D16:21:07.666000000 IBM O   43.82 3918
2020.12.26D16:13:22.181000000 IBM L   43.81 4869
```
### `xcol`
* used to rename columns of table
* it will only change column name from **left to right**
* it will rename with first column as first name provided by symbol list on left and so on..
```
q)`Time`Sym xcol trades
Time                          Sym  src price size
-------------------------------------------------
2020.12.26D08:00:17.350000000 CSCO N   35.46 2367
2020.12.26D08:00:30.245000000 DELL N   29.05 27
2020.12.26D08:01:40.842000000 AAPL L   25.37 2797
2020.12.26D08:02:11.579000000 CSCO L   35.46 1714
2020.12.26D08:02:12.246000000 YHOO N   35.5  166
..
```
### `xcols`
* reorder columns of the table
* it doesn't have to complete list
* it moves the specified columns to the left of the table
* column names need to be valid column names
```
q)`price`size xcols trades
price size time                          sym  src
-------------------------------------------------
35.46 2367 2020.12.26D08:00:17.350000000 CSCO N
29.05 27   2020.12.26D08:00:30.245000000 DELL N
25.37 2797 2020.12.26D08:01:40.842000000 AAPL L
35.46 1714 2020.12.26D08:02:11.579000000 CSCO L
35.5  166  2020.12.26D08:02:12.246000000 YHOO N
..
q)`price`size`polo xcols trades
'polo
  [0]  `price`size`polo xcols trades
                        ^
q))\
```
### `xkey`
* used to make specified columns as key of table
* can be multiple columns as well
* to key table in-place provid table name as symbol
```
q)`price`src xkey trades
price src| time                          sym  size
---------| ---------------------------------------
35.46 N  | 2020.12.26D08:00:17.350000000 CSCO 2367
29.05 N  | 2020.12.26D08:00:30.245000000 DELL 27
25.37 L  | 2020.12.26D08:01:40.842000000 AAPL 2797
35.46 L  | 2020.12.26D08:02:11.579000000 CSCO 1714
35.5  N  | 2020.12.26D08:02:12.246000000 YHOO 166
..
q)`price`src xkey `trades
`trades
```
### `keys`
* can be used to see keys of table
```
q)keys trades
`price`src
q)keys quotes
`symbol$()
```
### `key`
* to view full keyed columns of the table
```
q)key trades
price src
---------
35.46 N
29.05 N
25.37 L
35.46 L
35.5  N
..
```
* can also be used to get list of keys of the dictionary
```
q)d:`ab`bv!1 2
q)d
ab| 1
bv| 2
q)key d
`ab`bv
```
### `xgroup`
* used to group by column/s of a table
```
q)`src xgroup quotes
src| time                                                                    ..
---| ------------------------------------------------------------------------..
L  | 2020.12.26D08:00:02.809000000 2020.12.26D08:00:10.646000000 2020.12.26D0..
O  | 2020.12.26D08:00:09.282000000 2020.12.26D08:00:20.104000000 2020.12.26D0..
N  | 2020.12.26D08:00:10.075000000 2020.12.26D08:00:11.107000000 2020.12.26D0..
```
* `ungroup` can be used to flatten grouped table
## Statistical Keywords
### `first`, `last`, `max`, `min`, `sum`, `avg`
* used to find first, last, maximum, minimum, total and average of list/columns of table
```
q)select max price, min price, first price, last price, sum price, avg price from trades
price price1 price2 price3 price4   price5
--------------------------------------------
43.88 25.12  35.46  29.16  34679.26 34.67926
```
### `sums`, `maxs`, `mins`, `avgs`
* used to specify running total, running maximum, running minimum and running average from list/column of table
```
q)select sums price, maxs price, mins price from trades
price  price1 price2
--------------------
35.46  35.46  35.46
64.51  35.46  29.05
89.88  35.46  25.37
125.34 35.46  25.37
160.84 35.5   25.37
..
```
### `msum`, `mmax`, `mmin`, `mavg`
* used for moving total, moving maximum, moving minimum, moving average from list/column of table
* moving windows size is defined as left argument
```
q)select 2 msum price, 2 mmax price, 2 mmin price, 2 mavg price from trades
price price1 price2 price3
--------------------------
35.46 35.46  35.46  35.46
64.51 35.46  29.05  32.255
54.42 29.05  25.37  27.21
60.83 35.46  25.37  30.415
70.96 35.5   35.46  35.48
..
```
## Other useful keywords
```
q)//setup
q)t:0!select from trades where sym=`GOOG
q)t
price src time                          sym  size
-------------------------------------------------
41.29 L   2020.12.26D08:10:37.457000000 GOOG 1902
41.33 N   2020.12.26D08:11:24.604000000 GOOG 3150
41.31 L   2020.12.26D08:17:49.148000000 GOOG 2133
41.31 L   2020.12.26D08:18:04.621000000 GOOG 2623
41.28 N   2020.12.26D08:29:30.642000000 GOOG 825
..
```
### `differ`
* returns true `1b` if every element is different from previous element of the list/column of table
* always returns `1b` for first element
```
q)differ t`price
11101011111110111111111011111110111110111111111111011011111101111111111111011..
```
* can be used to find where price did changed
```
q)select from t where differ price
price src time                          sym  size
-------------------------------------------------
41.29 L   2020.12.26D08:10:37.457000000 GOOG 1902
41.33 N   2020.12.26D08:11:24.604000000 GOOG 3150
41.31 L   2020.12.26D08:17:49.148000000 GOOG 2133
41.28 N   2020.12.26D08:29:30.642000000 GOOG 825
41.24 N   2020.12.26D08:42:37.510000000 GOOG 4959
..
```

### `deltas`
* shows different between current and previous values of the list
* can be used to select trades where price has increased
```
q)select from t where (deltas price)>0
price src time                          sym  size
-------------------------------------------------
41.29 L   2020.12.26D08:10:37.457000000 GOOG 1902
41.33 N   2020.12.26D08:11:24.604000000 GOOG 3150
41.29 L   2020.12.26D08:47:15.737000000 GOOG 7592
41.32 O   2020.12.26D08:49:33.238000000 GOOG 3863
41.21 L   2020.12.26D09:08:34.960000000 GOOG 1534
..
```
### `next`
* it moves the elements of the list 1 space to the left and filling the empty space with null
### `prev`
* it moves the elements of the list 1 space to the right and filling the empty space with null
* can be used to find duration between trades
```
q)update duration:time-prev time from t
price src time                          sym  size duration
----------------------------------------------------------------------
41.29 L   2020.12.26D08:10:37.457000000 GOOG 1902
41.33 N   2020.12.26D08:11:24.604000000 GOOG 3150 0D00:00:47.147000000
41.31 L   2020.12.26D08:17:49.148000000 GOOG 2133 0D00:06:24.544000000
41.31 L   2020.12.26D08:18:04.621000000 GOOG 2623 0D00:00:15.473000000
41.28 N   2020.12.26D08:29:30.642000000 GOOG 825  0D00:11:26.021000000
..
```
### `xprev`
* shifts the elements of the list specified numbers to the right
* `xnext` keyword doesn't exist we can replicate the effect by specifying `-n` as argument to `xprev`
```
q)t`time
2020.12.26D08:10:37.457000000 2020.12.26D08:11:24.604000000 2020.12.26D08:17:..
q)2 xprev t`time
0N 0N 2020.12.26D08:10:37.457000000 2020.12.26D08:11:24.604000000 2020.12.26D..
q)-2 xprev t`time // works as xnext
2020.12.26D08:17:49.148000000 2020.12.26D08:18:04.621000000 2020.12.26D08:29:..
```
### `^` - `coalease` - `fill nulls`
* used to fill `0N` nulls in the list with specified atom or list items
* it type promote the element if filling element is of type greater than existing list
```
q)3 ^ 1 2 3 0N 0N
1 2 3 3 3
q)3f ^ 1 2 3 0N 0N
1 2 3 3 3f
q)type 1 2 3 0N 0N
7h
q)type 3f ^ 1 2 3 0N 0N
9h
q)4 5 6 7 8f ^ 1 2 3 0N 0N   // also works if list of same length is provided as filling argument
1 2 3 7 8f
```
### `fills`
* it fills the nulls in the list with first non null element in the left
```
q)fills  1 2 3 0N 0N 9 0N 8 0N 0N
1 2 3 3 3 9 9 8 8 8
```
### `iasc/idesc`
* it returns index of the elements to sort the list in ascending or descending order
```
q)l: 90 8928 4345 258972 2349827
q)idesc l
4 3 1 2 0
q)iasc l
0 2 1 3 4
```
* it can be used to order a list w.r.t another list  of same size
```
q)m:(1 2 4 3; 6 4 5; 2 6 3 7 5; 2 5 )
q)m
1 2 4 3
6 4 5
2 6 3 7 5
2 5
q)count each m
4 3 5 2
q)m iasc count each m
2 5
6 4 5
1 2 4 3
2 6 3 7 5
q)iasc count each m
3 1 0 2
```
### `rank`
* returns the positions would appear if it were sorted in sorted order
* equivalent to using `iasc` twice on a list
```
q)l
90 8928 4345 258972 2349827
q)rank l
0 2 1 3 4
q)iasc iasc l
0 2 1 3 4
q)iasc l
0 2 1 3 4
```
* can be used to rank the prices of trades
```
q)update prcrank:rank price from t
price src time                          sym  size prcrank
---------------------------------------------------------
41.29 L   2020.12.26D08:10:37.457000000 GOOG 1902 111
41.33 N   2020.12.26D08:11:24.604000000 GOOG 3150 124
41.31 L   2020.12.26D08:17:49.148000000 GOOG 2133 116
41.31 L   2020.12.26D08:18:04.621000000 GOOG 2623 117
41.28 N   2020.12.26D08:29:30.642000000 GOOG 825  109
..
```
### `xrank`
* used to bucket the list into specified number of bucket
* and result values specify which bucket each element of the list will fit into
```
q)l
90 8928 4345 258972 2349827
q)2 xrank l
0 0 0 1 1
q)1 xrank l
0 0 0 0 0
q)3 xrank l
0 1 0 1 2
q)4 xrank l
0 1 0 2 3
q)
```
