# Tools

## oprofile
```bash
$ opcontrol --init
$ opcontrol --status
$ opcontrol --start
$ opcontrol --list-events
$ opcontrol --reset
$ opcontrol --dump
$ opcontrol --stop
$ opreport image:path/to/your/exe -l
$ opannotate image:path/to/your/exe -s
```
### NOTES
* Compile your program with `-g` option.
* Do `--stop` or `--shutdown` the oprofiled process after profiling.
* Oprofile could only do profiling on the active process of your program. Thus it cannot
  help you much on the lock contention issues.



# Tips
===============
* Do not use too many parameters on your functions.
  Because on IA64 architecture, the arguments are passed through registers.
  When there are too many of them, arguments have to be pushed to stack, which is
  usually maintained by memory. That would slow your function calling down.
* Move the unnecessary repeated-calculations, namely invariants, out from loops.
* Use bit-operations, instead of multiplication or division, when necessary.
* Always concern about the cache when operating on arrays.
* Naming your threads might help with your debuging.

# To Be Added
* Cache coherence & cache consistency
* Memory Barriers
* Memory Models
