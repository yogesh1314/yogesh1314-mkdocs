# Functions

* block of code used to perform particular task
* can be executed by using its name and passing required variables
* defined using `{}`
* arguments required can be passed inside function definition in `[]`
* output is the result of execution of last line which doesn't end with `;`
  * if we want to suppress the output we add `;` to after last line of code

```q
q)f:{[a;b;c] a+b+c}
q)f[4;5;6]
15
q)f:{[a;b;c] a+b+c ; }         / if we add ; at end there is no output
q)f[4;5;6]
q)
```
* we can force the output using `:`, it also halts execution of function at that point

```q
q)f:{[a;b;c] :a+b+c; show "x"; }
q)f[4;5;6]
15
```
## Niladic Functions
* accept no arguments
* can be called with function name with no arguments or 1 argument which will be ignored

```q
q)f:{show "func exec "}
q)f[]
"func exec "
q)f[`a]
"func exec "
```

## Monadic functions

* takes exactly 1 argument
* not passing an argument or passing more than 1 argument, will cause error
* there are 3 special arguments `x`, `y` and `z`
  * if we don't specify arguments names in function definition and pass an argument then,
  * first argument will be stored in x, second in y and third in z

```q
q)f:{show "func exec "}
q)f[]
"func exec"
q)f[`a]
"func exec"
q)f3:{show x; show y; show z}
q)f3["Its";"Good";"Day"]
"Its"
"Good"
"Day"
q)f3:{x*y*z}
q)f3[4;5;6]
120
```

## Diadic function

* takes 2 arguments, we can specify arguments or use implict arguments `x`,`y` and `z`

```q
q)f:{[a;b] a*b}
q)f[5;4]
20
q)f4:{x*y}
q)f4[5;4]
20
```

## Triadic function

* takes 3 arguments, this is maximum where we can use implicit parameters `x`, `y` and `z`
```q
q) f:{x+y+z}
q) f[1;2;3]
6
```

## Multivalent function

* arguments more than 3
* here we specifically need to define argument names
```q
q)f:{x+y+z+a}
q)f[1;2;3;4]
'rank
  [0]  f[1;2;3;4]
       ^
q)f:{[x;y;z;a] x+y+z+a}
q)f[1;2;3;4]
10
```
## Special functions

### Plus `+`
* these can be applied in prefix or infix form
* user defined functions cant be applied in infix form
```q
q)f:+
q)+[1;2]
3
q)1 + 2
3
q)1 f 2
'type
  [0]  1 f 2
       ^
```
## Max arguments

* max arguments we can pass is 8
* we cant define a function with 9 parameters, we get `'params` error
* we can pass more number of arguments in complex data structure as list or dictionary

```q
q)f:{[a;b;c;d;e;f;g;h;i] a+b+c+d+e+f+g+h+i}
'params
q)f:{[a;b;c;d;e;f;g;h;i] a+b+c+d+e+f+g+h+i}
'params
q)d:(10?" ")!1+til 10
q)d
i| 1
y| 2
c| 3
z| 4
m| 5
l| 6
w| 7
h| 8
e| 9
v| 10
q)f:{[d]sum d}
q)f[d]
55
```

## `type` of function

* function is of type 100h
* get function_name gives its structure

```q
q)type f
100h
q)get f
0x78390002
,`d
`symbol$()
,`
8 4
"..f"
""
-1
"{[d]sum d}"
q)get [f][1]
,`d
```
## Function within function

* we can call and define a function within a function
* if we defined unnamed function within function its called lambda functions

### lambda
* lambdas need not be defined within a function we can define unnamed function(lambdas) by themselves

```q
q)z:{f:{x*x}; f[x]+10}
q)z[2]
14
q)z:{10+{x*x}[x]}
q)z[2]
14
q){x+y}[3;4]          / lambda without assigning to any variable and outside a function
7
```
## Projections

* if we need to change one parameter and keeping rest of parameters same, we use Projections
* if we change definition of f, definition of its projections wont change
* we can create projection of projection of another function/projection

```q
q)f:{x*y}
q)f[4;5]
20
q)g:f[;5]
q)type g
104h
q)g                    / g is projection on f with y=5
{x*y}[;5]
q)g[4]   
20
```
