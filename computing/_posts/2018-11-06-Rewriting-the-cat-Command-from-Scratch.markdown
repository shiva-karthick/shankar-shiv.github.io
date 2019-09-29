---
layout: post
title:  "Rewriting the cat Command from Scratch"
date:   2018-11-06 10:27:55 +0800
categories: Command Line
---

The cat (short for concatnate) command is one of the most frequently used commands in Linux/Unix like operating systems. The cat command allows us to create single or multiple files, view contents of a file, concatenate files and redirect output in terminal or files.

The Linux manual page for `cat`:

> The cat utility reads files sequentially, writing them to the standard output.  The file operands are processed in command-line order.  If file is a single dash ('-') or absent, cat reads from the standard input.  If file is a UNIX domain socket, cat connects to it and then reads it until EOF. This complements the UNIX domain binding capability available in inetd(8).

The `cat` command also takes in several command line flags but for simplicity; we will be ignoring them. Our minimal implementation of `cat` will have the following functionalities:

1. Takes a list of file names.
2. Copies their contents to `stdout`.
3. If one of the file names is `-` it reads from `stdin` until the user presses `control + D` to indicate end of file (`EOF` on `stdin`).

## Implementation

Just give me the code!

```c
#include <err.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/stat.h>

void cat(int rfd)
{
    int wfd = fileno(stdout), offset = 0;
    static char *buf;
    static size_t bsize;
    ssize_t nr, nw;
    struct stat sbuf;
    
    if (fstat(wfd, &sbuf)) {
        err(1, "stdout");
    }

    bsize = sbuf.st_blksize;
    buf = malloc(bsize);
    if (!buf) {
        err(1, 0);
    }

    nr = read(rfd, buf, bsize);
    while (nr > 0) {
        for (offset = 0; nr > 0; nr -= nw, offset += nw) {
            nw = write(wfd, buf+offset, nr);
            if (nw < 0) {
                err(1, "stdout");
            }
        }
        nr = read(rfd, buf, bsize);
    }
}

int main(int argc, char *argv[])
{
    int fd = fileno(stdin);
    ++argv;
    do {
        if (*argv) {
            if (!strcmp(*argv, "-")) {
                fd = fileno(stdin);
            } else {
                fd = open(*argv, O_RDONLY);
            }
            if (fd < 0) {
                err(1, "%s", *argv);
            }
            ++argv;
        }
        cat(fd);
    } while(*argv); 
}
```

We're working on a command line tool, so our C program needs to take in command line arguments. We initialize a file descriptor `int fd` that defaults to the file descriptor of `stdin`. We can retrieve the file descriptor of `stdin` using `fileno`. We skip over to the next argument in `argv` because our file name is always the first command-line argument. We would want to execute the `cat` command for all the file names we pass in as an argument to our program; a `do while` loop is suitable for this task.

We then check for empty command-line arguments. If the user passes in file names as command-line arguments, we check if the command-line argument is a `-` using `strcmp`. If the file name is a `-`, the file descriptor would be the file descriptor of stdin. If it's a regular file name, we `open` the file on read-only mode (`O_RDONLY`). `open` returns a file descriptor for the opened file. A file descriptor is a small, non-negative integer that is used in subsequent system calls to refer to the open file. 

**What is a file descriptor?**

> In simple words, when we open a file, the operating system creates an entry to represent that file and store the information about that opened file. So if there are 100 files opened in your OS, then there will be 100 entries in OS (somewhere in the kernel). These entries are represented by integers like (..., 100, 101, 102, ...). This entry number is the file descriptor. A file descriptor is an integer number that uniquely represents an opened file in the operating system. 

If the file descriptor is a negative number, we display a formatted error message on the standard error output. We define a function called `cat` and pass in the file descriptor of the opened file.

In the `cat` function, we take the file descriptor we're reading from and write that file to `stdout`. We'll use a writing file descriptor that holds the file descriptor value of `stdout`. We'll also use an `offset` value because write operations might not always be successful, so we need to keep track of *"how much"* of the file we've written to `stdout`. We initialize a buffer; `buf` for the writing file descriptor so that we can read n-bytes from the file descriptor into the buffer. We want the buffer to use the best block size, so we initialize a `static size_t` variable — `bsize`. We also need to keep track of the number of bytes read and written, so we initialize two variables, `nr`, and `nw`. We need to find out the preferred block size for an efficient file system I/O; `struct stat sbuf` stores this information. The `struct stat` is a system `struct` that is defined to store information about files. We need to fill up our `struct stat` with the file stats; this operation can fail, so we do some error handling. The `st_blksize` field of the `struct stat` contains the block size information our buffer needs, so we assign its value to bsize. Once we have the best block size available for our buffer, we allocate a section of memory that's `bsize` wide. Again, this operation could fail, so we do some error handling. Next, we read up to `bsize` bytes from file descriptor `rfd` into the buffer starting at `buf`. If the operation succeeds, the number of bytes read is returned (zero indicates the end of file), and the file position is advanced by this number. We loop and write to `stdout` as long as there's something to be read. The write operation might be prone to failure so we'll always start off at an `offset` value that was set by the previous write operation. The write function writes up to `nr` bytes from the buffer pointed `buf` with an offset to the file referred to by the file descriptor `wfd`. If the operation succeeds, the number of bytes written is returned (zero indicates nothing was written). On error, `-1` is returned, and `errno` is set appropriately. For every iteration, we decrease and increase the value `nr` and offset by `nw` respectively. We write to `stdout` until we have nothing to write.

Now, compile the program (make sure to save the file as cat.c):

```bash
gcc cat.c -o cat
```

To test out the program:

```bash
./cat /usr/share/dict/words
```

Congratulations! You've rewritten the `cat` command from scratch.
