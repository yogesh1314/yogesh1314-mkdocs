# Using Select Statements

## Select

* Select is used to extract a subset of data from a table
* general form `select [column/s] [by columns] from table_name where [condition/s]`
* select statement is executed in order:
  1. from table_name
  2. where [condition/s]
  3. [by columns]
  4. select [columns]

* example table:
```
q)sales:([]fruit:10?`apple`banana`orange;grocer:10?`ram`krishna`laxman;price:10?10;qty:10?100)    / sales table is defined
q)sales
fruit  grocer  price qty
------------------------
orange krishna 6     12
apple  laxman  6     10
apple  krishna 1     1
orange laxman  8     90
apple  krishna 5     73
banana ram     4     90
apple  ram     9     43
apple  krishna 2     90
apple  laxman  7     84
orange krishna 0     63
```
1. `select from table_name`
* in select if no columns are queried - it will select all columns from the table
* in its basic format - `from` is only required condition, all other(by, where, select) are optional
```
q)select from sales
fruit  grocer  price qty
------------------------
orange krishna 6     12
apple  laxman  6     10
apple  krishna 1     1
orange laxman  8     90
apple  krishna 5     73
banana ram     4     90
apple  ram     9     43
apple  krishna 2     90
apple  laxman  7     84
orange krishna 0     63
```
2. `select column/s from table_name`
* select columns separated by `,`
* even if single column is asked return type is table
```
q)select fruit, price from sales
fruit  price
------------
orange 6
apple  6
apple  1
orange 8
apple  5
banana 4
apple  9
apple  2
apple  7
orange 0
q)select fruit from sales
fruit
------
orange
apple
apple
orange
apple
banana
apple
apple
apple
orange
q)type select fruit from sales
98h
```
3. `select column/s from table_name where condition/s`
```
q)select fruit, price from sales where fruit=`apple
fruit price
-----------
apple 6
apple 1
apple 5
apple 9
apple 2
apple 7       
```
4. `select column/s by column/s from table_name`
* grouping data by column/s
```
q)select fruit, price by grocer from sales
grocer | fruit                            price
-------| ------------------------------------------
krishna| `orange`apple`apple`apple`orange 6 1 5 2 0
laxman | `apple`orange`apple              6 8 7
ram    | `banana`apple                    4 9
```
* we can apply aggregation to grouped data - `select aggregate_function column/s by column/s from table_name`
```
q)select sum price by grocer from sales
grocer | price
-------| -----
krishna| 14
laxman | 21
ram    | 13
q)select notional: sum price*qty by grocer from sales
grocer | notional
-------| --------
krishna| 618
laxman | 1368
ram    | 747
```
* if we don't query any select columns then all the columns are grouped by `by` column/s and `last` aggregate_function is applied by default
```
q)select by grocer from sales
grocer | fruit  price qty
-------| ----------------
krishna| orange 0     63
laxman | apple  7     84
ram    | apple  9     43
```

5. `select[+/- n] from table_name`
* returns n rows from start
```
q)select[3] from sales
fruit  grocer  price qty
------------------------
orange krishna 6     12
apple  laxman  6     10
apple  krishna 1     1
```
* if n < 0; returns n rows from end of table
```
q)select[-3] from sales
fruit  grocer  price qty
------------------------
apple  krishna 2     90
apple  laxman  7     84
orange krishna 0     63
```
* same can be applied to result set after the select is performed `n sublist select from table_name`
```
q)3 sublist select from sales
fruit  grocer  price qty
------------------------
orange krishna 6     12
apple  laxman  6     10
apple  krishna 1     1
```

6. `select[m n] from table_name`
* returns `n` rows starting from index `m`
```
q)select[3 4] from sales
fruit  grocer  price qty
------------------------
orange laxman  8     90
apple  krishna 5     73
banana ram     4     90
apple  ram     9     43
```
* same can be applied to result set after select is performed `m n sublist select from table_name`
```
q)3 4 sublist select from sales
fruit  grocer  price qty
------------------------
orange laxman  8     90
apple  krishna 5     73
banana ram     4     90
apple  ram     9     43
```

7. `select[</> column/s] from table_name`
* `<` means ascending order
* `>` means descending order
```
q)select[>grocer] from sales
fruit  grocer  price qty
------------------------
banana ram     4     90
apple  ram     9     43
apple  laxman  6     10
orange laxman  8     90
apple  laxman  7     84
orange krishna 6     12
apple  krishna 1     1
apple  krishna 5     73
apple  krishna 2     90
orange krishna 0     63
```
* same can be done using functions `xasc` and `xdesc` to sort the result table into ascending or descending order respectively - `column/s xasc/xdesc select from table_name`
```
q)`grocer xasc select from sales
fruit  grocer  price qty
------------------------
orange krishna 6     12
apple  krishna 1     1
apple  krishna 5     73
apple  krishna 2     90
orange krishna 0     63
apple  laxman  6     10
orange laxman  8     90
apple  laxman  7     84
banana ram     4     90
apple  ram     9     43
q)`grocer xdesc select from sales
fruit  grocer  price qty
------------------------
banana ram     4     90
apple  ram     9     43
apple  laxman  6     10
orange laxman  8     90
apple  laxman  7     84
orange krishna 6     12
apple  krishna 1     1
apple  krishna 5     73
apple  krishna 2     90
orange krishna 0     63
```
* all above points can also be combined
```
q)select[3;<price] fruit, price from sales where fruit in `apple
fruit price
-----------
apple 1
apple 2
apple 5
```

## Exec
* general form : `exec column/s by column/s from table_name where condition/s`
* can return data with type different than table, whereas select always returns data with type=table(98h)
* exec can only be applied to in-memory tables

1. `exec column/s from table_name`
```
q)exec fruit from sales
`orange`apple`apple`orange`apple`banana`apple`apple`apple`orange
q)type exec fruit from sales
11h
q)exec fruit,price from sales
fruit| orange apple apple orange apple banana apple apple apple orange
price| 6      6     1     8      5     4      9     2     7     0
q)type exec fruit,price from sales
99h
```
2. `exec column/s!column/s from table_name`
* return type is dictionary
* keys are column/s before `!` and values are ones after it
```
q)exec fruit!price from sales
orange| 6
apple | 6
apple | 1
orange| 8
apple | 5
banana| 4
apple | 9
apple | 2
apple | 7
orange| 0
```
3. `exec column/s!column/s by column/s from table_name`
```
q)exec fruit!price by grocer from sales
krishna| `orange`apple`apple`apple`orange!6 1 5 2 0
laxman | `apple`orange`apple!6 8 7
ram    | `banana`apple!4 9
```

## Update
* update allows us to add/modify columns from table
* general form: `update column_name/s: aggr column_name/s by column/s from table_name where condition/s`
1. `update new_column_name:aggr[column1;column2] from table_name`
```
q)update notional:price*qty from sales
fruit  grocer  price qty notional
---------------------------------
orange krishna 6     12  72
apple  laxman  6     10  60
apple  krishna 1     1   1
orange laxman  8     90  720
apple  krishna 5     73  365
banana ram     4     90  360
apple  ram     9     43  387
apple  krishna 2     90  180
apple  laxman  7     84  588
orange krishna 0     63  0
```
* update by default won't make changes to table, we can make changes into table by referring to table_name in update query as symbol - `update x:y from ``table_name`
```
q)update notional:price*qty from sales
fruit  grocer  price qty notional
---------------------------------
orange krishna 6     12  72
apple  laxman  6     10  60
apple  krishna 1     1   1
orange laxman  8     90  720
apple  krishna 5     73  365
banana ram     4     90  360
apple  ram     9     43  387
apple  krishna 2     90  180
apple  laxman  7     84  588
orange krishna 0     63  0
q)sales
fruit  grocer  price qty
------------------------
orange krishna 6     12
apple  laxman  6     10
apple  krishna 1     1
orange laxman  8     90
apple  krishna 5     73
banana ram     4     90
apple  ram     9     43
apple  krishna 2     90
apple  laxman  7     84
orange krishna 0     63
q)update notional:price*qty from `sales
`sales
q)sales
fruit  grocer  price qty notional
---------------------------------
orange krishna 6     12  72
apple  laxman  6     10  60
apple  krishna 1     1   1
orange laxman  8     90  720
apple  krishna 5     73  365
banana ram     4     90  360
apple  ram     9     43  387
apple  krishna 2     90  180
apple  laxman  7     84  588
orange krishna 0     63  0
```

## Delete
* used to delete columns from table
* general form is same as update/select
1. `delete column/s from table_name`
```
q)delete price from sales
fruit  grocer  qty notional
---------------------------
orange krishna 12  72
apple  laxman  10  60
apple  krishna 1   1
orange laxman  90  720
apple  krishna 73  365
banana ram     90  360
apple  ram     43  387
apple  krishna 90  180
apple  laxman  84  588
orange krishna 63  0
```
2. `delete from table_name where condition/s`
```
q)delete from sales where fruit=`apple
fruit  grocer  qty notional
---------------------------
orange krishna 12  72
orange laxman  90  720
banana ram     90  360
orange krishna 63  0
```
* we need to provide table_name as symbol to make changes permanent

```
q)delete notional from sales
fruit  grocer  qty
------------------
orange krishna 12
apple  laxman  10
apple  krishna 1
orange laxman  90
apple  krishna 73
banana ram     90
apple  ram     43
apple  krishna 90
apple  laxman  84
orange krishna 63
q)sales
fruit  grocer  qty notional
---------------------------
orange krishna 12  72
apple  laxman  10  60
apple  krishna 1   1
orange laxman  90  720
apple  krishna 73  365
banana ram     90  360
apple  ram     43  387
apple  krishna 90  180
apple  laxman  84  588
orange krishna 63  0
q)delete notional from `sales
`sales
q)sales
fruit  grocer  qty
------------------
orange krishna 12
apple  laxman  10
apple  krishna 1
orange laxman  90
apple  krishna 73
banana ram     90
apple  ram     43
apple  krishna 90
apple  laxman  84
orange krishna 63
```
* we can also use `delete` to remove entities from our workspace
```
q)tables[]
,`sales
q)delete sales from `.
`.
q)tables[]
`symbol$()
q)sales
'sales
  [0]  sales
       ^
q)
```
* we can also delete everything from current workspace using `delete`
```
q)delete from `.
`.
```

## Insert
* general form `table_name_as_symbol insert (values;as;column;types)`
* returned is the index number where rows are added into the table
```
q)cars:([]brand:`symbol$();model:`symbol$();purchasedate:`date$())
q)cars
brand model purchasedate
------------------------
q)`cars insert (`bmw;`z4;2020.01.01)
,0
q)cars
brand model purchasedate
------------------------
bmw   z4    2020.01.01
```
* more than 1 record can be added with values as list of lists
```
q)`cars insert (`tata`mahindra;`nexon`xuv500;2020.12.01 2020.10.02)
1 2
q)cars
brand    model  purchasedate
----------------------------
bmw      z4     2020.01.01
tata     nexon  2020.12.01
mahindra xuv500 2020.10.02
```
* we can also insert entire table, provided schema matches
* same can be done using `,:` adverb
```
q)`cars insert cars
3 4 5
q)cars
brand    model  purchasedate
----------------------------
bmw      z4     2020.01.01
tata     nexon  2020.12.01
mahindra xuv500 2020.10.02
bmw      z4     2020.01.01
tata     nexon  2020.12.01
mahindra xuv500 2020.10.02
q)cars,:cars
q)cars
brand    model  purchasedate
----------------------------
bmw      z4     2020.01.01
tata     nexon  2020.12.01
mahindra xuv500 2020.10.02
bmw      z4     2020.01.01
tata     nexon  2020.12.01
mahindra xuv500 2020.10.02
bmw      z4     2020.01.01
tata     nexon  2020.12.01
mahindra xuv500 2020.10.02
bmw      z4     2020.01.01
tata     nexon  2020.12.01
mahindra xuv500 2020.10.02
```
## Upsert
* can be used for both - keyed and unkeyed tables
* it will modify a row if it already exists and add if it doesn't
```
q)fruitprice:([fruit:`apple`banana] price:100 20; quantity:20 400)
q)fruitprice
fruit | price quantity
------| --------------
apple | 100   20
banana| 20    400
q)`fruitprice upsert (`apple;150;30)
`fruitprice
q)fruitprice
fruit | price quantity
------| --------------
apple | 150   30
banana| 20    400
```
* in order to add more than 1 row, we need to upsert a table
```
q)`fruitprice upsert (`apple`orange;150 30;30 50)
'length
  [0]  `fruitprice upsert (`apple`orange;150 30;30 50)
                   ^
q)`fruitprice upsert ([fruit:`apple`orange];price: 150 30;quantity: 30 50)
`fruitprice
q)fruitprice
fruit | price quantity
------| --------------
apple | 150   30
banana| 20    400
orange| 30    50
```
* upsert will work if order of columns are different or column values are missing
```
q)fruitprice
fruit | price quantity
------| --------------
apple | 150   30
banana| 20    400
orange| 30    50
q)`fruitprice upsert (fruit:`apple;price: 200 ;quantity: 20)
`fruitprice
q)fruitprice
fruit | price quantity
------| --------------
apple | 200   20
banana| 20    400
orange| 30    50
q)`fruitprice upsert ([]price:175 40;fruit:`apple`kiwi)
`fruitprice
q)fruitprice
fruit | price quantity
------| --------------
apple | 175   20
banana| 20    400
orange| 30    50
kiwi  | 40
```
