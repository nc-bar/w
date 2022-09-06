# Introduction

An operating system is a program that controls the hardware resources in one
or more computers. It gives programmers a simpler interface to the hardware
and other programs. It implements concepts such as _processes_ or _files_ ; it
manages resources so the programmer doesn't need to.

The abstract interface implemented by an operating system helps in:

  * _Portability:_ a program can _open_ , _read_ , _write_ or _close_ a file without having to control the hardware or worrying what fileystem the file is in. The same can be said about protocols for communicating with devices, the details of some hardware (e.g. the speed rotation of a hard drive). 
  * _Security:_ the integrity of an user's data, the reliability of the services, etc., is easier when an operating system is in charge of managing processes, memory, file system access, user sessions, etc. 

The operating system consist of a _kernel_ and a set of tools for the user.
The kernel is the low level code which interacts directly with the hardware,
has full access to the hardware, etc. A kernel by itself is not very useful,
at least from the user perspective; a set of tools, called _system programs_ ,
are also considered to be part of the operating system. What is part of the
operating system and what isn't, is not very clear, e.g. is the compiler part
of the operating system? and a standard library? the editors?. It gets more
confusing in the case of microkernels, exokernels, etc.

Here, the term operating system is used interchageably with kernel.

#### Computer hardware

    
    
    	+--------+           +-----+         +-----+
    	| memory | <-------> | bus | <-----> | I/O |
    	+--------+           +-----+         +-----+
    	                        ^
    	                        |
    	                   +---------+
    	                   | cpu1..n |
    	                   +---------+
    

This simplified model applies to most computers used everyday.

A _bus_ is a pathway where information can be sent and received. They carry
data between the memory, the devices, and the cpu's. Of course, there's
actually several buses, such as the internal buses connecting the components
of the cpus, the one between the memory and the cpu's, and others. Different
buses usually have different technologies and protocols, due to their design
constraints: does it need to be fast? carry a lot of data? or be resistant to
errors?. When the cpu needs to talk to I/O devices to carry some operation,
such as painting a pixel in a monitor or print a document, the data will flow
from the cpu, to the bus into the device (of course, the opposite flow is also
possible). Given that several elements of the system access the bus
concurrently some protocol is needed to make sure the information doesn't get
corrupted.

The I/O devices are the physical interface between the computer and the
outside world; the monitor, printer, keyboard, mouse are all I/O devices.
Typically, each device is connected to a controller and the operating system
is in charge of knowing the exact operations the device is capable of and how
to induce them, by writing bytes in their control and data registers. Device
_drivers_ are operating system's components in charge of talking to some
device or class of devices.

A program is a list of instructions the cpu understands. These instructions
are fetched from _memory_ and executed by the _cpu_. In a very simplistic way
this is all the cpu does:

    
    
    	1. fetch next instruction
    	2. execute instruction
    	3. go to 1
    

The main memory (ram) is divided into cells, each one with an address.

An _interruption_ is an asynchronous calling for the processor's attention. It
causes the cpu to jump to an _interrupt handler_ , which is an operating
system routine (another list of instructions stored in memory).

In most modern cpus, both hardware and software are able to interrupt the
processor.

Typical examples of interruptions (or traps) are:

  * a device interrupts the processor to communicate it finished some operation 
  * a program issues an interrupt to do some low low level task (implemented as part of the operating system) 

In the context of operating systems, one of the most important interruptions
is launched by the internal clock. A circuit sends electronic pulses in fixed
intervals of time, causing the cpu to interrupt it's normal cycle and execute
the interrupt handler. This address can be set by the programmer (using a
table of function pointers) or is fixed by the computer architecture. The list
of instructions executed by the processor while interrupted are called
interruption routine, interrupt handler or interrupt service routine. The last
instruction of these routines send the processor back to the last instruction
before the interruption.

The same mechanism can be used by I/O devices to communicate with the cpu:
when they require the processor intervention, they send an interrupt and wait
for a response. This is usually a better way of doing it, instead of having
the processor polling the resource waiting for something to happen. Also,
programmers can cause these interruptions (by using system calls or by causing
an exception or a trap, which are similar concepts) to interact with the
hardware or processes safely through the operating system.

Modern cpu's offer several mechanisms for protecting a _multiprogramming
system_ , i.e a system with several programs running concurrently. One of the
most important of all is the concept of _processor modes_. Some architectures,
such as amd64, have at least two modes: _kernel mode_ and _user mode_. The
first one is a processor state in which all resources of the cpu architecture
are accessible. It's in this mode that the kernel runs, because it has access
to all memory addresses, instructions, etc. The other mode is the user mode.
In it, programs have a set of instructions and resources they can use. If a
process breaks the rules set by the architecture, it triggers an exception in
the processor and a context switch occurs, i.e. the cpu goes from user mode to
kernel mode, and a routine defined in the kernel gets executed, usually
killing that process.

The program below uses a protected instruction. _[nasm](www.nasm.eu)_ can
compile the program and _ld_ links it, producing the executable:

    
    
    	; adapted from https://cs.lmu.edu/~ray/notes/nasmtutorial/
    
    	global _start
    
    	section   .text
    	_start:	
    		mov  rax, 1                       ; system call for write
    		mov       rdi, 1                  ; file handle 1 is stdout
    		mov       rsi, message            ; address of string to output
    		mov       rdx, 13                 ; number of bytes
    		syscall                           ; invoke operating system to do the write
    		wrmsr		                      ; privileged instruction, can only run in ring0
    		mov       rax, 60                 ; system call for exit
    		xor       rdi, rdi                ; exit code 0
    		syscall                           ; invoke operating system to exit
    
    	section   .data
    	message: db  "Hello, World", 10	  ; note the newline at the end
    

The process (i.e. a running program) throws and error, most likely
_Segmentation Fault_. To execute the instruction _wrmsr_ the process needs
kernel priviledge (in other words, the processor has to be in kernel mode).

This is an example of a program causing an exception (after the cpu fetches
the instruction _wrmsr_ , an exception occurs causing the cpu to move the
fetching target to the appropriate exception handling routine and ultimately
causing the operating system to kill the process).

The _amd64_ architecture refers to kernel mode as _ring0_. The operating
system creates the process and puts the processor in user mode; when executing
_wrmsr_ , the processor launches an exception and jumps to a routine in
memory, defined in the kernel, which handles the error.

In order to use those restricted resources (reading or writing to or from
disk, communication between processes, etc.), user mode programs have to ask
the operating system to do it in their behalf, through carefully designed
interfaces known as _system calls_ ; For example the _write_ system call is
used to write into a file.

    
    
    	user mode
    
    
    	process --> syscall(write)                                    process
    	---------------------------+--------------------------------^---------
    	                           |                                |
    	                           +---> kernel routine ---> ret ---+
    	kernel mode
    

Operating systems use this separation to accomplish their main goal, which is
to give programmers a simpler interface to the messy, complicated low level
resources and to manage them in an efficient, secure way.

The processor fetches instructions from a medium of storage, or _memory_. This
memory, usually called main memory, is a relatively fast device which stores
process instructions and data. this memory is usualy called _RAM_ memory
(Random Access Memory). It is orders of magnitude faster than disks, but
orders of magnitude slower than cpus. To solve this problem, the processor
uses a _cache_ , that is, an intermediate, faster, storage unit which saves
recent results, so it's not necessary to fetch them again from memory until
they become invalid or need to be removed due to space constraints

The allocation an deallocation of chached objects. follows rules defined by
the particular processor organization; but it is almost always based in the
observation that processes tend to request memory addresses which are near
each other in space and time.

The storage capacity of a computer can be represented as pyramid in which data
stored in slower but abundant storage (cheaper) gets cached into more
expensive and faster, but less available (expensive) storage:

    
    
    	|
    	|            |
    	|            |                  |
    	| disks ---> | main memory ---> | cpu cache ---> | cpu registers
    	|            |                  |
    	|            |
    	|
    

Both main memory and the cache are volatile: after the power is shut down the
information is lost. Therefore programs must be saved into more permanent
storage, such as magnetic or solid state disks to persist data between
shutdowns. This is usually where the kernel resides: when the system boots, a
special program called the bootloader, saved in a special region of the disk,
loads the operating system kernel into memory and starts it's execution.

In multiprogrammed operating systems, several processes are using the memory
at the same time. This could create a lot of problems and security concerns,
so modern cpus have capabilities to help the operating system protect a
process' memory area. A solution to this problem is through the concept of
paging, in which physical memory are divided into contiguous intervals of
adresses of some fixed size, called pages, o the kernel can restrict the
memory a process can use. This method is called _Paging_ , and is a common way
to implement _virtual memory_ ,

In virtual memory, the operating system, using processor features, keeps a map
between memory pages and real memory addresses. A process is given a list of
virtual pages, which map to real memory. A virtual memory page can be moved to
secondary storage (a hard drive for example); this is called _swaping_.

When a page was moved from memory to disk, i.e. swaped, and a process tries to
read it, a _page fault_ error is triggered by the processor. A routine is
called automatically to handle the error. This routine is part of the
operating system; using the virtual memory map, it loads the requested page
into main memory, and resumes the process execution.

The virtual memory can be implemented in a per process basis. The memory is
presented to the process s if it were the only one in execution. giving the
appearance to each process that they have almost the entire memory to
themselves. Others mechanisms than paging, suc as segmentation, exist, but are
not often used.

Much of the operating system's interaction with processes is trying to protect
them from each other.

####  Operating System Services

  * _Process management_ : the operating system must manage several processes executing in the same environment, keeping the system as a whole running securely and efficiently. This processes sometimes need to communicate with each other, so it must also provide the mechanisms to achieve _inter-process communications_ and the resources to do it safely and correctly. It's possible that some processes behave badly (intentionally or not) so there must be a way to control them.
  * _Memory Management_ : the memory is a vital resource of a computer system, and therefore, must be managed efficiently and securely. This can be done by the operating system in a transparent way, without the knowledge of the application programmer, using the tools the processor provides for such a purpose.
  * _I/O_ : to be useful a computer needs to take in input from outside it's system and return an output to the outside world. As explained earlier, to help programmers to focus on solving new problems, and given their complicated interfaces, hardware is managed by the operating system. Among this devices used to interface with the real world, the storage devices such as disks, are specially important. The operating system must provide simple interfaces to work with them. 
  * _Security_ : the operating system must enforce rules so that a multi-user and multiprogrammed system is usable and secure.

