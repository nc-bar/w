## Extending the file metaphor

Unix included security features from the time it introduced fle permissions
and users. Files have owners, both groups and users, and a set of permissions.
For example:

`

    
    
    d-rwxr-xr-x M 0 ncb ncb      0 Aug  6 12:24 forth
    d-rwxr-xr-x M 0 ncb ncb      0 Aug  3 23:03 groff
    --rw-r--r-- M 0 ncb ncb   1596 Jul 31 18:11 installing-noweb-in-openbsd.md
    d-rwxr-xr-x M 0 ncb ncb      0 Aug  5 15:47 ncb.ar
    

`

The list above is a list of files and directories (the latter beign just a
kind of file). The most important part of every item, concerning security, are
the ownership and the permissions flags:

    
    
    --rw-r--r-- ncb ncb      installing-noweb-in-openbsd.md
    d-rwxr-xr-x ncb ncb      ncb.ar
    

The second and third columns say that the `ncb` user and the `ncb` group own
the file.

The first column is a string of flags which say who can `read`, `open` or
`execute` the file. The first flag in the first row above, (`-`) indicates
that `installing-noweb-in-openbsd.md` is a normal file; the second row says
that `ncb.ar` is a directory (`d`).

The last three triplets set the `owner` user, `group` owner and `other` users
permissions. Letters `r`, `w` and `x` denote `read`, `write` and `execute`:

    
    
    	--rw-r--r-- ncb ncb      installing-noweb-in-openbsd.md
    	d-rwxr-xr-x ncb ncb      ncb.ar
    

In this case, the user `ncb` can `r`ead and `w`rite the `installing-noweb-in-
openbsd.md` file but it can't e`x`ecute it. The `d` in the beginning of the
second row implies the file is a directory; the execution permission in a
directory means it can be opened.

This scheme is very flexible and can be used to secure the access to files. If
that is the case, why not extend it to everything? I mean: want to launch a
process? you need permissions to open and write into `/proc`, after all,
creating a process from an executable file could be done by:

    
    
    	cp a.out /proc/user/
    

Want to open a socket?

    
    
    	fp = fopen("/sockets/some-name", RW);
    

By extending the file methaphor, all the different interfaces and permissions
merge into the file system. The programmer doesn't need to learn other system
calls and their permissions and restrictions. Libraries would also be
simplified, after all many syscalls would just be familiar operations on
files.

Another interesting application would be to restrict a process capabilities by
restricting it's view of the file system. For example, if some server should
never send or receive packets from an interface, just unmount that part of the
file system. Something similar to openBSD's
[unveil](https://man.openbsd.org/unveil.2). A process could unmount some
filesystem and then `exec` into another process which can't re-mount it.

