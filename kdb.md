# The Kernel Debugger (kdb)
KDB is a text-based debugger included in the linux kernel intended for debugging kernel problems and analyzing system states. Kdb is useful for quickly examining the state of a system.
To use kdb, the Linux kernel must be compiled with kdb support enabled. This often involves setting specific configuration options in the kernel (**CONFIG_KDB**, **CONFIG_DEBUG_KERNEL**, 
and others related to debugging features.

##  Entering kdb
### SysRq Trigger:
On systems where the Magic SysRq key is enabled, you can enter kdb by pressingg a skecific key combination. For example holding **Alt** + **SysRq** (often *Print Screen* on keyboards) + 
**g** can switch the system to kdb mode.

### Breakpoint or Panic:
The system might also automatically enter kdb in response to a kernel breakpoint or panic.

## Frequently used kdb commands
### bt or backtrace
Display the call stack of the current or a specific CPU. This is useful for understanding  the function call sequence leading up to a crash or breakpoint.
### md <address>
Display memory contents at a given address
### rd
Display the contens of the current CPU registers.
### ps
Show process status
### lsmod
List loaded kernel modules
### set
Set the value of a kernel variable.
### clear
Clear a breakpoint
### bl
List breakpoints
### go
Continue
### help
Print help information.
