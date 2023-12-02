# Scripts

* we can save, load and execute q expressions in `.q` file
* once q scipt is saved as `.q` file, we can load it from q console using `\l scriptFilePath.q` or from command line using `q scriptFilePath.q`

```q
PS C:\Users\Yogesh\Desktop\scripts> q .\test.q
KDB+ 3.5 2017.06.15 Copyright (C) 1993-2017 Kx Systems
w32/ 4()core 4095MB Yogesh yogesh-hp 192.168.56.1 NONEXPIRE
Welcome to kdb+ 32bit edition
To exit, type \\
"Loading q script"
q)\\
PS C:\Users\Yogesh\Desktop\scripts> q
KDB+ 3.5 2017.06.15 Copyright (C) 1993-2017 Kx Systems
w32/ 4()core 4095MB Yogesh yogesh-hp 192.168.56.1 NONEXPIRE
Welcome to kdb+ 32bit edition
To exit, type \\
q)\l test.q
"Loading q script"
q)\\
```

* if we define variables in q script same will be accessible in q console once script is loaded, but if same variables are already defined they will be overwritten

```q
PS C:\Users\Yogesh\Desktop\scripts> q
KDB+ 3.5 2017.06.15 Copyright (C) 1993-2017 Kx Systems
w32/ 4()core 4095MB Yogesh yogesh-hp 192.168.56.1 NONEXPIRE

Welcome to kdb+ 32bit edition
To exit, type \\

q)a:88
q)a
88
q)\l test.q
"Loading q script"
q)a
90
q)b
42 84
q)
```

* `/` used for single line comment
* multi line comments would start with `/` and end with `\`
* expression can be defined in multiple lines but when carried forward to next line whitespace is necessary
* whitespace needs to be there especially in function definition where multi-line definition of function needs whitespace before each new line including last `}`

```q
PS C:\Users\Yogesh\Desktop\scripts> q
KDB+ 3.5 2017.06.15 Copyright (C) 1993-2017 Kx Systems
w32/ 4()core 4095MB Yogesh yogesh-hp 192.168.56.1 NONEXPIRE

Welcome to kdb+ 32bit edition
To exit, type \\

q)\l test.q
"Loading q script"
'(
  [0]  C:\Users\Yogesh\Desktop\scripts\test.q:17: tab2:(a:10?`1; b:10?.z.d;
                                                                          ^
  [0]  (<load>)

  )\
q)\v
`s#`a`b`tab
q)tab
a b          c
----------------------
n 2004.02.01 1.963762
j 2005.06.06 2.585456
c 2017.06.05 2.579898
h 2013.02.11 2.033321
a 2010.12.20 0.8904193
b 2005.03.04 1.508861
o 2009.05.09 3.925165
j 2016.09.30 2.673548
c 2017.09.06 3.555858
b 2015.08.10 2.057985
q)\l test.q
"Loading q script"
'{
  [0]  C:\Users\Yogesh\Desktop\scripts\test.q:17: fun:{[a;b]
                                                           ^
  [0]  (<load>)

  )
```
## Command line parameters

* we can launch script with key value parameters
* `q script.q -key1 value1 -key2 value2 .... upto n`
* `.z.x` stores our command line arguments as strings and can be used to access them inside the script
* `.Q.opt` is useful to convert output of `.z.x` into meaningful dictionary with values passed with `-key` becomes as keys of dictionary
* we can store output of `.Q.opt .z.x` into a variable and this dictionary can be used within script in many ways
* another useful function is `.z.f` which stores name of script when script has finished loading

```q
q) / without .Q.opt
PS C:\Users\Yogesh\Desktop\scripts> q .\test.q -key test -xor test2
KDB+ 3.5 2017.06.15 Copyright (C) 1993-2017 Kx Systems
w32/ 4()core 4095MB Yogesh yogesh-hp 192.168.56.1 NONEXPIRE

Welcome to kdb+ 32bit edition
To exit, type \\

"Loading q script"
q) show .z.x
"-key"
"test"
"-xor"
"test2"
q)/ with .Q.opt
PS C:\Users\Yogesh\Desktop\scripts> q .\test.q -key test -xor test2
KDB+ 3.5 2017.06.15 Copyright (C) 1993-2017 Kx Systems
w32/ 4()core 4095MB Yogesh yogesh-hp 192.168.56.1 NONEXPIRE

Welcome to kdb+ 32bit edition
To exit, type \\

"Loading q script"
q) show .Q.opt .z.x
key| "test"
xor| "test2"
q)
```
