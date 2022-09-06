### File Systems

File systems are a way of organizing data in a storage medium, to accomplish
some design objectives, such as fast retrieval, redundancy, fast writes, etc.
Files are the fundamental unit for user interaction with the file system; they
usually consist of a sequence of bytes. Also, files are named, so they can be
referenced when operating on them.

Operations in most file systems include **open** , **close** , **read** and
**write**. The open operation opens a file in the file system, giving to the
process that called the operation a file descriptor, an identifier for that
file, with which it can reference it during further operations; in case the
file referenced doesn't exist, it could create it. Close is the reverse
operation. Read and write operations can add and retrieve information stored
into a file, in any part of the file.

In order to reference a file, a name is given to it. In UNIX like systems,
such as Linux or openBSD, a filename is a string of characters separated by
'/' which denotes a hierarchical separation between files. This introduces a
new concept: the directory. A directory groups files and other directories
under a common name, forming a tree, called a directory tree:

    
    
    /
    |
    +---------+-------+
    |         |       |
    /bin    /usr    /home
                      |
                      |
              +---------------+
              |               |
          /home/user1      /home/user2
    

Another interesting operation on file systems is the mount operation. Mounting
refers to hanging a directory tree view of a file system on a node of the
directory tree. With this operation is possible to have a directory tree
composed of different file systems and different devices. For example, let's
say a usb storage device is connected to the computer and the system
automatically mounts it into the current directory tree, under
/home/user2/devices/usb/:

    
    
    /
    |
    +---------+-------+
    |         |       |
    /bin    /usr    /home
                      |
                      |
              +---------------+
              |               |
          /home/user1    /home/user2
                              |
                         /home/users2/devices
                              |
                         /home/users2/devices/usb
                              |
                            .....
    

Now, all the files within the usb storage device can be accessed by prepending
/home/users2/devices/usb to their name. This kind of uniformity in dealing
with files on different devices and file systems allows the programmer to
abstract the particular details of each, and only use the same operations for
all of them. These operations are provided as part of the operating system
services, in the form of system calls. In UNIX operating systems, system call
such as:

    
    
    int open(const char *pathname, int flags, mode_t mode);
    int close(int fd);
    ssize_t read(int fd, void *buf, size_t count);
    ssize_t write(int fd, const void *buf, size_t count);
    

are used for any kind of file. The kernel knows what kind of underlying file
system a file is in and, when receiving call to perform an operation, it
applies the right algorithms and protocols to interact with the hardware and
data structures of the appropriate file system. This is the [Virtual File
System](https://en.wikipedia.org/wiki/Virtual_file_system)(vfs) in operation.
The vfs is a layer of abstraction on top of other files systems; this provides
the uniform access talked about above, defining a set of operations that can
be performed in every file system.

    
    
    +------------+     system calls   +-------------------+
    |Applications| <----------------> |Virtual File System|
    +------------+                    +-------------------+
                                        |       |
               +------------------------+       |
               |                                |
         +-----------+                    +-----------+
         |file system|                    |file system|
         +-----------+                    +-----------+
               |                                |
               |                                |
            +------+                        +------+
            |device|                        |device|
            +------+                        +------+
    

The indirection that the Virtual system provides allows for interesting
applications, with almost no cost to programmers. For example, the storage
medium could be behind a network, and the kernel would access the network with
the right protocols and operate on files remotely, transparently to the
programmer.

In operating systems based on Unix, the standard interfaces provided by file
systems (open, read, write) are used to represent other resources than storage
hardware. For example, in Linux, the proc file system gives information about
kernel data structures related to processes; in this directory there's a
directory of the name /proc/[pid] for every running process by pid inside of
which related information can be found, such as /proc/1000/cmdline contains
the command line string used to launch the program. It's also possible to
write into certain files and change some process attributes.

This approach of exposing interfaces as files was pioneered by Unix and taken
to the extreme in systems like [plan9](https://9p.io/plan9/), in which, most
resources are accessed through files. It provides an example of the
abstractions provided by the operating system, shaping the way in which the
system as a hole feels; if I'm programming for Unix like systems, and need to
interact with the system, I'll first check if reading a file can give me the
information I need, or writing a file can change what I want, and then use
text manipulation tools such as grep, sed, etc.

One important difference between file systems is the type of file access they
allow. Some allow for sequential access and others provide random access. In
the first type, it's only possible to read from beginning to end, not directly
in the middle or the end. The latter gives the possibility of accessing any
byte in the file directly; in this cases, the operation seek is usually
provided, which lets a program put the pointer inside the file (where the read
or write operations take or add data) into an arbitrary position.

#### The fat file system

The [File Allocation Table](https://wiki.osdev.org/FAT) (fat) file system
considers a storage media as an array of clusters, which are contiguous areas
of storage in the device. A table, called file allocation table (hence the
name of the scheme), implements a set of linked lists, one for every file,
containing the sequence clusters where a file is stored. Each entry in the
table has the address of the next cluster or a distinctive value indicating
the end of the file. So, a file, which is stored in several clusters, not
necessarily contiguous, is a linked list of clusters. For example to read some
data in the third cluster, first we find the first cluster in the table, from
then we get the address of the third cluster.

    
    
    simplified fat table
    
    cluster | next cluster
    --------+-------------
       0    |    1
       1    |    END
       2    |    4
       3    |    EMPTY
       4    |    END
    

In the example, we have two files. One spans the 0 and 1 clusters and the
second the 2 and 4.

The fat tables are located in the disk and the layout proposed by this system
is something like:

    
    
     +-------------+----------------------------+----------------------------+-------------+
     | boot record |  file allocation table 1   |   file allocation table 2  |   data area |
     +-------------+----------------------------+----------------------------+-------------+
    

A directory in this system is a file containing directory entries. This
directory entries have information about files in that directory, including
the name, first sector, and other important metadata. So, to find the file
/documents/file.txt, first we search in the directory entries of the root file
for the documents file; after getting the data, we can know the clusters
corresponding to the directory /documents, so we can read its own directory
entries, looking for the one containing the name file.txt. This directory
entry will contain the address of the first cluster. With the direction of the
first cluster, and the file allocation table, it's possible to finally read
the file. The root directory is always at the end of the second fat table, so
it's always possible to find it.

Given that secondary storage media access is comparatively slower, the fat
table can be kept in memory for efficiency reasons. The downside of this
approach is that if the system goes down, the changes since the last time it
was saved too disk are lost. For this reason, periodically the fat is saved to
permanent storage.

Several formats of the fat file system have been created, fat32, fat16, fat12,
which change the format a little, but the principle remains the same. The main
difference is the amount of bits that can use to address clusters, and so, the
amount of clusters they can use. For example, fat32 has, of course, 28 bits of
addresses.

#### The ext2 file system

The [ext2](https://www.nongnu.org/ext2-doc/ext2.html) file system is based on
the i-node data structure. An i-node represents an object on the file system
(e.g a file); it contains pointers to blocks of storage and the metadata of
said object (permissions, owner, last modification timestamp, etc.). Some of
those blocks of storage are also i-nodes, implementing another two levels of
indirection.

A file in the system is referenced by an i-node and its contents are saved in
storage blocks. The i-node points to those objects in a specific way:

    
    
    i-node pointers
    
    +-----------------------------+
    | first block address         |------->block with file contents
    +-----------------------------+
    | second block address        |------->block with file contents
    +-----------------------------+
    | ...................         |
    +-----------------------------+
    | 12th block address          |------->block with file contents
    +-----------------------------+         +--------------------------+----->block with file contents
    | indirect block address      |-------->|block filled with pointers|  ...
    +-----------------------------+         +--------------------------+----->block with file contents
    |doubly-indirect block address|-----+
    +-----------------------------+     |      +------------------------------------------+
                                        +----->|pointer to indirect address pointer blocks|
                                               +------------------------------------------+
    

So, a file that spans 12 blocks can be reached by reading the first 12 entries
in the i-node pointers. If more blocks are needed, then, the block address can
be found in the block pointed to the 13th entry, which contains a pointer to a
block of addresses, where the data is. So, to know where in the device the
file is, we only need to get a hold of its i-node.

A directory is implemented as a file holding directory entries. The directory
entries contain a name, i-node tuple, for every file or directory it contains;
so if an i-node for a directory is found, a list of it's files and directory
can be obtained by iterating its content. The root directory, by standard, is
always at the second i-node. To find a directory, starting from the root, read
the directory entries, find the next directory and keep doing it recursively
until the last directory, and then, locate the file.

In ext2, the device is divided up into block groups, each containing the same
amount contiguous blocks and i-nodes pointed to them. The i-nodes are
contained in tables, with each block group having its own. Each block group
contains a bitmap of allocated blocks and another one for free i-nodes. So the
contents of, e.g. a file, are in blocks of some block groups:

    
    
    storage device
    +--------------------------------------------------------------+
    | block group 0 | block group 1 | ............. | block group n|
    |               |               |               |              |
    +--------------------------------------------------------------+
    

The number of block groups are determined by the capabilities of the device,
e.g, the number of blocks. This information, along with all the metadata of
the file system, is located in the superblock. The superblock is, by standard
definition, at the byte number 1024 of the device; information such as amount
of i-nodes, amount of blocks per block group, free i-nodes, etc, is contained
in this superblock. Because of its contents, its very important for mounting
the system, and so, several backups of the superblock exist, in different
block groups.

After the superblock the block group table is located; it describes where the
important information is in each block group, such as the table of i-nodes,
the free i-nodes and blocks bitmap, etc.

As mentioned earlier, in order to find a file or a directory, finding it's
i-node was sufficient and the procedure for doing so was explained. The
missing piece of information is, how are the i-nodes referenced? and how to
find one?. Each i-node has a unique number, counting from 1. As said before,
the second i-node, that is the i-node number 2, is reserved for the root
directory. With that reference and the information of how many i-nodes per
group there are, simple arithmetic (i-node number / i-nodes per group) gives
the block group where the i-node is. Then, inside the i-node, computing the
rest of the previous division gives the place inside the i-node table (which
we can find using the block group table).

To summarize, the layout of a storage device formatted using ext2 is:

    
    
    storage device
    +--------------------------------------------------------------+
    | boot data and | block group 0 | ............. | block group n|
    | information   |               |               |              |
    +--------------------------------------------------------------+
    

And each group contains:

    
    
    +-------------------------------+--------------+--------+--------+--------+
    | superblock | Block group      | block bitmap | i-node | i-node | data   |
    |            | descriptor table |              | bitmap | table  | blocks |
    +------------+------------------+--------------+--------+--------+--------+
        1 block      x blocks           1 block     1 block  y blocks  z blocks
    

Depending on the version of ext2, the superblock may or may not be replicated
in each block group.

#### GFS

The [Google File System](https://research.google.com/archive/gfs-sosp2003.pdf)
(gfs) was design by Google in order to solve some internal storage problems.
It's last version was released in 2010. It's an example of a file system
running in several machines (distributed) and also implemented in user space;
some computers are used as storage, one as a master which keeps metadata and
responds to several control messages, and the clients making the requests to
both kinds of servers.

The design objectives address specific needs, and observations made by Google
engineers.

Each computer in the system is going to be built out of inexpensive commodity
components prone to failure; so, the system can not assume reliability from
its hardware components, and so, must provide it itself.

The expected number of files are around a million, typically 100MB in size
each, but often multi-GB files are expected. The system should optimize for
large files and should manage files of any size, but small ones don't require
optimization. The kinds or reads expected are large and sequential reads, from
hundreds of KB to several MB's each; small random reads are often a few KB's
in size. When a read is performed, often the following adjacent data are also
read. Once written, files don't change often. The typical change through a
write are append operation with the same kind of data magnitudes as in reads.
Random writes should be supported, but no optimizations should be needed. An
operation for atomic appends for concurrent clients operating in the same file
should exist without imposing synchronization logic over the applications.

According to the [paper](https://research.google.com/archive/gfs-
sosp2003.pdf), application using this file system benefit more from a high,
sustained bandwidth than low latency.

The file system is made up of the following parts or concepts:

The **chunks** are a similar concepts as blocks in ext2. They are a chunk of
storage, each one of the same size. **Files** are stored in several chunks.
Each chunk has a 64 bits immutable unique identification number, called
**chunk handle**. The size of the chunks chosen was 64MB, in order to decrease
the amount of chunks, which helps to keep their ids in a memory table. Also,
the clients need less communication with the master to figure out where the
chunks are, since more data is in the same chunk.

    
    
                              Chunk handles
    +-------+
    | file  |---------------> 0xffff
    |       |---------------> 0xaa23487ff
    |       |---------------> 0xadf112304
    +-------+
    

A **chunk server** is in charge of storing the chunks. Write and read
operations reach these servers and are applied to the data or the data is
transferred. They are implemented on top of a Linux server and file system in
user space. Each chunk can be saved as a file in the underlying file system.
For reliability, the chunks are redundantly stored in several chunk servers;
this implies that synchronization mechanisms are needed when a mutation is
performed.

File system metadata is stored and handled in a centralized server called the
**master**. It stores information such as the file tree and its access control
list, chunk id's and the map between files and chunks; those data structures
are kept in memory to make the system more efficient. This server responds to
control messages such as a client asking where a files' chunk is. It also
handles asking the chunk servers to communicate their state, periodically
removing unused chunks, helping synchronize writes through the lease
mechanism, and asking the chunk servers for their state at start up. Master
servers are implemented as user processes running on top of, for example,
Linux. The master server operates and stores its data structures in memory, so
to protect the system from failures, an operation log is kept in disk and
safely replicated, so the state of the operating system can be reestablished;
to decreased startup time after a failure, a snapshot of the structures in
memory is taken after the operation log saves enough operations.

Finally, the **clients** communicate with the master server to get information
about the file system and interchange data with chunk servers. It can be
implemented as code library which follows the API to talk to the servers. To
reduce the amount of contention in the master server, the client caches
metadata.

Together, all the above pieces form a **gfs cluster**.

    
    
    +----------------------------------+
    | master server:                   |  chunk handle and location
    | - file tree                      |---------------------------->+-------------------+
    | - chunk handles -> chunk servers |                             |Application through|
    | - files -> chunk handles         |<----------------------------|gfs client         |
    +----------------------------------+    filename at chunk number +-------------------+
          |                     ^                                      |          ^
          |                     |                                      |          |
          |instructions to      |chunk server                          |          |
          | chunk server        | state                                |          |
          \/                    |           chunk handle and byte range|          | data
    +--------------+      +--------------+<----------------------------+          |
    |chunk server 1| .... |chunk server n|                                        |
    +--------------+      +--------------+----------------------------------------+
    Linux file system      Linux file system
    

The namespace is the collection of file names in the system. As usual, they
are a combination of characters separated by /, for example /home/user. All
namespace operations are carried out in the master server. In order to create
a file, the client sends a control message to the master, which adds the file
name into the namespace. Since several clients could be trying to perform the
same action, or make changes to the namespace concurrently which could induce
an inconsistency, a series of read and write locks are used to synchronize the
operations. When a new file is created, chunks need to be allocated for it,
which the master is also in charged of; by sending instructions to the chunk
servers to allocate a chunk and associate it with the provided id, and also.
Finally, the master will update its data structures to reflect the change.

Since reliability is expected of the system, but the hardware won't provide
it, the master is also in charged of setting up replicas for the recently
created chunks. Those replicas serve 2 purposes. First, in case of hardware
failure, data lost is prevented by replication. Second, the master performs
load balancing, so clients can ask different servers for parts of the data,
also better using the available bandwidth.

To read a file, a client needs the chunk containing the data. Since a file can
have several chunks, and each chunk is of fixed size, the byte offset in the
file can be used to compute the chunk number. With that data, the client sends
a request to the master (if the information is not cached), expecting a
response with the chunk handle and location it needs; the request includes the
file name and chunk number, used by the master server to perform a lookup.

Once the client has the chunk handle and location (chunk server), it sends a
data request to the appropriate chunk server. The request includes the chunk
handle and the range of bytes the chunk server should transfer. The client
should cache the metadata to speed up its operations and to decrease the load
on the master.

Since writes can be done concurrently on the same file by more than one
client, and the data is replicated across many chunk servers, writes are a bit
more complicated. Changes to the data should be done in every replica; to do
so, the master chooses one of the servers as the primary chunk server
temporarily (every 60 seconds, the primary server can ask to renew). This
process is called giving a lease. The primary server is in charged of
serializing the operations upon the chunk, and the other replicas have to
apply the operations following that serialization.

When a client wants to write to a file, which finally translates to writing
into a chunk, it asks the master for the primary chunks server and the
secondary replicas. If no server was established as primary, the master
chooses one, and sends back the decision to the client. The client then sends
the data to all the replicas, and, after confirmation of the transfer, it
sends the operation to perform. The operations are performed first in the
primary server, and if no errors occur, the serialization of the operations
are sent to the secondary replicas, who then perform the operations.

Reference: [paper](https://research.google.com/archive/gfs-sosp2003.pdf).

