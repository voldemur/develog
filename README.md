# Tools
Sharpen Your Gears, Release Your Mind.

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

## GCC

`__attribute__` on variable or function declaration:
* `((noreturn))`, this function would never return.
* `((aligned(sizeof(long))))`, align this variable or variables of this type on specific boundary.
* `((packed))`, do not add padding to fields of this struct.
* `((unused))`, this symbol would not be used, suppress the compiler's complains.

```bash
# Print the name of each header file used, in addition to other
# normal activities. Each name is indented to show how deep in
# the #include stack it is.
$ cc -H main.c
```

## GDB
```bash
$ gcore `pidof PROGRAM` # generate a core dump file
```

```bash
# lock the thread to prevent it being rescheduled, while debuging
# other than in step mode
(gdb) set scheduler-locking on
# view all general registers
(gdb) i reg
# view specific register
(gdb) i reg rip
# view whole set of registers
(gdb) i all-reginsters
# combine `bt` and `i locals`
(gdb) bt full
# add debuginfo for a separate dso
(gdb) i sharedlibrary
From                To                  Syms Read   Shared Object Library
0x00000038fd800a70  0x00000038fd8166de  Yes         /path/to/lib.so
(gdb) add-symbol-file /path/to/lib.so.debug 0x00000038fd800a70
# add debuginfo for the whole executable
(gdb) symbol-file /path/to/tair_server.debug
```

Ctrl+X Ctrl+A opens the window to present the corressponding source code, which sometimes is a bit more convenient.


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
# turn on/off the swap facility
$ sudo swapoff -a
$ sudo swapon -a
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
* `likely` or `unlikely` macros are used to direct compilers arrange the instructions properly, to maximize the branch prediction of CPU, to stall the instruction pipelines less.
* To make GCC optimize your code partially, you could use `__attribute__((optimize(0)))` on the __declaration__ of specific function.

# Assembly

I love the AT&T syntax.

## syntax

Memory operands are represented as `section:disp(base, index, scale)`. When the `disp` and `scale` are constant, `$` shouldn't be prefixed. The `base` and `index` must be registers.

Use `__asm__` instead of `asm` if the latter conflicts with something in the program.

Do use `local labels` in embedded assembly, preventing the multi-definition issue when functions are inlined.
```c
void foo() {
  /*
   * cannot be expanded twice in the same context.
   */
  asm volatile(
      "jmp L1;"
      "L1:":::);
}
```

`local labels` are declared as a single digit followed by letter `:`, and referenced by this digit followed by a letter `f` for `forwads` or `b` for `backwards`.

```c
void foo() {
  asm volatile(
      "jmp 1f;"
      "1:":::);
}
```

### inline assembly

```c
asm ( <assembler template>
    : ["constraints"(var)] [,"constraints"(var)]  /* output operands */
    : ["constraints"(var)] [,"constraints"(var)]  /* input operands */
    : ["register"] [,"register"] [,"memory"]      /* clobbered registers */
    );
```

Common used constraints are the following:
* `r`, register operands, any one of the followings
  * `a`, `%rax`, `%eax`, `%ax`, `%al`
  * `b`, `%rbx`, `%ebx`, `%bx`, `%bl`
  * `c`, `%rcx`, `%ecx`, `%cx`, `%cl`
  * `d`, `%rdx`, `%edx`, `%dx`, `%dl`
  * `S`, `%rsi`, `%esi`, `%si`
  * `D`, `%rdi`, `%edi`, `%di`
* `q`, register operands, any one of `a`, `b`, `c` or `d`
* `m`, memory operands
* digit, matching constraints, operands are used both for input and output
* `f`, floating point register

Contraints modifiers are the following:
* `=`, operand is write-only, thus previous value in it would be decarded 


Use `__asm__` instead of `asm` if the latter conflicts with something in the program.

```c
/* syscall write */
inline int as_write(int fd, char *buf, size_t n) {
  int ret;
  asm (
      "int $0x80\n\t"
      : "=a"(ret)
      : "0" (4), "b"(fd), "c"(buf), "d"(n)
      );
  return ret;
}
```

## instructions

Instructions(X86) that could be used with `lock` are the following:
* `lock` prefix is automatically assumed for `xchg`
* `bts`, `btr`, `btc`
* `xchg`, `xadd`, `cmpxchg`, `cmpxchg8b`
* `add`, `sub`, `xor`, `adc`, `sbb`
* `and`, `or`, `not`, `neg`, `inc`, `dec`

# Atomic Operations

Instructions could be classified into R(Read), W(Write) and RMW(Read-Modify-Write).

Rs and Ws are guaranteed to be atomical by modern X86 architecture, provided that the operand is aligned on their natually boundaries. With unaligned operand, Rs and Ws could also be atomically performed, only if the operand is cached in the same cache-line.

As for RMWs, they are not atomical since they involve multiple memory accesses. For these instructions, `lock` prefix should be add to emit a `LOCK#` signal to the memory bus. While the  `LOCK#` signal is asserted, the corresponding memory location could not be accessed by other processors. Through this, the atomicity of RMWs are guaranteed.

# Memory Ordering

The term `memmory ordering` refers to the order in which the processor issues reads(loads) and writes(stores) through the system bus to system memory.

Different architecture support different memory-ordering models.
For example, the Intel386 processor enforces programming ordering(string ordering), where reads and writes are issued on the system bus in the order they occur in the instruction stream under all circumstances.
To allow performance optimization of instruction execution, the IA32 architecture allows departures from strong model, from Pentium 4, Intel Xeon, and P6 family processors.
These processors allow performance enhancing operatons such as allowing reads to go ahead of buffered writes.

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

State, in memory, the most valuable thing of this world.

Memory banks in Linux are treated as __nodes__, even if the system has only one node.
Every node is divided into __zones__, represented as ZONE_DMA, ZONE_NORMAL, and ZONE_HIGHMEM, while all memory are comprised of fixed-size chunks called __page frames__ whose size are typically 4KB.
ZONE_DMA is located at the lower part, used by DMA devices.
ZONE_NORMAL is directly mapped by kernel into linear address space.
ZONE_HIGHMEM exists only in 32bit architecture. Because the kernel just has 1GB linear address space, thus could not map physical memory above 1GB directly where this very part is called ZONE_HIGHMEM.
To manage the memory of ZONE_HIGHMEM, the kernel leaves a region (typically 128MB) to complete this task.

For each physical page frame, thers is an associated `struct page`.
There is a global array holding all `struct page` called `mem_map`, which indexed by `pfn`, the page frame number.
For UMA system, the allocation of `mem_map` is straight forward.
For NUMA system, the global `mem_map` is just a virtual array. The actual struct pages are allocated in the corresponding node, pointed by `node->node_mem_map`.

* Kernel and user process sharing the same address space, avoids the TLB flush everytime when doing syscall. 
* With CONFIG_SLAB_DEBUG enabled, slab allocator marks either end of a object. If the marker(red zoning) is disturbed, a bug is reported. One object in free stat being filled with the pattern 0x5A(poisoning), if a newly allocated object does not match this pattern, a bug is reported.  

# Conventions
* Header files tend to be included by source files. Thus, try not to include other unecessary header files in your ones, especially for the interface headers. Or, Not only would this increase the build time, but also would annoy your users. 

# To Be Added
* SystemTap
* Cache Coherence & Cache Consistency
* Memory Barriers
* Memory Models
