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