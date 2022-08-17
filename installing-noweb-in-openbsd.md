# Notes for installing _noweb_ in openbsd 7.1

Download the source code

	git clone https://github.com/nrnrnr/noweb.git

Following the instructions in `noweb/src/INSTALL`:

	$ cd noweb/src
	$ ./awkname awk	# openbsd uses awk, not nawk
	
Now the right `awk` will be used.

The next step is to set environmental variables for the build process. In
the `INSTALL` file, the author recommends to use a script and not to 
modify `Makefile` directly. Modifying the `Makefile` is faster, so,
the variables to change are:

	# Adjust these two lines for your ANSI C compiler
	# changed gcc to cc since openbsd 7.1 uses clang by default
	CC=cc -ansi -pedantic -O -Wall -Werror
	CFLAGS=

	# BIN is where the commands (notangle, noweave, nountangle, noroots) land
	# LIB is where the pieces of the pipes (nt, markup, unmarkup) are stored
	# MAN is the root of your local man pages directory
	# MANEXT is the extension for your commands' man pages (usually 1 or l)
	# MAN7EXT is the extension for the nowebstyle man page (usually 7)
	# TEXINPUTS is the directory for TeX macro files
	# ELISP is the directory for emacs lisp files, or /dev/null not to install
	BIN=/home/ncb/bin
	LIB=/usr/local/noweb/lib
	MAN=/usr/local/noweb/man
	MANEXT=1
	MAN7EXT=7
	TEXINPUTS=/home/ncb/bin/tex/inputs
	ELISP=/dev/null

To install, first issue (in `noweb/src/`):

	$ make boot		# to prevent using noweb to build noweb
	$ make all install	# build and move files to the right places

To clean some generated files

	$ make clean

References:

- https://www.cs.tufts.edu/~nr/noweb/
- https://github.com/nrnrnr/noweb
