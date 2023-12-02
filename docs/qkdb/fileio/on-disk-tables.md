# On Disk Tables
We can save a table to disk in 3 different formats 
 1. flat table
 2. splayed table
 3. partitioned table

## Flat Table
* Saved to disk as-is, with all the information in one file
* These can only be used by loading them into memory
* Used only by small, frequently used databases
* Generally not suitable for large dataset
### Save flat table
* Lets create a table in-memory 
```q
q)price:([]fruit: `banana`grapes`orange; qty: 10 15 13; prc: 10.78 40.88 30.87)
q)price
fruit  qty prc
----------------
banana 10  10.78
grapes 15  40.88
orange 13  30.87
```
* we can save the table to disk using `handle_to_file set table_name` 
* here `handle_to_file` can be relative if saving in current directory or full path if saving in another directory provided write permissions are valid
```q 
q)`:pricetab set price
`:pricetab
```
* or we can use `save` in case we want to keep filename same as tablename 
```q
q)save `:price
`:price
```
* after saving to disk we can see there are two flat files in the working directory
```
q)system"ls -lh"
"total 16"
"-rw-r--r--  1 ranayk  staff   118B Oct  4 22:26 price"
"-rw-r--r--  1 ranayk  staff   118B Oct  4 22:25 pricetab"
```
### Load flat table
* to read these flat files we can use function `get` or `load`
```q
q)get `:price
fruit  qty prc
----------------
banana 10  10.78
grapes 15  40.88
orange 13  30.87
q)load `:pricetab
`pricetab
q)pricetab
fruit  qty prc
----------------
banana 10  10.78
grapes 15  40.88
orange 13  30.87
```
## Splayed table
* In splayed table a directory is created with name same as table name and having files for each columns and also a `.d` file which contains the order of the columns 
* to splay we add the `/` to the end of file handle to specify it is a directory - it is same for both unix and windows system i.e; `/` forward slash
### Save splayed tables
* let's create a new table called `prices2` which we can save as splayed table
```q
q)prices2:([]cars:"ABC"; price: 500 1100 1500 ; qty: 1000 350 20)
q)prices2
cars price qty
---------------
A    500   1000
B    1100  350
C    1500  20
```
* we can save splayed table using `set` but adding `/` at the end of file handle
```q
q)`:splayedprice2/ set prices2
`:splayedprice2/
q)system"ls -lh"
"total 16"
"-rw-r--r--  1 ranayk  staff   118B Oct  4 22:26 price"
"-rw-r--r--  1 ranayk  staff   118B Oct  4 22:25 pricetab"
"drwxr-xr-x  6 ranayk  staff   192B Oct  4 22:43 splayedprice2"
q)system"ls -alh splayedprice2"
"total 32"
"drwxr-xr-x  6 ranayk  staff   192B Oct  4 22:43 ."
"drwxr-xr-x  5 ranayk  staff   160B Oct  4 22:43 .."
"-rw-r--r--  1 ranayk  staff    23B Oct  4 22:43 .d"
"-rw-r--r--  1 ranayk  staff    19B Oct  4 22:43 cars"
"-rw-r--r--  1 ranayk  staff    40B Oct  4 22:43 price"
"-rw-r--r--  1 ranayk  staff    40B Oct  4 22:43 qty"

```
### Load splayed tables
* when we load the splayed table into the memory it is mapped to the memory not loaded into the memory - we can see this by checking `mmap` value from result of `.Q.w[]` function
* before loading the table `.Q.w[]` shows `0` for `mmap`
```qq).Q.w[]
used| 357056
heap| 67108864
peak| 67108864
wmax| 0
mmap| 0
mphy| 8589934592
syms| 663
symw| 28351
```
* we can load the splayed table using `get`
* after loading the table into memory we can re-examine the `mmap` value from `.Q.w[]` output
```q
q)prices2
cars price qty
---------------
A    500   1000
B    1100  350
C    1500  20
q).Q.w[]
used| 358336
heap| 67108864
peak| 67108864
wmax| 0
mmap| 99
mphy| 8589934592
syms| 669
symw| 28547
```
### Edit splayed table on disk
* we can re-order the table's columns by editing `.d` file
```q
q)get `:splayedprice2/.d
`cars`price`qty
q)`:splayedprice2/.d set `qty`price`cars
`:splayedprice2/.d
q)load `:splayedprice2
`splayedprice2
q)splayedprice2
qty  price cars
---------------
1000 500   A
350  1100  B
20   1500  C
```
* we can add the columns to splayed table on disk 
* and we need to edit the `.d` file as well
```q
q)`:splayedprice2/date set 2022.10.01 2022.10.02 2022.10.03
`:splayedprice2/date
q)`:splayedprice2/.d set (get `:splayedprice2/.d),`date
`:splayedprice2/.d
q)load `:splayedprice2
`splayedprice2
q)splayedprice2
qty  price cars date
--------------------------
1000 500   A    2022.10.01
350  1100  B    2022.10.02
20   1500  C    2022.10.03
```
* similarly we can delete the column from splayed table on disk using `hdel` and here as well we need to edit `.d` file
```q
q)hdel `:splayedprice2/date
`:splayedprice2/date
q)(get `:splayedprice2/.d) except `date
`qty`price`cars
q)`:splayedprice2/.d set (get `:splayedprice2/.d) except `date
`:splayedprice2/.d
q)load `:splayedprice2
`splayedprice2
q)splayedprice2
qty  price cars
---------------
1000 500   A
350  1100  B
20   1500  C
```
* we can also add a new row to splayed table on disk using `upsert` function
```q
q)`:splayedprice2/ upsert (1500;70;"D")
`:splayedprice2/
q)get `:splayedprice2/
qty  price cars
---------------
1000 500   A
350  1100  B
20   1500  C
1500 70    D
```
* we can also sort the table - using `xasc`
```q
q)`qty xasc `:splayedprice2/
`:splayedprice2/
q)get `:splayedprice2/
qty  price cars
---------------
20   1500  C
350  1100  B
1000 500   A
1500 70    D
```
## Partitioned tables
* Partitioned tables can be split horizontally as well as vertically - meaning data is split by row value(date or month) then each column is splayed into single file on disk - similar to splayed table
### Save partitioned table
* Lets define 2 tables for each parition and save them to disk using `set`
* here we are providing directory, parition and table name in file handle
```q
q)price1:([]car:"ABC";qty:10 20 30; price: 33 44 55)
q)price1
car qty price
-------------
A   10  33
B   20  44
C   30  55
q)price2:([]car:"ZXY";qty:99 88 77; price: 10 20 30)
q)price2
car qty price
-------------
Z   99  10
X   88  20
Y   77  30
```
* on disk structure
```
q)system"ls -lah"
"total 16"
"drwxr-xr-x   6 ranayk  staff   192B Oct  4 23:25 ."
"drwxr-xr-x+ 54 ranayk  staff   1.7K Oct  4 23:22 .."
"drwxr-xr-x   4 ranayk  staff   128B Oct  4 23:25 partprice"
"-rw-r--r--   1 ranayk  staff   118B Oct  4 22:26 price"
"-rw-r--r--   1 ranayk  staff   118B Oct  4 22:25 pricetab"
"drwxr-xr-x   6 ranayk  staff   192B Oct  4 23:04 splayedprice2"
q)system"ls -lah partprice"
"total 0"
"drwxr-xr-x  4 ranayk  staff   128B Oct  4 23:25 ."
"drwxr-xr-x  6 ranayk  staff   192B Oct  4 23:25 .."
"drwxr-xr-x  3 ranayk  staff    96B Oct  4 23:25 2022.10.01"
"drwxr-xr-x  3 ranayk  staff    96B Oct  4 23:25 2022.10.02"
q)system"ls -lah partprice/2022.10.01/"
"total 0"
"drwxr-xr-x  3 ranayk  staff    96B Oct  4 23:25 ."
"drwxr-xr-x  4 ranayk  staff   128B Oct  4 23:25 .."
"drwxr-xr-x  6 ranayk  staff   192B Oct  4 23:25 price"
q)system"ls -lah partprice/2022.10.01/price/" //if we had more tables in this partition it would show as another directory with directory name same as table's name
"total 32"
"drwxr-xr-x  6 ranayk  staff   192B Oct  4 23:25 ."
"drwxr-xr-x  3 ranayk  staff    96B Oct  4 23:25 .."
"-rw-r--r--  1 ranayk  staff    22B Oct  4 23:25 .d"
"-rw-r--r--  1 ranayk  staff    19B Oct  4 23:25 car"
"-rw-r--r--  1 ranayk  staff    40B Oct  4 23:25 price"
"-rw-r--r--  1 ranayk  staff    40B Oct  4 23:25 qty"
q)system"ls -lah partprice/2022.10.02/"
"total 0"
"drwxr-xr-x  3 ranayk  staff    96B Oct  4 23:25 ."
"drwxr-xr-x  4 ranayk  staff   128B Oct  4 23:25 .."
"drwxr-xr-x  6 ranayk  staff   192B Oct  4 23:25 price"
q)system"ls -lah partprice/2022.10.02/price/"
"total 32"
"drwxr-xr-x  6 ranayk  staff   192B Oct  4 23:25 ."
"drwxr-xr-x  3 ranayk  staff    96B Oct  4 23:25 .."
"-rw-r--r--  1 ranayk  staff    22B Oct  4 23:25 .d"
"-rw-r--r--  1 ranayk  staff    19B Oct  4 23:25 car"
"-rw-r--r--  1 ranayk  staff    40B Oct  4 23:25 price"
"-rw-r--r--  1 ranayk  staff    40B Oct  4 23:25 qty"
```
### Load the partitioned table
* we can use `\l` with directory name to load the partitioned table
```q
q)\l partprice
q)price
date       car qty price
------------------------
2022.10.01 A   10  33
2022.10.01 B   20  44
2022.10.01 C   30  55
2022.10.02 Z   99  10
2022.10.02 X   88  20
2022.10.02 Y   77  30
```
### `.Q.ind`
* we can use `.Q.ind` to access rows of table by providing indexes as argument
```q
q).Q.ind[price;2 5]
date       car qty price
------------------------
2022.10.01 C   30  55
2022.10.02 Y   77  30
```
### `.Q.chk`
* if we have partitions with table directories missing then we can use `.Q.chk` to populate them using the schema of last parition
```q
 q)\ls
"partprice"
"price"
"pricetab"
"splayedprice2"
q)system"mkdir partprice/2022.10.03"
q)system"ls -lh partprice/2022.10.03" //latest partition doesn't have tables data populated
"total 0"
q).Q.chk[`:partprice]
,`:partprice/2022.10.03
()
()
q)system"ls -lh partprice/2022.10.03" //now it is populated
"total 0"
"drwxr-xr-x  6 ranayk  staff   192B Oct  4 23:39 price"
q)\l partprice
q)price
date       car qty price
------------------------
2022.10.01 A   10  33
2022.10.01 B   20  44
2022.10.01 C   30  55
2022.10.02 Z   99  10
2022.10.02 X   88  20
2022.10.02 Y   77  30
```
* directory structure till now
* partprice is partitioned table
* price and pricetab are flat tables, and
* splayedprice2 is splayed table
```
├── partprice
│   ├── 2022.10.01
│   │   └── price
│   │       ├── car
│   │       ├── price
│   │       └── qty
│   ├── 2022.10.02
│   │   └── price
│   │       ├── car
│   │       ├── price
│   │       └── qty
│   └── 2022.10.03
│       └── price
│           ├── car
│           ├── price
│           └── qty
├── price
├── pricetab
└── splayedprice2
    ├── cars
    ├── price
    └── qty
```
* Now if we try to save splayed or partitioned table with symbol column it will fail as it needs enumeration
## Enumeration
* to enumerate means - unique symbols are identified and index assigned to each symbol - this map is stored in `sym` file in the root of database directory
* We use `.Q.en` to enumerate the table and generate/upsert values to `sym` file when saving down splayed or partitioned table
```q
q)price
fruit  qty prc
----------------
banana 10  10.78
grapes 15  40.88
orange 13  30.87
q)`:ensplayed/price/ set .Q.en[`:ensplayed;price]
`:ensplayed/price/
```
* now we can see additional `sym` file created for price splayed table under ensplayed directory
```q
q)system"ls -lh"
"total 16"
"drwxr-xr-x  4 ranayk  staff   128B Oct  4 23:52 ensplayed"
"drwxr-xr-x  5 ranayk  staff   160B Oct  4 23:38 partprice"
"-rw-r--r--  1 ranayk  staff   118B Oct  4 22:26 price"
"-rw-r--r--  1 ranayk  staff   118B Oct  4 22:25 pricetab"
"drwxr-xr-x  6 ranayk  staff   192B Oct  4 23:04 splayedprice2"
q)system"ls -lh ensplayed"
"total 8"
"drwxr-xr-x  6 ranayk  staff   192B Oct  4 23:52 price"
"-rw-r--r--  1 ranayk  staff    29B Oct  4 23:52 sym"
```
* we can also load it into memory
```q
q)\l ensplayed/price
`price
q)price
fruit  qty prc
----------------
banana 10  10.78
grapes 15  40.88
orange 13  30.87
```
## `.Q.dpft`
* we can use `.Q.dpft` to save partitioned table on disk 
* it needs table to be defined in-memory
* `.Q.dpft` - directory, partition, column to apply part attribute to, (in-memory)tablename
```q
q)price
fruit  qty prc
----------------
banana 10  10.78
grapes 15  40.88
orange 13  30.87
q)meta price
c    | t f a
-----| -----
fruit| s
qty  | j
prc  | f
q).Q.dpft[`:hdb;2022.10.01;`fruit;`price]
`price
```
* directory structure
```q
hdb
└── 2022.10.01
    └── price
        ├── fruit
        ├── prc
        └── qty
```