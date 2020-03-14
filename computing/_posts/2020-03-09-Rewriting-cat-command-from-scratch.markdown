---
layout: post
title:  "Rewriting cat command from scratch"
date:   2020-03-09 00:20:00 +0800
categories: Linux commands
mathjax: true
---


# What is cat command ?
From the Linux man-pages,
> cat - concatenate files and print on the standard output. With no FILE, or when FILE is -, read standard input.


Let's get it started with understanding the code in main loop ...

```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <string.h>
#include <err.h>
#include <sys/stat.h>
#include <unistd.h>

void cat(int readFileDescriptor)
{
    // read from a file
    // to stdout
    int writeFileDescriptor;

    // with a buffer
    static char *buffer;

    // with the correct block size
    static size_t blockSize;

    // that we get from the file stats
    struct stat sbuf;

    writeFileDescriptor = fileno(stdout); // fileDescriptor = 1

    if (fstat(writeFileDescriptor, &sbuf))
    {
        err(1, "stdout");
    }
    blockSize = sbuf.st_blksize;
    printf("blocksize is %lu", blockSize); // blockSize = 1024
    buffer = malloc(blockSize);

    if (!buffer)
    {
        err(1, 0);
    }

    ssize_t nr, nw;
    int offset = 0;

    nr = read(readFileDescriptor, buffer, blockSize);
    printf("nr is %lu", nr); // nr is also 1024

    while (nr > 0)
    {
        for (offset = 0; nr > 0; nr -= nw, offset += nw)
        {
            nw = write(writeFileDescriptor, buffer + offset, nr);
            if (nw < 0)
            {
                err(1, "stdout");
            }
        }

        nr = read(readFileDescriptor, buffer, blockSize);
    }
}

int main(int argc, char *argv[])
{
    // take a list of filenames
    // copies their contents to stdout
    // if one of their filenames is '-', read from stdin
    // no flags implemented

    // segmentation fault occurs when no arguments are given to the cat command

    int fileDescriptor;

    ++argv;
    fileDescriptor = fileno(stdin); // fileDescriptor = 0
    //  The fileno() function shall return the integer file descriptor
    // associated with the stream pointed to by stream.

    do
    {
        if (*argv)
        {
            if (strcmp(*argv, "-") == 0)
            {
                fileDescriptor = fileno(stdin); // fileDescriptor is 0.
            }
            else
            {
                fileDescriptor = open(*argv, O_RDONLY);
            }

            if (fileDescriptor < 0)
            {
                err(1, "%s", *argv);
            }

            ++argv;
        }
        else
        {
            printf("Yo no parameter passed dud; seg fault is prevented \n");
        }

        cat(fileDescriptor); // the thing
    } while (*argv);

    return 0;
}
``` 

### Common terms to demystify the code above
+ argc[**int**] - (ARGument Count) the number of command-line arguments passed to the program 
    + The name of the program is also included (i.e `./a.exe test1 test2 test3`) would result in `argc = 4`
+ argv - (ARGument Vector) a *character pointer* 1 dimensional array which points to each argument passed to the program.
  + argv[0] is the name of the program, argv[argc - 1] are command line arguments and are pointers to strings.
  + argv[argc] is a NULL pointer.
  + argv[1] points to the first command line argument and argv[n] points last argument.
+ fileDescriptor - The Linux kernel maintains a table of all open file descriptors. A file descriptor is a number that represents an open file in a process. Processes have file descriptors, not files. Processes can and often do have more than three file descriptors, and can have fewer. Stdin (standard input), stdout(standard output) and stderr(standard error) are the names for file descriptors 0, 1 and 2 because they have a conventional meaning.

### Main loop

The `int fileDescriptor` initialises to hold the number for standard input (stdin). `++ argv` is used to increment the pointer to the next parameter (`mycat.c`) after the `./a.out`.

In the `do {...} while (*argv);` loop section, we use `if else` statements to test if the 3rd argument is a `-` which means the cat command displays the information as the user types in real-time. `Else`, it returns the file descriptor (an integer value) for the given file name. If the `fileDescriptor`, is less than 0, a formatted error message is printed on the standard error output. If no parameter is passed, we also warn the user that a segmentation fault is prevented. 
Finally, the cat function is invoked with the `fileDescriptor` passed as its parameter. The body of the code in `do {} while ();` loop continues until there is no parameter.

### cat function

I will be going through the code from top to bottom explaining the purpose of every line of code. 

The `int writeFileDescriptor;` variable is used to store the standard output's file descriptor number. The `fstat()` function will obtain information about an open file when given the file descriptor and would write it to the area pointed to by `sbuf`.

The `sbuf` argument is a pointer to a `stat` structure, as defined in `<sys/stat.h>`, into which information is placed concerning the file.

Upon successful completion, 0 shall be returned. Otherwise, −1 shall be returned and errno set to indicate the error.

For example, let's print the contents present in the `sbuf` for the above code.

```c

#include <time.h>
...

puts("fstat() returned:");
printf("block size:   %lu", sbuf.st_blksize);
printf("  inode:   %d\n", (int)sbuf.st_ino);
printf(" dev id:   %d\n", (int)sbuf.st_dev);
printf("   mode:   %08x\n", sbuf.st_mode);
printf("  links:   %ld\n", sbuf.st_nlink);
printf("    uid:   %d\n", (int)sbuf.st_uid);
printf("    gid:   %d\n", (int)sbuf.st_gid);
printf("created:   %s", ctime(&sbuf.st_atime));

...
```

```c
fstat() returned:
block size: 1024
  inode:   3
 dev id:   23
   mode:   00002190
  links:   1
    uid:   1000
    gid:   5
created:   Sat Mar 14 07:20:28 2020
```

`blockSize = sbuf.st_blksize;` 

The `st_blksize` is one of the many members in the sbuf. The `blockSize` is returned as 1024.

`buffer = malloc(blockSize);`

“malloc” or “memory allocation” method in C is used to dynamically allocate a single large block of memory with the specified size (blockSize). It returns a pointer of type void which can be cast into a pointer of any form. Here, we set it to type static char pointer.

```c
if (!buffer){
        err(1, 0);
}
```

The above code verifies that the buffer isn't a Null pointer. An integer constant expression with the value 0, or such an expression cast to type void *, is called a null pointer constant. 

```c
#include <unistd.h>

ssize_t read(int fd, void *buf, size_t count);
```

`read()` attempts to read up to `count` bytes from file descriptor `fd` into the buffer starting at `buf`.
On success, the number of bytes read is returned (zero indicates end of file), and the file position is advanced by this number.

``` c
ssize_t nr, nw;
int offset = 0;
nr = read(readFileDescriptor, buffer, blockSize);
// printf("nr is %lu", nr); // nr is also 1024

while (nr > 0){
    for (offset = 0; nr > 0; nr -= nw, offset += nw){
        nw = write(writeFileDescriptor, buffer + offset, nr);
        if (nw < 0){
            err(1, "stdout");
        }
    }
    nr = read(readFileDescriptor, buffer, blockSize);
}
```

As we copy data from the file(`sample.txt`) to standard out, we need to keep track the number of bytes read(`nr`) and written(`nw`). `ssize_t` is used for functions whose return value could either be a valid size, or a negative value to indicate an error. It is guaranteed to be able to store values at least in the range `[-1, SSIZE_MAX]`. We are using `ssize_t` because the `read` function can fail but as long there are bytes to be read, we will write the bytes to standard output. 

``` c
#include <unistd.h>

ssize_t write(int fd, const void *buf, size_t count);
```

`write()` writes up to count bytes from the buffer starting at buf to the file referred to by the file. On success, the number of bytes written is returned.  On error, -1 is returned, and errno is set to indicate the cause of the error. descriptor `fd`. If `write()` returns an error, -1 is returned, and errno is set to indicate the cause of the error. descriptor `fd`.

We will keep track of the `write()` by introducing an offset variable because `write()` may not write all the data to the standard output. In the `for` loop, the `offset` variable will get incremented by the `nw` (number of bytes written). At the same instance, the `nr` will be subtracted by nw. `nr = nr - nw`.

The `buf` will also be incremented in terms of the offset(1024, 2048 ...) value. This ensures that we output all the bytes from the input file.

---

I hope you understand how the cat command works and with that understanding I am leaving you a question.

Why didn't we use free() to de-allocate the memory for `sbuf` ?

Answer : The OS reclaims the memory faster than the program and the buffer is static and should only be allocated once. Try running Valgrind on them and see for yourself !