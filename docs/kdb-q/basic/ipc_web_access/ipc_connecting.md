# IPC: Connecting

Kdb+ uses tcp/ip connection to talk between two kdb+ processes or when using http browser

All messages in kdb+ connection goes via message handler functions

* to declare port number inside a q process we use `/p [port_number]`, we can also assign port number when starting the process using `-p [port_number]` 

* when inside a process we can use `\p` to see the port number, by default if port number is not declared it `0i`

```q
user@host:~ $ q
q)\p 1234
q)\p 
1234i

user@host:~ $ q -p 1234
q)\p 
1234i

user@host:~ $ q
q)\p
0i
```

* let's create 2 q processes:
** server - one which is serving queries
** client - one which is making queries

* we can open a connection from client to server using function `hopen`
* we can also see which socket this handle is assigned to by checking value of `h`
```q
q)// server
q)\p 
1234i
```

```q
q)// client
q)h:hopen 1234
q)h
3i
```

* we can use another version of hopen command by specifyign host and port number - `hopen ``:host:port`
```q
q)// client 
q)h1: `:localhost:1234
q)h1
3i
q)h1:`::1234 // if we dont provide host, it defaults to localhost
q)h1
4i

```
* we can see all the handles open on client process by checking dictionary `.z.W`

```q
q)//client
q).z.W
q).z.W
3|
4|
```

* to close a connection we use function `hclose`
* we can also give handle number as an argument to `hclose` function to close a handle - handle number can be checked from `.z.W` command
  
```
q)//client
q)hclose h1
q)hclose 4i
```

## Message Handler Functions

* we can reset any of the message handler function to their original definition by using `\x function_name` or `\x .z.pw`
  
### `.z.pw`

* used to check password of user while opening connection 
* arguments to `.z.pw` function are username and password
* can be reset to default using `\x .z.pw`
  
```q
// server
q)permittedusers:`abc`xyz`pqr
q).z.pw:{[u;p] (u in permittedusers) and (p~"pass")}
```

```q
//client
q)h:hopen `::1234
'access
  [0]  h:hopen `::1234
         ^
q)h:hopen `::1234:abc:pass
q).z.W
7|
```

### `.z.po`

* this is executed whenever a handle is opened and `.z.pw` has executed successfully
* we can print details of just opened handle in this function, like: ip address, username, date time and handle number
* argument passed to function `.z.po` is handle number
* `.z.a` - show ip address
* `.z.p` - current timestamp
* `.z.u` - user opening handle

```
//server
q).z.po:{[x] show(.z.a;.z.p;.z.u;x)}
{[x] show(.z.a;.z.p;.z.u;x)}
q)2130706433i
2022.02.21D03:38:51.578467000
`abc
10i
```q


```q
//client
q)h2:hopen `::1234:abc:pass
```

### `.z.pc`

* we can use `.z.pc` to be executed whenever a handle is closed to server

```
//server
q).z.pc:{[x] show(.z.h;.z.p;x)}
q)`ykr-mbam1.local
2022.02.21D03:42:57.717442000
10i
```q


```q
//client
q)hclose h2
```

## Timeout

* we can also define timeout when opening a handle

```q
// server
q)system "sleep 60"
```

```q
// client
q)h2:hopen (`::1234:abc:pass;10)
'timeout
  [0]  h2:hopen (`::1234:abc:pass;10)
```

## Restricting user access on start-up

* we can define username and password combination in a file and specify it using `-u filepath` when strating q session
* Only username password combination in the file will be allowed when opening handle

```q
// server

ranayk@ykr-mbam1: $ cat users.txt
abc:pass1
xyz:pass2
qwe:pass3

ranayk@ykr-mbam1: $ q -p 1234 -u users.txt
KDB+ 4.0 2021.04.26 Copyright (C) 1993-2021 Kx Systems
m64/ 8(16)core 8192MB ranayk ykr-mbam1.local 127.0.0.1 EXPIRE 2022.05.30 yankeekiloromeo@gmail.com KOD #4176402

q)
```

```q
//client
q)h2:hopen `::1234:abc:pass
'access
  [0]  h2:hopen `::1234:abc:pass
          ^
q)h2:hopen `::1234:abc:pass1
q)h2:hopen `::1234:xyz:pass1
'access
  [0]  h2:hopen `::1234:xyz:pass1
          ^
q)h2:hopen `::1234:xyz:pass2
q)
```