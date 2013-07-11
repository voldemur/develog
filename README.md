# Tools
Sharpen Your Gears.

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
# view all general registers
(gdb) i reg
# view specific register
(gdb) i reg rip
# view whole set of registers
(gdb) i all-reginsters
# combine `bt` and `i locals`
(gdb) bt full
```

Ctrl+X Ctrl+A opens the window to present the corressponding source code, which sometimes is a bit more convenient.

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

## svn
```bash
# revert a bad commit
$ svn merge -r COMMITTED:PREV .
# view diff with vim
$ svn diff | vim -
# generate and apply patches
$ svn diff > patch
$ patch -p0 < patch
```

## miscs
```bash
# issue a command as a daemon
$ (command &)
# simple calculator only for integers
$ echo $((1<<30))
# floating calculation
$ echo '3.14 * 1.5^2' | bc -l
# radix convertion
$ printf "%x\n" 12345
```


# Tips
* Optimize the code that is on the critical executive path of your program, and do not optimize it too early.
* Do not use too many parameters with your functions.
  In IA64 architecture, the arguments are passed through registers.
  When there are too many of them, arguments have to be pushed to stack, which is
  usually maintained by memory. That would slow your function calling down.
* Move the unnecessary repeated-calculations, namely invariants, out from loops.
* Use bit-operations, instead of multiplication or division, when necessary.
* Always concern about the cache when operating on data sets.
* Naming your threads might help with your debuging. `prctl(PR\_SET\_NAME, thread\_name, 0, 0, 0)`
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
* Spinlock
  * Choose spin lock on tiny critical section, which will reduce the context switching.
  * Using spin lock, pay attention to race of threads running on the same cpu core.
  * By convention, threads tring to acquire a spinlock should yield the cpu, after spinning enough times.
* The accuracy of `usleep` depends on the version of kernel and the load of the system, roughly millisecond other than microsecond.

# Networks
* Each TCP connection is identified by a 4-tuple combination: <client IP, client port, server IP, client port>.
* The sliding-window protocol is a flow control for receiver, while the congestion window for intermediate network.
* The active close end of the connection enters into TIME-WAIT state, for the reasons
  * The last ACK may be lost.
  * In case of the establishment of the incarnation of the old connection.
* The passive close end of the connection enters into CLOSE-WAIT state. A perssist connection of this state often indicates a bug.
* Normally, connection is established already after the synchronous `connect` returned, even before the server `accept` it.
  Thus, at the server side, before a `accept`, a connect may have been established, even though the corresponding `fd` has not been allocated util `accept`.

# MM
* Kernel and user process sharing the same address space, avoids the TLB flush everytime when doing syscall. 
* With CONFIG_SLAB_DEBUG enabled, slab allocator marks either end of a object. If the marker(red zoning) is disturbed, a bug is reported. One object in free stat being filled with the pattern 0x5A(poisoning), if a newly allocated object does not match this pattern, a bug is reported.  

# To Be Added
* SystemTap
* Cache Coherence & Cache Consistency
* Memory Barriers
* Memory Models
