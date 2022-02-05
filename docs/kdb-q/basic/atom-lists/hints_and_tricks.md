# Hints and Tricks

## Time of expression
* `\t` is used to see runtime of an expression in milliseconds
* `\t:n` this is used when we want to run same expression `n` times
* for sum of two large lists/array, q is quite fast because of vector calculation. In other languages its iterative method to go to each element of both lists then add, in vector calculation operation is performed simultaneously on every element of lists

## [help.q](https://github.com/KxSystems/help/blob/master/help.q)
* This script can be put in QHOME directory and loaded in q.q `\l help.q`, thus when q sessions opens q.q is called and help.q will be loaded
* It acts like man page for q, providing info for most of the cases required in daily use

```q
q)help`
attributes| data attributes
cmdline   | command line parameters
data      | data types
debug     | debug - debugger commands
define    | assign, define, control and debug
doth      | .h namespace: markup
dotj      | .j namespace: JSON
dotq      | .Q namespace: utilities
dotz      | .z namespace: system information and callbacks
envvar    | envvar - environment variables
errors    | error messages
internal  | negative bang - -n!x - internal system calls
iterator  | iterators (formerly adverbs)
keyword   | keywords (functions)
kill      | kill - killing a q session
lic       | licenses - kdb+ license files
operator  | operators (functions)
save      | save/load tables
syscmd    | system commands
temporal  | temporal - date & time casts
```

* providing key in will show all existing options

```q
q)help`cmdline
q [f] [-b] [-c r c] [-C r c] [-g 0|1] [-l] [-L][-o N] [-p N] [-P N] [-q]
      [-r :H:P] [-s N] [-t N] [-T N] [-u|U F] [-w N] [-W N] [-z 0|1]

f load script (*.q, *.k, *.s), file or directory

-b               block client write access
-c r c           console maxRows maxCols
-C r c           http display maxRows maxCols
-e [0|1|2]       disable|enable|display error trap clients
-g [0|1]         disable|enable immediate garbage collect, default 0
-l               log updates to filesystem
-L               as -l, but sync logging
-o N             offset hours (from GMT: affects .z.Z/T/P/N), default 0
-p N             port kdbc(/jdbc/odbc) http(html xml txt csv)
-p -N            port multithreaded kdbc
-P N             printdigits, default 7, 0=>all
-q               quiet, no startup banner text
-r :H:P          replicate from :host:port
-s N             slaves for parallel execution
-S N             set random seed
-t N             timer milliseconds, default 0 (=> none)
-T N             timeout seconds(applies to all client queries), default 0 (=> none)
-u F             usr:pwd file, no access above start directory
-u [0|1]         enable|disable system escapes
-U F             as -u, but no file restrictions
-w N             workspace MB limit (default: 2*RAM)
-W N             week offset, default 2, 0=>saturday
-z [0|1]         "D" uses 0:mm/dd/yyyy or 1:dd/mm/yyyy, default 0
```

* Helps to reduce time in opening up ref page of kdb
