# IPC - Sending and Receiving 

* server at port 1234 and client with handle opened to server
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
## Sync Send and Receive
* Synchronous message: client sends the message and blocks the handle until servers sends a response
* To send a message we can use "string" format along with handle h
```q
//client
q)h"2+2"
4i
q)h"g:23"
q)
```

```q
//server
q)g
23
```
* Messages can also be send in functional form - `handle(function;arg1;arg2;..;arg n)`
```q
//client
q)h(+;2;2)
4i
q)h(set;`abc;245)
`abc
```
```q
//server
q)abc
234
```
* For assigning value in functional form we use variable name as symbol or we can use function name as string
* We can also execute functions defined in server side from client
```q
//server
q)f:{x*x}
q)f
{x*x}
```
```q
//client
q)h(`f;2)
4
q)h("f";2)
4
```
* we can also send client side function to server along with arguments and get the results with server running the execution 
```q
//client
q)f2:{x-y}
q)h(f2;45;23)
22
```
```q
//server
q)f2
'f2
  [0]  f2
       ^
```
* we can get data from server using `get` function or using variable name directly which exists in server
```q
//server
q)list: 1 2 3 4 5
```
```q
//client
q)h(get;`list)
1 2 3 4 5
q)h(get;"list")
1 2 3 4 5
q)h"list"
1 2 3 4 5
q)h("list")
1 2 3 4 5
```

## `.z.pg`
* message handler function for handling synchronous messages
* executes everytime process/server receives synchronous message
```q
//server
q).z.pg:{0N!value x}
q)13
```
```q
//client
q)h"2+3+8"
q)13
```
## Async Send and Receive 
* client doesn't wait for server to send the response, it continues its execution
* to send a async message we use negative handle
* Sending async messages does not send the message immediately. It serializes x, and queues it for sending at a later time - either when the main loop spins, or a blocking request is issued on that handle
* `.z.ps` async message handler
```q
//client
q)(neg h)"2*5"
q)(neg h)(+;2;5)
q)(neg h)"mko:908"
q)h"" // to ensure async messages queue is processed
q)(neg h)[] //used to flush the queue
```
```q
//server
q)mko
908
```
* To confirm async messages are received and processed, chase with sync message `q)h""`
* To block until all pending outgoing messages have been written into the handle
```q
q)(neg h)[] //or 
q)(neg h)(::)
```
* if we want to ensure an async message really has been sent as soon as possible, use: 
```q
q)(neg h)"x";
q)(neg h)[];
```