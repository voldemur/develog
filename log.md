Tools
===============

oprofile
=======
```bash
$ opcontrol --init
$ opcontrol --status
$ opcontrol --start
$ opcontrol --list-events
$ opcontrol --reset
$ opcontrol --dump
$ opreport image:path/to/your/exe -l
$ opannotate image:path/to/your/exe -s
```


Tips
===============
* Do not use too many parameters on your functions.
  Because on IA64 architecture, the arguments are passed through registers.
  When there are too many of them, arguments have to be pushed to stack, which is
  usually maintained by memory. That would slow your function calling down.
