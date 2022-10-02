# Debugging

* normal mode for q is `q)` q with single parenthesis
* `q))` this means error trap or debug mode, `q)))` means error trap within error trap mode
* typing single `\` back slash used to toggle to **k** console
* to return back to **q** console type `\` from **k** console

```q
q)\                     / goes into k mode


\               
q)                      / comes back to q mode
 ```
* if an error occurs while executing commands in console, it goes in error trap mode showing what type of error occured -  `'type` is most common error :P
* when it goes into error trap mode, its stuck where the error occurred, so if its within the function we can check what are the values passed which caused error

```q
q)f:{x*y}
q)f[3;4]
12
q)f[`sym;4]
'type
  [1]  f:{x*y}           
 q))x
 `sym
 q))y
 4
 ```
* 3 useful things to help debugging:
** `.z.s` shows function definition where its stuck when in error trap mode -  useful for nested functions
** `'` tick  - used to go 1 level higher in case of nested functions
** `:` or `:value` - we can force to continue the execution with `:value` meaning use value provided with `:` instead of where error is occurring and continue the execution

```q
q)f:{x*x}
q)g:{f[x]*y+10}
q)g[3;4]
126
q)g[`aa;4]
'type
  [2]  f:{x*x}
           ^
q)).z.s                     / print function where error is
{x*x}
q))'                        / go 1 level up - it goes from f to g
'
  [1]  g:{f[x]*y+10}
          ^
q)):3                       / continue execution with value 3 - thus 10+4=14*3=42
42
```

* another example

```q
q)f:{x+10}
q)g:{x*2}
q)h:{n:f[x]+4; m:n+g[x] +g[y]}
q)h[2;10]
40
q)h[`aa;10]                             
'type
  [2]  f:{x+10}
           ^
q)).z.s                             / error at f[x]
{x+10}
q)):10                              / we force execution with 10 - thus f[x] is 10 now - thus n:14
'type
  [2]  g:{x*2}
           ^
q)).z.s                             / now we are stuck at g[x]
{x*2}
q)):2                               / we force execution with 2 - thus g[x] is 2 now - thus m:14+2:16 + g[y]:10*2:20 - thus return value is m+20:16+20
36
```
* to help with debugging within the function we use `:` and `'`
* now these will help in debugging function line by line

```q
q)f:{a:x+3; y+5; a}
q)f[3;4]
6
q)f:{a:x+3; :y+5; a}                / we can force return with :
q)f[3;4]
9
q)f:{a:x+3; 'polo; a:99; a}         / we can use ' as 'polo
q)f[3]
'polo
  [1]  f:{a:x+3; 'polo; a:99; a}
                  ^
q))a                                / we check a isnt assigned 99 till now coz there break statement before it
6
q))\
q)
```
* we can also use `0N!` and `-3!` to return each statement of function as its evaluated or print as string

```q
q)f:{0N!a:x+3;0N!a*:a; 0N!a+3}
q)f[3]
6
36
39
39
q)f:{0N!a:x+3;0N!a*:a; -3!a+3}
q)f[3]
6
36
"39"
```
