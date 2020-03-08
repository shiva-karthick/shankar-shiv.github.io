---
layout: post
title:  "Rewriting cat command from scratch"
date:   2020-03-09 00:20:00 +0800
categories: Linux commands
mathjax: true
---


Here is the code ...

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
    // printf("The buffer pointer is %p \n", buffer);
    // printf("The buffer pointer is %p \n", buffer++); // same address as above

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