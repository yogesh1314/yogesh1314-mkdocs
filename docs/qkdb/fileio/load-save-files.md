# Load Save Files

## Writing to csv file 
* Given an in-memory table we can write it to disk as `.csv` file
* `save` can be used if we want filename be same as tablename in memory 
```q
q)tab:([]sym:`a`b`c`d; prc: 1 2 3 4;cmt: "ABCD"; dt: 2022.10.01 2022.10.02 2022.10.03 2022.10.04)
q)tab
sym prc cmt dt
----------------------
a   1   A   2022.10.01
b   2   B   2022.10.02
c   3   C   2022.10.03
d   4   D   2022.10.04
q)tab2:([]col1: `q`w`e; col2: 10 20 30)
q)tab2
col1 col2
---------
q    10
w    20
e    30
q)save `:tab.csv
`:tab.csv
q)save `:tab2.csv
`:tab2.csv
```
## Loading csv
* command to load file is `0:` 
* right hand argument is handle to csv file
* left hand argument is list with first item listing type of columns in string type format and second argument is deliminator for csv it is `","` - enlisting deliminator means we have column headers in our file and they will be column name when read as table
```q
q)tab:("SICD";enlist",")0: `:tab.csv
q)tab
sym prc cmt dt
----------------------
a   1   A   2022.10.01
b   2   B   2022.10.02
c   3   C   2022.10.03
d   4   D   2022.10.04
q)tab2:("SI";enlist",")0: `:tab2.csv
q)tab2
col1 col2
---------
q    10
w    20
e    30
```
* we can leave out a column if we dont want to read it into memory by keeping type as blank
```q
q)("SICD";enlist ",")0: `:tab.csv
sym prc cmt dt
----------------------
a   1   A   2022.10.01
b   2   B   2022.10.02
c   3   C   2022.10.03
d   4   D   2022.10.04
q)("SIC ";enlist ",")0: `:tab.csv //leave out last column
sym prc cmt
-----------
a   1   A
b   2   B
c   3   C
d   4   D
q)("S CD";enlist ",")0: `:tab.csv //leave out one of middle column
sym cmt dt
------------------
a   A   2022.10.01
b   B   2022.10.02
c   C   2022.10.03
d   D   2022.10.04
```
* we can also read data without headers
```q
q)("SICD";",")0: `:tab.csv
sym a          b          c          d
    1          2          3          4
    A          B          C          D
    2022.10.01 2022.10.02 2022.10.03 2022.10.04
q)flip `col1`col2`col3`col4!1_'("SICD";",")0: `:tab.csv
col1 col2 col3 col4
-------------------------
a    1    A    2022.10.01
b    2    B    2022.10.02
c    3    C    2022.10.03
d    4    D    2022.10.04
```
## `read0`
* we can read file in string format using `read0`
* can be used with csv or txt files as well
```q
q)read0 `:tab.csv
"sym,prc,cmt,dt"
"a,1,A,2022-10-01"
"b,2,B,2022-10-02"
"c,3,C,2022-10-03"
"d,4,D,2022-10-04"
q)read0 `:tab.txt
" 1 a 3 b 4"
" 2 b 5 c 6"
" 3 d 4 e 5"
```
## `read1`
* we can also read file as binary
```q
q)read1 `:tab.txt
0x203120612033206220340a203220622035206320360a203320642034206520350a
```