### Process Management

#### Multiprogramming

Most modern operating systems have multiprogramming, meaning that several
programs may be running concurrently, sharing hardware resources. This kinds
of systems are also the most interesting ones in relation to processes.

In a multiprogrammed system, several programs in execution have to use the
shared resources correctly, without rendering the system unusable; also, it's
possible to make this units of execution talk to each other, achieving Inter
Process Communication (IPC). The cpu or cpu's can be used more fully, thanks
to the non blocking nature of this kinds of systems: a process waiting for the
completion of an I/O transaction can be temporarily ignored by the processor,
and another one executed in its place.

#### Introduction to processes

A process is a program in execution. The same program can have several
processes executing its instructions. To be useful a process needs resources
such as CPU use, I/O devices, memory, etc. To maintain processes running
concurrently, the operating system needs to provide ways to create, stop and
administer the resources they use and the processes themselves.

In most operating systems, each new process gets a unique identifier, called
the PID. This identifier is usually a positive integer.

A program ends up as a process after several steps. First the program source
is transformed into an executable file (in the case of an interpreted language
like python, this is different, in that the interpreter is the executable, and
not the program) which holds the information needed for its execution (static
libraries linked to it, references for dynamic libraries to be loaded, global
variables, executable code, etc.) usually in a standard format such as elf in
Linux or exe in Windows. Then a system call is used to take this file and
transform it into a process, that is, the operating system creates internal
structures to represent the running program; this also entails allocating the
resources needed for a new process.

Let's say I have the following C program:

    
    
    #include <stdio.h>
    
    int main() {
        printf("Hello, world\n");
    }
    

in the file `hello.c`. Then, using a C compiler and linker we can produce an
executable:

    
    
    $ gcc hello.c -o hello
    

The generated executable's format is elf in Linux. In a shell, we can execute
it:

    
    
    $ ./hello
    Hello, world
    

#### Process states

During it's lifetime, a process changes its state. A state is one of the
possible activities a process can be doing:

  * _New:_ the process is being created. In this stage the structures used to represent a program in the kernel are created as well as the context needed for it to run correctly.
  * _Running:_ the cpu is executing the instructions defined by the program associated with the process.
  * _Waiting:_ the process is waiting some event so it can be a candidate for execution again.
  * _Ready:_ the process can execute, but is waiting to be assigned a processor by the operating system.
  * _Terminated:_ The process stopped executing.

This is of course a template, and some operating systems might have different
states. For example, an operating system which can only run one process might
not have a ready queue.

A state machine is a very appropriate model for the lifetime of a process:

    
    
                                os assigns              process stops
          creation              a processor               executing
    NEW -------------> READY -----------------> RUNNING ---------------> TERMINATED
                       ^  ^                        |  |
                       |  |      interruption      |  |
                       |  |(syscall, hardware, etc)|  |
                       |  --------------------------  |
                       |                              |
                       |                              |
                       |-----------WAITING <-----------
                    event                          waiting for an event (I/O, etc)
    

Let's see an example in Linux. The following program, when launched, will
start in the NEW state, then when Linux finishes creating the process, it will
pass to the READY state, then when it's assigned a processor will go to the
RUNNING state. Given that Linux can preemptively move a process from RUNNING
to READY, the process might bounce between those states, but lets ignore that.
Then when the user is asked to enter a character, the process will then block,
going to the WAITING state while waiting until the I/O event of the user
pressing a key (and enter) will move the process to the READY queue again.

    
    
    #include <stdio.h>
    
    int main() {
        int c;
        printf("enter one character (program will go to WAITING state):\n");
    
        c = getchar();
        /* WAITING */
        if (c == EOF) {
            perror("something weird happened: ");
            return(-1);
        }
    
        printf("you entered: %c\n", c);
    }
    

you can compile and execute:

    
    
    $ gcc main.c -o main
    $ ./main
    enter one character (program will go to WAITING state):
    a
    you entered: a
    

#### The tree of processes

Operating systems usually expos several system calls related to processes. One
important operation the user might want to do is the creation of processes.

A typical example of a tool used to create other processes is the shell. This
program instructs the operating system to create a process, passing a
program's path as an argument.

In Linux it takes 2 system calls to create a process, `fork` and `exec`. The
fork system call creates a process which is a copy of its parent and shares
the same code and other attributes, while `exec` replaces the code of the
calling process for the one in the argument and the `wait` system call makes
the program wait until the child process terminates. So:

    
    
    #include <sys/wait.h>
    #include <sys/types.h>
    #include <unistd.h>
    #include <stdio.h>
    
    int main() {
        pid_t pid;
        pid = fork();
    
        /* Error in fork() */
        if (pid < 0) {
            perror("fork() error: ");
            return(-1);
        }
    
        if (pid == 0) {
            char *args[] = {"/bin/ls", NULL};
            execvp(args[0], args);
        } else {
            wait(NULL);
        }
    
        return 0;
    }
    

is a program which creates a copy of itself using `fork` and then replaces
it's own code with the program `/bin/ls/` using the `execvp` system call.

When a process creates another one, they join a tree like relationship between
themselves and all others; usually, the one on top is called the parent and
those it creates the children:

    
    
              PID1 (parent)
                  |
                  |
            -------------
            |           |
    (child) PID10       PID20 (child)
            |           |
            |           |
           ...         ...
    

here PID10 and PID20 are children or sub-processes of PID1.

In some operating systems, specially those influenced by Unix, there's a
process, usually called init, which receives the pid 1 and is the parent of
all the other processes in the system.

The init process performs some administrative tasks and is in charge of
launching some daemons, processes running in the background constantly. For
example, when a machine is used as a web server, it's useful that every time
it boots it launches a http server, rather that needing someone at a terminal
to execute the server every time. This is achieved by telling the init process
to execute some program in the background.

There isn't a standard defined for what the init system should do, and so
there's a lot of different programs: systemd,
[runit](https://github.com/vulk/runit/blob/master/src/runit.c), OpenRC, etc.
Of course, any program could be init, but not all can prepare a system to be
useful.

When a process terminates, some operating systems also terminate all it's sub-
processes, while others make the sub-processes direct descendants of the init
process. Process termination can be done by the process itself using a system
call usually named `exit`. Some operating systems provide mechanisms to kill
other programs, maybe restricting which programs can kill others.

In Linux, the parent process can obtain information about the exit status of
its children. The `exit` system call takes an integer as a parameter which
represents the exit status; by convention 0 means correct and non zero means
error so they can be given an unique code.

In the following program, 4 processes are created and loop forever, while the
main process uses the `waitpid` system call to wait until each of the
processes are killed.

    
    
    #include <sys/wait.h>
    #include <sys/types.h>
    #include <unistd.h>
    #include <stdio.h>
    
    #define PROC_MAX 4
    
    int main() {
        pid_t me, pid[PROC_MAX];
        int i, wstatus;
    
        me = getpid();
        printf("I'm %d, don't delete me, I have children!\n", me);
    
        for (i = 0; i < PROC_MAX; i++) {
            pid[i] = fork();
    
            /* Error in fork() */
            if (pid[i] < 0) {
                perror("fork() error: ");
                continue;
            }
    
            if (pid[i] == 0) {
                while (1) {
                    sleep(1);
                }
            }
        }
    
        for (i = 0; i < PROC_MAX; i++) {
            if (pid[i] < 0)
                continue;
    
            waitpid(pid[i], &wstatus, 0);
            if (WIFEXITED(wstatus) || WIFSIGNALED(wstatus)) {
                printf("%d killed!\n", pid[i]);
            }
        }
    }
    

Given that the 4 created processes never end, to watch the program work we'll
use the `kill` Linux utility to kill the sub-processes, unlocking the main
process and letting it end. Assuming the program is in a file called main, we
can open two terminals and in one write

    
    
    $ gcc main.c -o main
    $ ./main
    I'm 10581, don't delete me, I have children!
    

and in the other one, we use the `ps` program to figure out the children pid's
and send a signal to stop them:

    
    
    $ ps a | grep main
    10581 pts/1    S+     0:00 ./main
    10582 pts/1    S+     0:00 ./main
    10583 pts/1    S+     0:00 ./main
    10584 pts/1    S+     0:00 ./main
    10585 pts/1    S+     0:00 ./main
    10592 pts/2    S+     0:00 grep main
    
    $ kill 10582
    $ kill 10583
    $ kill 10584
    $ kill 10585
    

after all the sub-processes were stopped, you will see in the first terminal:

    
    
    $ gcc main.c -o main
    $ ./main
    I'm 10581, don't delete me, I have children!
    10582 killed!
    10583 killed!
    10584 killed!
    10585 killed!
    

#### A process in memory

There's at least two important operations the kernel does to take a program
and spawn a process based on it. Setting some structures needed to keep meta
information about the processes like their state (RUNNING, WAITING, etc.),
memory layout information (each process might have its own virtual memory
mapping using pages), running time to help the scheduler, and a lot more. Also
the code, or stream of instructions, needs to be laid out in memory in a way
that the processor architecture understands and can execute. This not only
means arranging the code in a certain way in memory, but also loading dynamic
libraries, setting up the correct values in the processor registers (defining
the stack for example), etc.

The way a process is laid down in memory varies between architectures, but a
good template is:

    
    
        higher memory addresses
    
        /\/\/\/\/\/\/\/\/\
        |                |
        |                |
        |----------------|
        |   stack        | function parameters, local variables
        |                |
        |----------------|
        |          ||grow|
        |/\        \/    |
        |||grow          |
        |----------------|
        |   heap         | dynamic memory
        |                |
        |----------------|
        |   data         | global variables
        |                |
        |----------------|
        |   text         | program instructions
        |                |
        \/\/\/\/\/\/\/\/\/
    
    lower memory addresses
    

The stack is used to hold temporary variables and function parameters; the
heap is used to allocate dynamic memory, i.e, the memory requested by the
programmer during runtime; the data segment is used to hold global variables;
the text segment is the list of instructions defined by the programmer (or the
compiler most likely).

If the kernel wants to begin the execution of a program, it needs to point the
cpu's instruction pointer to the first instruction in the text segment, point
the stack register to the correct memory address, and a lot more.

The operating system needs a way to keep track of the
different processes, the resources they use, and the state they are in. The
structure used to represent a process is called the Process Control Block
(PCB) or Task Control Block (TCB). This structure is defined by the kernel and
it saves important administrative information about processes: process state,
process id, program counter, registers, memory information, etc.

A PCB may contain this information:

    
    
    state: NEW | READY | RUNNING | WAITING | TERMINATED | ...
    priority: scheduler priority
    running time: total running time used by the scheduler to set the priorities
    PID: process identifier
    program counter: next instruction to execute when changing to RUNNING
    registers: the state of the registers before a switch from RUNNING
    memory information: paging tables, etc.
    filed: list of open files
    

probably implemented in a `struct` if the kernel is written in C.

#### Process Scheduling

The scheduler is the part of the operating system that decides which process
gets cpu time next. It's composed of an algorithm making the decision and the
dispatcher in charge of instructing the cpu to execute the chosen process.

By switching the processes in execution the system can use the cpu more fully.
To provide an example, let's assume a computer with only one cpu and two
processes and the simplest scheduler. A process is allowed to run until it
exits or makes an I/O request, and while the process waits for the I/O
termination, the cpu is idle. Graphically:

    
    
    process1       process2         CPU
    +--------+     +--------+     +--------+
    |        |     |        |     |        |
    |cpu time|     | ready  |     | in use |
    |        |     |        |     |        |
    +--------+     |        |     +--------+
    +--------+     |        |     +--------+
    |        |     |        |     |        |
    |        |     |        |     |        |
    |        |     |        |     |        |
    |I/O wait|     |        |     |  idle  |
    |        |     |        |     |        |
    |        |     |        |     |        |
    |        |     |        |     |        |
    +--------+     |        |     +--------+
    +--------+     |        |     +--------+
    |        |     |        |     |        |
    |cpu time|     |        |     | in use |
    |        |     |        |     |        |
    +--------+     +--------+     +--------+
    

The cpu is idle a long time in the previous example. In that time, a more
sophisticated scheduler would give the cpu to another process. For example:

    
    
     process1       process2         CPU
    +--------+     +--------+     +--------+
    |        |     |        |     |        |
    |running |     | ready  |     | in use |
    |        |     |        |     |        |
    +--------+     +--------+     +--------+
    +--------+     +--------+     +--------+
    |        |     |        |     |        |
    |        |     |        |     |        |
    |        |     |        |     |        |
    |I/O wait|     |running |     | in use |
    |        |     |        |     |        |
    |        |     |        |     |        |
    |        |     |        |     |        |
    |        |     +--------+     +--------+
    |        |     +--------+     +--------+
    |        |     |        |     |        |
    |        |     |        |     | idle   |
    |        |     |        |     |        |
    +--------+     | exited |     +--------+
    +--------+     |        |     +--------+
    |        |     |        |     |        |
    |running |     |        |     | in use |
    |        |     |        |     |        |
    +--------+     +--------+     +--------+
    

In this contrived example, we see that the cpu is more efficiently utilized.
This scheduler better exploits the system resources, showing the advantage of
a multiprogrammed operating system.

While a program executes it alternates between cpu bursts and I/O bursts. A
cpu burst is a time slice in which the cpu is executing instructions from a
process while an I/O burst is when a process is waiting for data to be
transferred from or to a device.

There's two types of schedulers, preemptives and non preemptives (also called
non cooperatives). The non preemptives are the ones which carry out scheduling
policy when a process is in the waiting state or when one terminates. The
preemptive ones can decide to change the process running in other times, for
example when an interrupt happens.

Preemptive ones usually need some hardware capabilities, such as a clock
sending periodical signals causing an interruptions.

There's a lot of different scheduling algorithms which are appropriate for
different uses and kind of systems.

Each time a process gets from the running state to the ready or waiting state,
a context switch occurs. The scheduler needs to save the state of the current
process, which includes the general purpose register values, the instruction
pointer, into memory. This takes time which is not really useful, because it's
just bookkeeping to make the system function.

#### Scheduler algorithms

The purpose of the system determines what properties the scheduler has to
have. Some systems are made to optimize the amount of processes that finish
per unit of time, others need to be responsive of I/O events.

Some metrics are used to measure an operating system scheduler properties. The
most used are:

  * _Turnaround time_. The time between a process first enters the ready state until it finishes.
  * _Response time_. The time it takes an interactive program to start outputting a response to a command. This is an important metric for interactive systems.
  * _Waiting time_. Time the processes spend in the ready queue.
  * _Throughput_. The amount of processes completed per unit of time.
  * _Cpu use_. The percentage of time the cpu was executing instructions.

As mentioned before there's a lot of different ways a system can choose what
is the next process who gets an idle cpu.

#### First-Come, First-Serve (FCFS)

This algorithm is one of the simplest ways an scheduler can function. It
consists of a queue of processes ordered by the time they enter the ready
state, and when a cpu is vacant, the top of the queue is put there.

Let's see an example. Processes p1 and p2 enter the ready state

    
    
    executing
       ^
       |
    +----+   +----+
    | p1 |<--| p2 |
    +----+   +----+
    

Process p3 enters the ready state, goes to the end of the queue

    
    
    executing
       ^
       |
    +----+   +----+   +----+
    | p1 |<--| p2 |<--| p3 |
    +----+   +----+   +----+
    

Process p1 finished and a cpu is free to use, the next process in the queue
enters the cpu

    
    
    executing
       ^
       |
    +----+   +----+
    | p2 |<--| p3 |
    +----+   +----+
    

One of the advantages of this algorithm is the simplicity of the code and data
structures involved. On the other side, it has several disadvantages. An
important one is that the algorithm is non preemptive, so a process in a cpu
burst is not taken out of the cpu until it performs an I/O operation or it
finished, so other processes have to wait in ready state. This algorithm is
also not the best choice for interactive systems which need a short time
between the user launching a command and getting some kind of response; this
happens because a CPU intensive process could prevent those processes which
need interactivity.

#### Shortest job first (SJF)

SJF is specially useful for [batch
systems](https://en.wikipedia.org/wiki/Batch_processing), which execute a set
of programs that don't need user interaction. To use it the running time or
the cpu bursts of each process need to be known, or at least an accurate
approximation is needed. When a cpu is available the algorithm chooses the
process with the smallest cpu burst.

The fact that the bursts times need to be known is a big disadvantage, it has
a couple of interesting properties. For example, it's possible to
mathematically prove that this scheduler minimizes the average waiting time.

#### Round Robin

Round Robin is a preemptive scheduler which gives the same cpu time to all the
processes before preemptively deallocating it from the cpu. So, it simply goes
through the ready (circular) queue and gives the same maximum time to all of
them before taking it out of the cpu. That fixed amount of time is called a
quantum. Doing a [Gantt chart](https://en.wikipedia.org/wiki/Gantt_chart);
let's assume we have 3 processes with a running time of 4 milliseconds without
using I/O, with a quantum of 2 milliseconds:

    
    
    +-----------------------------+
    | p1 | p2 | p3 | p1 | p2 | p3 |
    +-----------------------------+
    0    2    4    6    8   10   12
    

To function, this scheduler needs a clock producing regular interruptions so
it can keep track of time and fields in the Process Control Block to save the
running time.

The most important parameter is the quantum. If the quantum is too big, then
the system responsivity goes down because a process could use too much cpu
time at once, for example during a cpu burst. If it's too small, then the time
spent in context switching is too much in proportion with the amount of useful
cpu execution time. It's interest to note that if the quantum tens to
infinity, then RR is FCFS.

#### Multilevel feedback queue

Just like RR keeps processes in a queue and assigns them a time slice to use
before being deallocated. But Multilevel Feedback Queue has several queue's
each with a different priority and quantum. First, there's a priority among
the queues, when a queue of greater priority runs out of ready processes, the
scheduler goes to the next.

Processes can be moved between the queues. Usually, I/O bound processes get
into the queues with greater priority and the ones which have need too much
cpu are moved to the bottom.

#### Inter process communication (IPC)

The processes in a running system are not completely isolated, the operating
system provides mechanisms for them to interact; those mechanisms are called
Inter Process Communication (IPC). Using IPC processes can exchange
information, distribute computation and cooperate with each other.

IPC mechanisms can be separated into message passing and shared memory. The
first one works by several processes sending data to each other; the shared
memory works by defining a piece of contiguous memory as shared, that is, some
processes can read and write to it.

The best way to explain them is to show different implementations.

### POSIX shared memory

[POSIX](https://en.wikipedia.org/wiki/POSIX) is a family of standards created
by the [IEEE](https://en.wikipedia.org/wiki/POSIX). It consists of several
api's for services a system has to implement. Some are related to operating
system features, syscalls, C libraries or tools and it aims to provide a
portable interface for programmers. POSIX shared memory is a standard api
implementing shared memory.

The shared memory IPC method is very fast because communication happens
through reading and writing into some memory area, without the need to use
system calls (after the shared memory region was established). The main
disadvantage is that it requires more careful programming to avoid several
programs from interfering with each other: if several programs try to write to
the same memory address, the end result could be garbage; so it requires
synchronization.

Shared memory is implemented using the following functions:

  * `int shm_open(const char *name, int oflag, mode_t mode);`
  * `int shm_unlink(const char *name);`

The first, `shm_open`, creates or opens a POSIX shared memory object. This
object can then be mapped into other processes' own memory using `mmap`, and
finally exchange information between each other. The second function,
`shm_unlink`, removes an object created by `shm_unlink`.

In order to do something complicated using shared memory, it's necessary to
understand come synchronization concepts, like semaphores or atomic variables;
since I haven't discussed them yet, the following example will be very simple,
just to demonstrate how it works.

The example consists of two programs. The client takes user input and writes
it to the shared buffer, while the server constantly prints the contents of
said buffer each second. This way we don't need to be worried about
synchronization since we don't really expect any kind of correct result.

The client sends a string to the server:

    
    
    #include <stdlib.h>
    #include <string.h>
    #include <stdio.h>
    #include <sys/mman.h>
    #include <fcntl.h>  /* for O_* constants */
    #include <semaphore.h>
    
    #define MUTEX_CLIENT "/mutex-client-posix-shared-memory"
    
    int main() {
        const char *msg = "Message from client!";
        int fd;
        void *shm;
        sem_t *mutex_client;
    
        if ((fd = shm_open("/testingposixsharedmemory", O_CREAT | O_RDWR, 0666)) == -1) {
            perror("Couldn't open or create posix shared memory object");
            exit(1);
        }
    
        if ((shm = mmap(NULL, sizeof(msg), PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0)) == MAP_FAILED) {
            perror("Couldn't map shared memory");
            exit(1);
        }
    
        if ((mutex_client = sem_open(MUTEX_CLIENT, 0,0,0)) == SEM_FAILED) {
            perror ("No mutex");
            exit(1);
        }
    
        strcpy(shm, msg);
    
        sem_post(mutex_client);
    
        if (munmap(shm, sizeof(msg))) {
            perror("Couldn't unmap");
            exit(1);
        }
    
        if (sem_close(mutex_client)) {
            perror("Closing mutex");
            exit(1);
        }
    
        exit(0);
    }
    

The server receives a string from the client, prints it and exits:

    
    
    #include <stdlib.h>
    #include <string.h>
    #include <stdio.h>
    #include <unistd.h>
    #include <sys/mman.h>
    #include <fcntl.h>  /* for O_* constants */
    #include <semaphore.h>
    
    #define MUTEX_CLIENT "/mutex-client-posix-shared-memory"
    #define SHM_NAME "/testingposixsharedmemory"
    #define MSG_LEN 30
    
    int main() {
        int fd;
        void *shm;
        sem_t *mutex_client;
    
        if ((fd = shm_open(SHM_NAME, O_CREAT | O_RDWR, 0666)) == -1) {
            perror("Couldn't open or create posix shared memory object");
            exit(1);
        }
    
        if (ftruncate(fd, MSG_LEN) == -1) {
            perror ("ftruncate");
            exit(1);
        }
    
        if ((shm = mmap(NULL, MSG_LEN, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0)) == MAP_FAILED) {
            perror("Couldn't map shared memory");
            exit(1);
        }
    
        if ((mutex_client = sem_open(MUTEX_CLIENT, O_CREAT, 0660, 0)) == SEM_FAILED) {
            perror ("No mutex");
            exit(1);
        }
    
        sem_wait(mutex_client);
    
        printf("%s\n", (char *) shm);
    
        if (sem_close(mutex_client)) {
            perror("Closing mutex");
        exit(1);
        }
    
        if (munmap(shm, MSG_LEN)) {
            perror("Couldn't unmap");
            exit(1);
        }
    
        shm_unlink(SHM_NAME);
    
        exit(0);
    }
    

In Linux, you can compile and execute using:

    
    
    gcc server.c -o server -lrt -lpthread
    gcc client.c -o client -lrt -lpthread
    

Then in two separate terminals:

    
    
    ./server
    

and then

    
    
    ./client
    

As mentioned earlier, I didn't include much synchronization logic because it
wasn't explained yet. For example, I'm assuming that the client asks the mutex
after it was created, which might not be true: recall from the scheduling
section that there's the possibility that the scheduler is going to take the
server out of the cpu and then run the client before the mutex was created.
But, it's very unlikely and the example is enough to show two processes
interacting using shared memory. If you don't know what a mutex is, you can
just skip this paragraph.

### Unix pipes

Pipes are a unidirectional mechanism for IPC. It consist of an object with two
endpoints, one for writing and the other for reading:

    
    
                              pipe                           pipe
                          ___________                   ___________
    input -> process1 -> )___________) -> process 2 -> )___________) -> process3 ->...-> output
    

Pipes are a very interesting concept originated by [Douglas
McIlroy](https://en.wikipedia.org/wiki/Douglas_McIlroy) because they help
apply the Unix philosophy of software design:

  * Write programs that do one thing and do it well.
  * Write programs to work together.
  * Write programs to handle text streams, because that is a universal interface.

The Unix philosophy is about making software composable. Pipes aid in this
design by making it possible to stitch one program's output to another
program's input. For example, Linux shells usually provide the programmer with
the operator `|` which creates a pipe and redirects the standard output (the
output that eventually gets printed in the terminal) to the standard input
(usually the input from the keyboard) of another program, so we can do things
like:

    
    
    ls -l | grep ".*\.c"
    

in which `ls -l` returns a list of files in the current directory and `grep
.c` filters out all the lines not ending in '.c'. Without the ability to
compose programs in this manner, we would need to add to each command a
feature to filter its output; but using pipes, we can use a program
specifically designed to filter lines of texts and compose it with others, in
essence programming a lot of tools to do one thing well and then using hem
together.

The Linux kernel provides the `pipe` system call in order to create pipes.
When the shell gets the input `ls -l | grep ".*\.c"` it creates a pipe and
then using `fork` (pipes are inherited) it redirects the standard output of
one child to the standard input of the other, and then they call `exec` to
execute `ls` and `grep` respectively:

    
    
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    
    #define STDOUT 1
    #define STDIN 0
    
    int main(void) {
        int fd[2];
    
        pipe(fd);
        if (pipe(fd) == -1) {
            perror("dup");
            exit(1);
        }
    
        if (!fork()) {
            close(STDOUT);
            if (dup(fd[1]) == -1) {
                perror("dup");
                exit(1);
            }
            close(fd[0]);
            execlp("ls", "ls", "-l", NULL);
        } else {
            close(STDIN);
            if (dup(fd[0]) == -1) {
                perror("dup");
                exit(1);
            }
    
            close(fd[1]);
            execlp("grep", "grep", ".*\\.c", NULL);
        }
    
        return 0;
    }
    

Here, the function `dup` is used to create a copy of the file descriptor
passed as a parameter. The function `pipe` creates two file descriptors and
saves them into the array parameter; the first (in this case `fd[0]`) is the
read end, and the other the write end. When data is written into `fd[0]` it
comes out of `fd[1]`.

Of course, pipes are not just used by the shell, but that application is the
most visible. Every program can use pipes, because they are a service provided
by the operating system.

#### Signals

Another way to make programs interact with each other is through the use of
signals. A signal is similar to a notification, in that it's an asynchronous
calling for attention, in this case, of a process. Each signal can be one of a
set of predefined types. After receiving a signal, the process' signal handler
for the appropriate signal type is executed; in this way, a signal is similar
to an interrupt, and the signal handler is similar to the interrupt handler.

A program can define its own signal handler for almost every signal type,
otherwise the default signal handler is used.

The `kill` system call is used to send signals:

    
    
    int kill(pid_t pid, int sig);
    

It sends the signal of code `id` to the process whose process id is `pid`.
Actually is a bit more complicated, the full details are in the `man` pages.

Linux provides a program using that system call wrapper to send processes
signals from the shell. The tool is also named `kill` and it's used like this:

    
    
    kill -s signal_code pid
    

That command will send the signal `signal_code` to the process with `pid` pid.
Each signal is identified by a number.

I will use that command to show a program defining its own signal handler. In
order to do that, we need to tell the kernel what function should be called
when certain signals arrive; the system call used to do that is:

    
    
    int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
    

Here we are telling the kernel to call the function defined in `act`. The
definition of that structure is:

    
    
    struct sigaction {
               void     (*sa_handler)(int);
               void     (*sa_sigaction)(int, siginfo_t *, void *);
               sigset_t   sa_mask;
               int        sa_flags;
               void     (*sa_restorer)(void);
           };
    

where `sa_handler` is the the signal handler (although you can change that by
setting `SA_SIGINFO` in `sa_flags`, then `sa_sigaction` would be the handler).
`sa_mask` tells the kernel which signals to block while this one is being
handled) and `sa_flags` modifies the behavior of the signal.

The following program prints to the terminal every time a signal arrives:

    
    
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <signal.h>
    
    #define STDOUT 0
    
    void handle_sigint(int sig) {
        write(STDOUT, "SIGINT ARRIVED\n", 15);
    }
    
    void handle_sighup(int sig) {
        write(STDOUT, "SIGHUP ARRIVED\n", 15);
    }
    
    
    int main(void) {
        struct sigaction sa_sigint, sa_sighup;
    
        /* print pid to make it simpler  */
        fprintf(stderr, "%d\n", (int) getpid());
    
        sa_sigint.sa_handler = handle_sigint;
        sa_sighup.sa_handler = handle_sighup;
        sa_sigint.sa_flags = sa_sighup.sa_flags = SA_RESTART;
        sigemptyset(&sa_sigint.sa_mask);
        sigemptyset(&sa_sighup.sa_mask);
    
        if (sigaction(SIGINT, &sa_sigint, NULL) == -1) {
            perror("sigaction sigint");
            exit(1);
        }
    
        if (sigaction(SIGHUP, &sa_sighup, NULL) == -1) {
            perror("sigaction sigint");
            exit(1);
        }
    
        while (1) {
            sleep(1);
        }
    
        exit(0);
    }
    

To run the example, compile and execute. The program will output its pid; with
that information you can use the `kill` utility to send signals of type
`SIGINT` or `SIGHUP`.

An interesting point is that if you press ctrl+c, which usually lets you
terminate a process, now it just prints SIGINT ARRIVED; this is because if you
are using bash, after pressing ctrl+c, bash sends a `SIGINT` to the process in
the terminal foreground.

Some signals can't be handled by user defined functions, for example `SIGSTOP`
and `SIGKILL`.

#### Unix sockets

Unix sockets allow bidirectional communication between processes. This means
that, if there's two processes p1 and p2 communicating using sockets, then p1
can send a message to p2 and p2 can send a message to p1, in both cases using
the same object. In other words, if pipes are one way streets, sockets are two
way streets.

These objects are represented as files in the file system to which processes
can connect to. Usually, a process will act as a server, creating the socket
and responding to requests for clients. The client processes will, knowing the
socket filename, open a connection.

Sockets can send messages with several underlying protocols, such as TCP and
UDP, but to keep things local, I'll discuss only the Unix sockets in which
several local processes talk to each other with the kernel being the
intermediary.

The services for achieving this type of IPC are exposed through the following
system calls:

    
    
    - int socket(int domain, int type, int protocol);
    - int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
    - int listen(int sockfd, int backlog);
    - int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
    - int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
    

The `socket` system call creates a socket of the specified type, that is,
using some underlying protocol and address space (ip address, file path,
etc.). `bind` gives a socket an effective address other processes can connect
to; it creates a file in the file system which represents the new socket in
the case of Unix sockets. Now, if the program is supposed to be a server,
`listen` is used to specify that the socket passed as a parameter will sit
idle and wait for other processes to connect to it using the `connect` system
call. When a passive socket gets an incoming connection, a call to `accept` is
used to finally establish the connection between processes; the return value
is, in case of success, a brand new socket file descriptor, so the new one can
be used for communication while the old one keeps listening for new incoming
connection. After the connection was successfully established, the usual
system calls for reading and writing are used: `read` and `write`.

The mechanism is a bit convoluted, so a graphic aid can be useful:

    
    
      SERVER                                 CLIENT
      ------                                 ------
    
    +--------+                             +--------+
    |socket()|                             |socket()|
    +--------+                             +--------+
        | socket created                        | socket created
        |                                       |
    +--------+                             +---------+
    | bind() |                             |connect()|
    +--------+                             +---------+
        | give the socket an address             socket got connected to another one
        |                                        now the processes can communicate
    +--------+
    |listen()|
    +--------+
        | socket defined as passive:
        | it will wait for connection
        | attempts
    +--------+
    |accept()|
    +--------+
      new socket created to
      represent the connection
    

Each socket represents an endpoint in a possibly bidirectional communication.
Messages written into one end appear in the other end (if everything went
well):

    
    
                     r/w    +------+         +------+   r/w
    server process <----->  |socket| <-----> |socket| <-----> client process
                            +------+         +------+
    

After the connection between two sockets is established, it's possible to
`read` and `write` the sockets to send or get information.

Let's see an example server:

    
    
    #include <stdlib.h>
    #include <stdio.h>
    #include <unistd.h>
    #include <string.h>
    #include <sys/types.h>
    #include <sys/socket.h>
    #include <sys/un.h>
    
    
    #define SOCKET_NAME "./socket"
    #define MAX_CONNECTIONS 100
    
    void serve(int fd) {
        int n;
        char buf[101];
    
        while ((n = read(fd, buf, sizeof(buf))) > 0) {
            printf("%s\n", buf);
            memset(buf, 0, sizeof(buf));
        }
    
        if (n == -1) {
            perror("read");
            exit(-1);
        } else if (n == 0) {
            printf("\n");
            close(fd);
        }
    }
    
    int main(void) {
        struct sockaddr_un addr, caddr;
        int fd, cfd;
        socklen_t caddr_size;
    
        if ((fd = socket(AF_UNIX, SOCK_STREAM, 0)) == -1) {
            perror("socket");
            exit(-1);
        }
    
        memset(&addr, 0, sizeof(struct sockaddr_un));
        addr.sun_family = AF_UNIX;
        strncpy(addr.sun_path, SOCKET_NAME, sizeof(addr.sun_path) - 1);
        unlink(SOCKET_NAME);
    
        if (bind(fd, (struct sockaddr *) &addr, strlen(addr.sun_path) + sizeof(addr.sun_family)) == -1) {
            perror("bind");
            exit(-1);
        }
    
        if ((listen(fd, MAX_CONNECTIONS)) == -1) {
            perror("socket");
            exit(-1);
        }
    
        caddr_size = sizeof(struct sockaddr_un);
        while ((cfd = accept(fd, (struct sockaddr *) &caddr, &caddr_size))) {
            if (cfd == -1) {
                perror("accept");
                continue;
            }
    
            if (!fork()) {
                serve(cfd);
            }
        }
    
        exit(0);
    }
    

The server waits for connections and when one arrives, it creates another
process to handle the interaction and goes back to waiting. Each sub-process
prints the messages coming from the socket. The server could use some
improvements, for example, what happens with the child processes if the server
is terminated?

The client connects to the server and waits for user input and sends it to the
server:

    
    
    #include <stdio.h>
    #include <stdlib.h>
    #include <sys/socket.h>
    #include <sys/un.h>
    #include <unistd.h>
    
    #define SOCKET_NAME "./socket"
    
    int main(int argc, char *argv[]) {
        struct sockaddr_un addr;
        char buf[100];
        int fd, n;
    
        if ((fd = socket(AF_UNIX, SOCK_STREAM, 0)) == -1) {
            perror("socket");
            exit(-1);
        }
    
        memset(&addr, 0, sizeof(addr));
        addr.sun_family = AF_UNIX;
        strncpy(addr.sun_path, SOCKET_NAME, sizeof(addr.sun_path)-1);
    
        if (connect(fd, (struct sockaddr*) &addr, sizeof(addr)) == -1) {
            perror("connect");
            exit(-1);
        }
    
        while ((n = read(STDIN_FILENO, buf, sizeof(buf))) > 0) {
            if (write(fd, buf, n) == -1) {
                /* ignore partial writes */
                perror("write error");
                exit(-1);
            }
            memset(buf, 0, sizeof(buf));
        }
    
        return 0;
    }
    

#### Multi-threaded programming

A thread, by definition, is the smallest object which can be independently
scheduled by the operating system. It's made of a set of cpu registers, a
stack and a program counter, which means that any processor is composed of at
least one thread. In most operating systems today, a process can have more
than one thread; in that case, a single process can have several scheduled
objects which have their own register set copy, their own program counter and
their own stack. In practical terms, a single program can be executed
concurrently by using threads:

    
    
    +----------------------------------------+
    |   code      data    file descriptors   | shared by all threads
    +----------------------------------------+
    |   thread1   |   thread2   |  thread2   |
    |             |             |            |
    | registers   |  registers  |  registers |
    |             |             |            |
    |  stack      |   stack     |   stack    |
    |             |             |            |
    |  program    |   program   |   program  |
    |  counter    |   counter   |   counter  |
    +----------------------------------------+
    

By being independently scheduled and having it's own copy of the resources
needed for execution, while sharing the code, data and file descriptors (among
other things), we can execute different procedures in the same code
concurrently, but in a lightweight manner. While one thread is in charge of
listening to incoming requests over the web, other thread can be sorting
files.

In a multi-core processor, threads can be executed in parallel if different
threads are assigned different cores by the scheduler. This can provide
serious speed advantages as well as an increased responsiveness.

    
    
    +----------------------------------------+
    |   code      data    file descriptors   | shared by all threads
    +----------------------------------------+
    |   thread1   |   thread2   |  thread2   |
    |             |             |            |
    | registers   |  registers  |  registers |
    |             |             |            |
    |  stack      |   stack     |   stack    |
    |             |             |            |
    |  program    |   program   |   program  |
    |  counter    |   counter   |   counter  |
    +----------------------------------------+
    | core1       |    core2    |   core3    |
    +----------------------------------------+
    

The same could be achieved by forking several processes, but threads require
much less resources, and so could be more scalable. Another interesting
advantage of threads over process creation is that by sharing data no
additional IPC mechanisms are needed to share information.

Using threads is not without drawbacks. The sharing of the data between each
thread, which result in economy of resources, means that programmers have to
be very careful not to mess with other thread's data.

#### Multi-threading models

Multi-threading can be implemented in the kernel or at the user level. Most
modern operating systems such as Linux or OpenBSD, offer services for creating
threads; some languages such as Erlang or Go, manage their threads completely
or partially (as in also using kernel threads) by themselves, in the virtual
machine or the runtime.

The usual trade-offs of implementing a service inside the kernel or outside
apply. If we need the kernel to create a thread, a context switch is needed,
thus increasing the overhead; also, adding code to the kernel makes it more
prone to bugs and increases its complexity, which in turn makes it more
difficult and expensive to maintain. On the other hand having the kernel
manage multi-threading allow any computer language to use it and that's
simpler than requiring every runtime to implement their own. Also, there are
some problems that user threads have: what happens if a thread crashes? since
they are scheduled by a process, all other threads are affected too. User
threads are cheaper to create, given that no context switch is needed, so
that's an advantage.

There are compromises between the previous two models. If the kernel provides
multi-threading, the following systems can be implemented:

  * Many to Many (M:N). In this model, several kernel threads manage several user level threads, as the following illustration shows:
    
        t  t  t    t  t  t    t  t  t      Many user threads into N kernel threads
    |  |  |    |  |  |    |  |  |
    |  |  |    |  |  |    |  |  |      t = thread
    |  |  |    |  |  |    |  |  |      k = kernel read
    -------    -------    -------
       |          |          |
       |          |          |
       k          k          k
    

This models is the most complicated one to implement, because it needs the
scheduler in the kernel and each kernel thread also has to schedule each user
level thread. It's main advantage is that it makes switches between threads
very fast, by not needing to call the kernel to block, and at the same time,
the kernel could schedule some of those kernel threads into different cores,
which could increase performance. This approach is used by many modern
programming languages (or rather some of their compilers or interpreters) such
as Go, Erlang and Haskell.

  * One to one (1:1). In this model, every thread is a kernel thread:
    
        t  t  t  t  t  t      Each thread is a kernel threads
    |  |  |  |  |  |
    |  |  |  |  |  |      t = thread
    |  |  |  |  |  |      k = kernel thread
    k  k  k  k  k  k
    

By having every thread be a kernel thread, the scheduler can better use
multiple cpu's to speed up computation, by giving some threads its own cpu,
this can't be done with user level thread management. The main disadvantage is
that thread creation involves a system call, and so, higher overhead. Also,
having too many kernel threads can overwhelm the system, by taking up too much
cpu time. This type of threading is usually used by C and C++ programs.

  * Many to One (M:1). The threads are all user level.
    
        t  t  t  t  t  t  t  t  t      Many user threads into one kernel thread
    |  |  |  |  |  |  |  |  |
    |  |  |  |  |  |  |  |  |      t = thread
    |  |  |  |  |  |  |  |  |      k = kernel thread
    -------------------------
                |
                |
                k
    

Fast thread creation and switching are its main advantages. The fact that the
kernel is only called at process creation makes it very fast to change between
threads and to create new ones. The problems of this model are mainly the fact
that if the process blocks or crashes, the entire thread pool has to block or
crash too, even if they don't need to; also, the kernel cant exploit multiple
cpu's by scheduling several threads into several cpu's at the same time.

### POSIX Threads Programming

The pthread library implements the standard POSIX threads in Linux and other
systems. In a nutshell, pthreads allows us to execute routines as a separate
threads; in order to create a new thread, the following functions and data
structures can be used:

    
    
    int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                       void *(*start_routine) (void *), void *arg);
    
    Arguments:
        pthread_t *thread is an unique identifier for each new thread, returned by pthread_create
        pthread_attr_t *attr is used to set attributes for the thread, or NULL for the defaults
        void *(*start_routine) (void *) this procedure will be executed in a new thread
        void *arg is the argument to start_routine
    

In other words, if we want to create a new thread to execute a part of a
program, we can wrap it into a routine and launch the thread using
`pthread_create`. When it's time for the thread to end, it can call
`pthread_exit`:

    
    
    void pthread_exit(void *retval);
    
    Arguments:
        void *retval is the returning value of the thread
    

If the main process exits, all of it's threads are destroyed too, so it's
important to wait for the threads to terminate before exiting. One way to do
that is by using `pthread_join`, which blocks the calling thread (including
the main thread) until the thread passed by argument exits:

    
    
    int pthread_join(pthread_t thread, void **retval);
    
    Arguments:
        pthread_t thread is the id of the thread to wait for, the id given by pthread_create
        void **retval is the return value of the thread to wait for
    

For a thread to be used by `pthread_join` it needs to be created as a joinable
thread. By default, `pthread_create` creates them as joinable, but it's
usually a good idea to explicitly define them that way so it's more portable.
In order to do that the methods `pthread_attr_init`, `pthread_attr_destroy`
and `pthread_attr_setdetachstate`. This methods are used to initialized,
destroy and set the some data to the attributes passed to `pthread_create`..

    
    
    #include <pthread.h>
    #include <stdio.h>
    #include <stdlib.h>
    
    #define THREADS 10
    
    void *print_thread_id(void *id) {
        printf("Thread number: %d\n", *((long *) id));
        pthread_exit(NULL);
    }
    
    int main() {
        pthread_t thread_ids[THREADS];
        pthread_attr_t attr;
        int i, ret;
    
        pthread_attr_init(&attr);
        pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);
    
        for (i = 0; i < THREADS; i++) {
            ret = pthread_create(&thread_ids[i], &attr, print_thread_id, &i);
            if (ret) {
                exit(-1);
            }
        }
    
        pthread_attr_destroy(&attr);
    
        /* wait for threads to finish */
        for(i = 0; i < THREADS; i++) {
            ret = pthread_join(thread_ids[i], NULL);
            if (ret) {
                exit(-1);
            }
        }
        return 0;
    }
    

To compile this program we need to explicitly link the pthreads library:

    
    
    gcc -pthread main.c -o main
    

After executing it a few times, an error will be evident. Here are a few runs
of the above program:

    
    
    $ ./main
    Thread number: 1
    Thread number: 2
    Thread number: 3
    Thread number: 4
    Thread number: 5
    Thread number: 6
    Thread number: 7
    Thread number: 9
    Thread number: 9
    Thread number: 10
    
    $ ./main
    Thread number: 1
    Thread number: 2
    Thread number: 3
    Thread number: 4
    Thread number: 5
    Thread number: 6
    Thread number: 8
    Thread number: 0
    Thread number: 8
    Thread number: 9
    
    $ ./main
    Thread number: 1
    Thread number: 2
    Thread number: 3
    Thread number: 4
    Thread number: 5
    Thread number: 6
    Thread number: 7
    Thread number: 9
    Thread number: 9
    Thread number: 10
    

Sometimes the thread numbers are bigger than 9, other times different threads
have the same number. Why is this happening? Well, since all threads are
sharing the `i` variable in the counter, it could happen, for example, that
the main thread is taken out from the cpu by a preemptive scheduler and then,
before the `i` variable increases, several threads read and print it. This
illustrates that a great deal of attention is needed for implementing
multithreading (and concurrent) algorithms. In the next section a technique is
shown that helps preventing this types of problems.

For more information, [here's](https://computing.llnl.gov/tutorials/pthreads/)
a good introduction to pthreads.

#### Synchronization of concurrent processes

When several processes have access to the same piece of data, one process can
interfere with another, causing incorrect and inconsistent results. This is
often called a [race condition](https://en.wikipedia.org/wiki/Race_condition),
that is, the system is dependent on the timing of some events outside of our
control (such as the scheduling of threads).

For example, let's say we have an application and want to track the amount of
registered users. For that, every time a new user signs up, the following
procedure is launched:

    
    
    void increase_user_count() {
        int x; /* thread local variable */
        x = users;
        x = x + 1;
        users = x;
    }
    

In a sequential setting, the previous code saves the correct value into the
global variable `users`. But, given that we don't want to block the entire
program for a sign up, we have to execute each request in parallel. Now that
the global variable `users` is shared, both of this executions are possible,
assuming an initial value of `users = n`:

    
    
    |   Scenario 1                        Scenario 2
    |
    |   Thread 1          Thread 2        Thread 1          Thread 2
    |   x = users;                        x = users;
    |   x = x + 1;                                          x = users;
    |   users = x;                                          x = x + 1;
    |                     x = users;                        users = x;
    |                     x = x + 1;      users = x;
    |                     users = x;      x = x + 1;
    |
    |   Result: users = n + 2             Result: users = n + 1
    \/
    time
    

In the first scenario, the expected result was achieved, obviously, because
they were computed sequentially; it could correspond to the scheduler giving
enough time to each thread to finish execution. In the second scenario, the
scheduler gives an order of execution in which both threads copy the `users`
variable into their own local variable, and then increment the same value,
resulting in the wrong result. It's said that a concurrent process gives a
wrong answer when no sequential order of execution could produce the same
result.

It's important to note that the problem is present at the instruction level of
execution. Meaning, it's not a result of the particular programming language
semantics assumed. The scheduler can stop any process from executing (assuming
it's preemptive) in between any consecutive instruction. The previous example,
after compilation into a register based processor architecture would be
something like:

    
    
    mov r1, [users]
    inc r1
    mov [users], r1
    

so the problem remains. In other words, concurrency break our implicit
assumptions about atomic (as in can't be split) execution of sequential code;
now we need to pay attention to the possibility of other code touching our
variables.

#### Critical section

A critical section is the part of a program that could potentially alter a
shared resource, so if it's possible for two processes to execute the
instructions within that area, then the results of the computation may be
incorrect. To make sure this doesn't happen, a solution has to have these
properties:

  * Mutual exclusion: only one process in the critical section at a time
  * Bounded wait: a process waiting to enter the critical section can't wait indefinitely. In other words, there's a finite number of processes which can enter before it
  * Progress: if no process is in the critical section, then one can enter without a waiting period

There's an ingenious software based solution to this problem called [Petterson
algorithm](https://en.wikipedia.org/wiki/Peterson%27s_algorithm), but modern
processor architectures provide a hardware based solution.

#### Test And Set

A computation is said to be atomic if it can't be interrupted in the middle of
its execution; some architectures define certain instructions as atomic,
meaning their execution will not be interrupted in any way. Some of these
instructions give the programmer a sort of anchor they can use to avoid the
problems caused by a preemptive scheduler in concurrent programs.

Test and set is an example of such an instruction. This is the computation it
performs:

    
    
    int test_and_set(int *x) {
        bool old = *x;
        *x = 0;
        return old;
    }
    

Of course, if we compile this, there's no warranty the procedure will be
atomic; this is only for demonstration. To avoid writing assembly code
specific to one architecture, the examples will use the function in a c-like
language and we'll assume the compiler translates the function into the
appropriate instruction, making it atomic.

Just for curiosity, in x86 processors the computation is implemented using
[bts](https://mudongliang.github.io/x86/html/file_module_x86_id_25.html) with
[lock](https://mudongliang.github.io/x86/html/file_module_x86_id_159.html).

How to use this to solve the critical section problem?

    
    
    int lock = 1; /* so the first one may enter */
    
    int main() {
        while(test_and_set(&lock)) {}
        critical_section(); /* critical section code */
        lock = 1; /* open */
    }
    

If there's a set of processes waiting to enter, eventually the first will
arrive at the `test_and_set` and will, atomically, read the lock and change it
to 0, blocking the others, and since the initial value was 1, it will enter.
Then, after leaving the critical section it'll put the lock variable to 1, so
the process can repeat.

Following that protocol, only one process may enter the critical section at
once. To prove it, let's assume there's two processes, P and Q, in the
critical section, at the same time and they are following the protocol. Then
it must be true that both were at some point in the situation in which the
test

    
    
    test_and_set(&lock)
    

was 1 for both. If P entered before Q, and since P didn't leave before Q the
critical section and therefore couldn't have set `lock = 1`, then
`test_and_set(&lock)` returned 0, because P also executed
`test_and_set(&lock)` and it atomically set lock to 0. Then the conclusion is
that P couldn't enter before Q. It's easy to repeat the same reasoning but
swapping P and Q, reaching the conclusion that Q couldn't enter before P. The
only possibility remaining is that they enter at the same time. But, since
`test_and_set` is atomic, they couldn't have. Then, the assumption that P and
Q were at some point in the critical section together must be false.

It's also easy to note that, if a process waits (and the one in the critical
section doesn't hang up), then eventually it will enter it's critical section
(assuming a constant number of waiting processes). Furthermore, if no one is
in it's critical section, one will be allowed to enter.

To finish this section, the following program fixes the user counting example
from the introduction:

    
    
    int lock = 1;
    void increase_user_count() {
        int x; /* thread local variable */
        while(test_and_set(&lock)) {}
        x = users;
        x = x + 1;
        users = x;
        lock = 1;
    }
    

#### Atomic variables

Just like `test_and_set`, some architectures provide sets of instructions over
certain data types that are atomic, for example, integers. If that's the case,
either by using assembly or letting the compiler or interpreter handle it, the
programmer can operate on those data types using those atomic instructions and
avoid some race conditions. Let's go back to the user registration example
above:

    
    
    void increase_user_count() {
        int x; /* thread local variable */
        x = users;
        x = x + 1;
        users = x;
    }
    

As before, we can have the following problems, among others:

    
    
    |   Scenario 1                        Scenario 2
    |
    |   Thread 1          Thread 2        Thread 1          Thread 2
    |   x = users;                        x = users;
    |   x = x + 1;                                          x = users;
    |   users = x;                                          x = x + 1;
    |                     x = users;                        users = x;
    |                     x = x + 1;      users = x;
    |                     users = x;      x = x + 1;
    |
    |   Result: users = n + 2             Result: users = n + 1
    \/
    time
    

Graphically the origin of the problem is clear, the following computation is
not atomic:

    
    
    void inc(int *x) {
        int tmp = *x;   /* fetch int at memory location */
        tmp = tmp + 1;  /* increase it by one */
        *x = tmp;       /* put result in memory location */
    }
    

or in assembler for an imaginary architecture:

    
    
    mov reg, [x]
    add reg, 1
    mov [x], reg
    

But, if the program is compiled to an architecture with the atomic `inc`
instruction, the problems disappear. In assembler:

    
    
    |   Thread 1          Thread 2
    |   inc [users]
    |                         inc [users]
    |
    |   Result: users = n + 2
    \/
    time
    

Both [C++](https://www.cplusplus.com/reference/atomic/atomic/) and
[C11](https://en.cppreference.com/w/c/language/atomic) have support for
defining atomic operations.

#### Semaphores

What happens if we follow the `test_and_set` (or similar) solution if the
`critical_section` takes a long time to complete? Well the waiting processes
are going to `while(test_and_set(&lock))` until they are preempted by the
scheduler. All those cycles are wasted, and would have been better used by
another process in the waiting queue that doesn't need to wait for the lock.
This is bad for the responsiveness of the whole system and bad for the waiting
process itself because usually schedulers diminish the priority of cpu
intensive processes.

A better solution would be to make the processes waiting their turn to enter
the critical section go into the scheduler waiting queue. This is a usual
design decision when a part of a system needs to react to an event:

  * Solution 1: make the waiting objects listen to events and react (e.g web servers, hardware interruptions)
  * Solution 2: make the waiting objects actively question whether the event happen already ([polling](https://en.wikipedia.org/wiki/Polling_\(computer_science\)), [busy waiting](https://en.wikipedia.org/wiki/Busy_waiting))

Solution 1 is usually better, but requires additional capabilities (for
example in the case of hardware interruptions) and in some cases the overhead
incurred by waking processes up to respond to the event is too much and makes
Solution 2 a better approach. Solution 2 is usually a bad choice, but if the
time the process is expected to wait too small, it can be preferable to
Solution 2.

[Semaphores](https://en.wikipedia.org/wiki/Semaphore_%28programming%29) are a
Solution 2 type approach to the critical section problem invented by [Edsger
W. Dijkstra](https://en.wikipedia.org/wiki/Edsger_Dijkstra).

From the point of view of the programmer, a semaphore is an abstract data type
with 2 operations:

  * P() (also called wait, down)
  * V() (also called signal, up, post)

A semaphore contains a number defined at the moment of creation and is
affected by it's two operations in the following way, assuming `s` is the
semaphore:

  * when a process executes `s.wait()` the number in the semaphore decreases by one and if the result is negative, the process blocks.
  * if a process executes `s.signal()` the number increases and if there's some process waiting, one of them will wake up.

The implementations of semaphores vary between operating systems and
programming languages, but the main idea is the same.

To clarify, let's say there's 3 processes {p1, p2 , p3} sharing a semaphore
`s` initiated with a 1. Then `p1` does `s.wait()`, then the value inside `s`
will be 0. Then `p2` and `p3` execute `s.wait()`, and since in both cases the
number in the semaphore is negative, they both block, leaving the number in
-2. Now, after `p1` did it's job (for example `critical_section()`) it
performs a `s.signal()`, at which point the semaphore will contain the number
-1 and one of {`p2`, `p3`} will be chosen, eventually, by the scheduler to
wake up, do it's job and then do `s.signal()`, leaving the number at 0 and
wake up `p1`, who will also `s.signal()`, finally leaving the number at 1
(just like in the beginning).

It's important to note that the value inside the semaphore can't be retrieved,
it can only be set at the beginning.

Knowing how to use semaphores, a solution to our user computationally
intensive critical section problem is apparent:

    
    
    s = semaphore(1); /* global variable */
    void f() {
        s.wait();
        critical_section();
        s.signal()
    }
    

This will work just like discussed earlier, leaving only one process at a time
inside it's critical section, with the advantage that the waiting processes
will be in the waiting queue and the previously wasted cpu cycles can be used
by other processes if the scheduler decides so.

In a way, the provided solution, makes the execution of the code between
`s.wait()` and `s.signal()` atomic with respect to all of the processes
sharing `s`. But then, why not just wrap every sentence in a program between
them? The problem is that semaphores need the kernel to be implemented which
implies context switches, introducing overhead.

Semaphores allow [elegant
solutions](https://greenteapress.com/semaphores/LittleBookOfSemaphores.pdf) to
a lot of synchronization problems, by providing the programmer with clear
rules and abstractions they can work with more comfortably. Some patterns
emerge in concurrent programming that are useful in a multitude of situations.
To show and implement a few of them, an implementation of semaphores is
required.

The POSIX semaphore library provides methods for operating on and creating
semaphores. To use it in Linux, include it with `semaphore.h` and link with
-lpthread. The most important methods it provides are:

    
    
    int sem_init(sem_t *sem, int pshared, unsigned int value);
    

used to create a semaphore with an initial value of `value` and returns it in
the `sem` variable. The `pshared` variable indicates that the semaphore should
be shared among several threads if the value is 0.

    
    
    int sem_destroy(sem_t *sem);
    

Which destroys the semaphore at address `sem`.

    
    
    int sem_wait(sem_t *sem); /* P */
    int sem_post(sem_t *sem); /* V */
    

These two methods implement the two defining operations of a semaphore. All
methods return 0 on success and -1 on error; in the case of `sem_wait` and
`sem_post` if there's an error, no changes are made to the semaphore.

The first pattern to implement is the mutex pattern. A system implementing a
mutex around a critical section, only allows one process at a time to enter;
and only the process inside can decide when to liberate the critical section.
A good problem to solve using mutex, which shows how it works and is simple,
is to count the amount of threads created by letting the threads increment by
one a shared variable initialized with 0; this problem is very similar to the
user registration count problem at the beginning.

    
    
    #include <stdio.h>
    #include <semaphore.h>
    #include <pthread.h>
    #include <stdlib.h>
    
    #define SHARED 0
    #define THREADS 20
    
    int thread_count = 0;
    sem_t sem;
    
    void *thread_inc(void *x) {
        sem_wait(&sem);
        thread_count++;
        sem_post(&sem);
        pthread_exit(NULL);
    }
    
    int main(void) {
    
        pthread_t thread_ids[THREADS];
        pthread_attr_t attr;
        int i, ret;
    
        pthread_attr_init(&attr);
        pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);
    
        if (sem_init(&sem, SHARED, 1) == -1) {
            perror("aisdhfashdf");
            exit(-1);
        }
    
        for (i = 0; i < THREADS; i++) {
            ret = pthread_create(&thread_ids[i], &attr, thread_inc, &i);
            if (ret) {
                exit(-1);
            }
        }
    
        pthread_attr_destroy(&attr);
        for(i = 0; i < THREADS; i++) {
            ret = pthread_join(thread_ids[i], NULL);
            if (ret) {
                exit(-1);
            }
        }
    
        printf("Thread Count: %d\n", thread_count);
    
        if (sem_destroy(&sem) == -1)
            perror("sem_init");
    
        return 0;
    }
    

Since the semaphore is initialized with 1, it lets in the first program to
`sem_wait`, which will also block the others. So, as the program requires,
only one will be allowed in at the same time. Furthermore, only the one inside
the critical section will determine when others can enter, since the last
thing the other threads can do is block. After that thread executes its
critical section, it wakes one up using `sem_post` and the process continues.

The use of semaphores is not without problems. The most notable is the
_deadlock_. A deadlock occurs when every party in a system is waiting for the
other to complete a task. In concurrent programming it can happen in subtle
ways, difficult to debug; usually when two or more programs are waiting for
each other to signal a semaphore. There's a [theoretical
result](https://en.wikipedia.org/wiki/Deadlock) that states that a deadlock
can happen if and only if:

  * at least one resource is held in a non-shareable way, that is, a resource which can be used by one process at a time (Mutual exclusion); and
  * a process holding at least one resource is requesting other resources being held by other processes (hold and wait); and
  * a resource can only be released by the process holding it (no preemption); and
  * there is a set of processes `{p1, ... , pn}` in which `pi` is waiting for `pi+1` to release its resource (circular wait)

Another interesting example is the _livelock_ , which is similar to the
deadlock, but doesn't cause the participants to be blocked, but rather they
continually repeat the same steps to respond to other processes doing the
same. Intuitively, it's what happens when you're walking and another person is
doing the same towards you, and eventually both end up moving to the right and
left to allow the other to pass; but no useful work is being done.

