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
#### NOTES
* Compile your program with `-g` option.
* Do `--stop` or `--shutdown` the oprofiled process after profiling.
* Oprofile could only do profiling on the active process of your program. Thus it cannot
  help you much on the lock contention issues.


## strace
```bash
$ strace -f -c -p `pidof PROGRAM`
```

## GDB
```bash
$ gcore `pidof PROGRAM` # generate a core dump file
```

```bash
# lock the thread to prevent it being rescheduled, while debuging other than in step mode
(gdb) set scheduler-locking on
```

```bash
# add debuginfo for a separate dso
(gdb) i sharedlibrary
From                To                  Syms Read   Shared Object Library
0x00000038fd800a70  0x00000038fd8166de  Yes         /path/to/lib.so
(gdb) add-symbol-file /path/to/lib.so.debug 0x00000038fd800a70
```

## top
```bash
$ top -H -d 1
$ # in top
$ # press `1` to ses individual cpu core status
$ # press `f` then `j` to see which core a specific thread is running on
```


# Tips
===============
* Optimize the code that is on the critical executive path of your program, and do not optimize it too early.
* Do not use too many parameters with your functions.
  )n IA64 architecture, the arguments are passed through registers.
  When there are too many of them, arguments have to be pushed to stack, which is
  usually maintained by memory. That would slow your function calling down.
* Move the unnecessary repeated-calculations, namely invariants, out from loops.
* Use bit-operations, instead of multiplication or division, when necessary.
* Always concern about the cache when operating on data sets.
* Naming your threads might help with your debuging.`prctl(PR_SET_NAME, thread_name, 0, 0, 0)`
* Try cpu binding.
* Be careful of the false-sharing issues on global variables.
  * Unintentionally use the same cache line in different threads.
  * Use padding bytes between them
* Factors that impact on the scalability of muti-threaded programs.
  * Context Switching
  * Cost of Synchronization
  * Unbalanced Loads
  * Memory Saturation
* Granularity
  * Large granularity of the concurrent task increase the performance.
  * Large granularity of the critical section harms the performance.
    * Choose spin lock on tiny critical section, which will reduce the context switching.
    * Using spin lock, pay attention to race of threads running on the same cpu core.
    * By convention, threads tring to acquire a spinlock should yield the cpu, after spinning enough times.

# To Be Added
* SystemTap
* Cache Coherence & Cache Consistency
* Memory Barriers
* Memory Models
