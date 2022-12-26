Common practice in data analysis is compare an activity for a specific time period against the average activity(profile) over a wider time 
* We will compare trading volumes on a specific date against the average daily volume (ADV)
* first, let's create hdb for 60 days
```
q)\l fakedb.q
USAGE: makedb[NUM QUOTES;NUM TRADES] eg makedb[100000;10000]

makedb1[NUM QUOTES;NUM TRADES;DATE;RANDOMISED COUNT FACTOR] eg makedb1[100000;10000;.z.d;.3]

makehdb[HDBDIR;NUM DAYS;APPROXIMATE NUM QUOTES PER DAY; APPROXIMATE NUM TRADES PER DAY] eg makehdb[`:hdb; 5; 100000; 10000]

makecsv[CSVDIR;NUM DAYS;NUM QUOTES;NUM TRADES]

q)makehdb[`:hdb;60;100000;10000]
2022.12.26T12:29:09.312 saving data for date 2014.04.21 to :hdb
2022.12.26T12:29:09.380 saving data for date 2014.04.22 to :hdb
..
..
2022.12.26T12:29:13.521 saving data for date 2014.07.10 to :hdb
2022.12.26T12:29:13.580 saving data for date 2014.07.11 to :hdb

q)\l hdb
```
* we want to get out average daily profile for which we define time period 

```
q)dts:(2014.04.22;2014.05.22)
q)p: select sum size by date,sym, 5 xbar time.minute from trades where date within dts
q)p
date       sym  minute| size
----------------------| -----
2014.04.22 AAPL 08:00 | 8536
2014.04.22 AAPL 08:05 | 15896
2014.04.22 AAPL 08:10 | 18935
2014.04.22 AAPL 08:15 | 6874
```
* next we update to get cumulative sum using `sums`
```
q)p: update sums size by date,sym from p
q)p
date       sym  minute| size
----------------------| ------
2014.04.22 AAPL 08:00 | 8536
2014.04.22 AAPL 08:05 | 24432
2014.04.22 AAPL 08:10 | 43367
2014.04.22 AAPL 08:15 | 50241
2014.04.22 AAPL 08:20 | 65972
2014.04.22 AAPL 08:25 | 86996
2014.04.22 AAPL 08:30 | 97990
2014.04.22 AAPL 08:35 | 116046
```
* next we get average size for each sym and minute
```
q)p1: select avgSize: (sum size)%count distinct date by sym,minute from p
q)p1
sym  minute| avgSize
-----------| --------
AAPL 08:00 | 25892.09
AAPL 08:05 | 51668.17
AAPL 08:10 | 82826.09
AAPL 08:15 | 107311.2
AAPL 08:20 | 135899.5
```
* now we need to rack this data
* first, we create rack - which is cross of distinct syms and all possible time interval between start and end with 5 minute time slice
```
q)rack: (select distinct sym from p) cross ([]minute: start+00:05*til `int$end-start)
q)rack
sym  minute
-----------
AAPL 08:00
AAPL 08:05
AAPL 08:10
AAPL 08:15
AAPL 08:20
AAPL 08:25
```
* we can then left join this to table of daily averages
```
q)rack lj p1
sym  minute avgSize
--------------------
AAPL 08:00  25892.09
AAPL 08:05  51668.17
AAPL 08:10  82826.09
AAPL 08:15  107311.2
AAPL 08:20  135899.5
AAPL 08:25  163998.3
AAPL 08:30  192402.9
AAPL 08:35  216513.4
AAPL 08:40  244737.7
AAPL 08:45  270592.9
AAPL 08:50  294260
```

* Now let's create a script for same
```
q)adv:{[startdate;enddate;bucketsize]
    // extract the bucketed totals
    p: select sum size by date, sym, bucketsize xbar time.minute from trades where date within (startdate;enddate);

    // make total cumulative
    p: update sums size by date, sym from p;

    // calculate avergae daily volume across all dates
    p: select avgSize: (sum size)% count distinct date by sym,minute from p;

    // rack the data
    start: min exec minute from p;
    end: max exec minute from p;
    rack: (select distinct sym from p) cross ([]minute: start+bucketsize*til `int$(end-start)%bucketsize);

    // join and fill racked data
    update fills avgSize by sym from rack lj p
    }
    // 
```
* now we have loaded the function and hdb in same session
```
q)adv
{[startdate;enddate;bucketsize] p: select sum size by date, sym, bucketsize x..
q)adv[2014.04.22;2014.05.22;5]
sym  minute avgSize
--------------------
AAPL 08:00  25892.09
AAPL 08:05  51668.17
AAPL 08:10  82826.09
AAPL 08:15  107311.2
AAPL 08:20  135899.5
```
* we can compare by joining on cumulative size data for specified date
* we can also fill forward to remove any blank value
```
q)update sums size from select sum size by sym,5 xbar time.minute from trades where date=2014.05.23
sym  minute| size
-----------| ------
AAPL 08:00 | 31991
AAPL 08:05 | 53513
AAPL 08:10 | 88931
AAPL 08:15 | 125699
AAPL 08:20 | 151826
q)adv[2014.04.22;2014.05.22;5] lj update sums size from select sum size by sym,5 xbar time.minute from trades where date=2014.05.22
sym  minute avgSize  size
---------------------------
AAPL 08:00  25892.09 17140
AAPL 08:05  51668.17 45260
AAPL 08:10  82826.09 58595
AAPL 08:15  107311.2 87317
q)update fills size from adv[2014.04.22;2014.05.22;5] lj update sums size from select sum size by sym,5 xbar time.minute from trades where date=2014.05.22
sym  minute avgSize  size
---------------------------
AAPL 08:00  25892.09 17140
AAPL 08:05  51668.17 45260
AAPL 08:10  82826.09 58595
AAPL 08:15  107311.2 87317
AAPL 08:20  135899.5 113833
AAPL 08:25  163998.3 131745
AAPL 08:30  192402.9 176591
AAPL 08:35  216513.4 184667
AAPL 08:40  244737.7 197525
```
* now we can compare 1 date's data to adv for date range

```
q)update sizeComp: 100*size%avgSize from update fills size from adv[2014.04.22;2014.05.22;5] lj update sums size from select sum size by sym,5 xbar time.minute from trades where date=2014.05.22
sym  minute avgSize  size   sizeComp
------------------------------------
AAPL 08:00  25892.09 17140  66.19783
AAPL 08:05  51668.17 45260  87.59744
AAPL 08:10  82826.09 58595  70.74462
AAPL 08:15  107311.2 87317  81.36804
AAPL 08:20  135899.5 113833 83.76265
AAPL 08:25  163998.3 131745 80.33317
AAPL 08:30  192402.9 176591 91.7819
```
* now we can make new function for this comparison 

```
q)comparetoAdv:{[startdate;enddate;bucketsize;comparisonDate]
    res: update sums size from select sum size by sym,bucketsize xbar time.minute from trades where date=comparisonDate;
    res1: update fills size from adv[2014.04.22;2014.05.22;5] lj res;
    update sizeComp: 100*size%avgSize from res1 
    };
q)compareToAdv[2014,04,22;2014.05.22;5;2014.05.22]
sym  minute avgSize  size   sizeComp
------------------------------------
AAPL 08:00  25892.09 17140  66.19783
AAPL 08:05  51668.17 45260  87.59744
AAPL 08:10  82826.09 58595  70.74462
AAPL 08:15  107311.2 87317  81.36804
AAPL 08:20  135899.5 113833 83.76265
```