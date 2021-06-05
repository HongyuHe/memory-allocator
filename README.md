# Memory Allocator

[![Maintenance][maintain-badge]][maintain-act] [![PRs Welcome][pr-badge]][pr-act] [![Build Status][travisci-badge]][travisci-builds]


## Introduction

This memory allocator is akin to `ptmalloc2`, `jemalloc`, `tcmalloc` and many others. These allocators are the underlying code of `malloc`. 

Heap allocators request chunks of memory from the operating system and place several (small) object inside these. Using the free call these memory objects can be freed up again, allowing for reuse by future malloc calls. Important performance considerations of a heap allocator not only include being fast but also to reduce memory fragmentation.

`malloc`, `free` and `realloc` have been implemented without using the standard library versions of these functions. `brk(2)` and `sbrk(2)` has been used for asking the kernel for more heap space.

*(Normal heap allocators may also use `mmap` to request memory from the kernel.)*

## Implementation Details

1. `malloc`, `free` and `realloc` behave exactly as specified by their man-page description, whilst the Notes sections have been skipped as these are implementation-specific. 
3. This allocator does not place any restrictions on the maximum amount of memory supported or the maximum number of objects allocated. For example, it will scale regardless of whether the maximum `brk` size is 64KB or 1TB.
4. A region of memory can be reused after freeing it with `free`.
5. `realloc` behaves as described on its man-page and only allocates a new object when needed.
6. The `allocator` batches `brk` calls, i.e., it does not need to request memory from the kernel for every allocation.
7. The amortized overhead per allocation is on average 8 bytes or less.
8. The allocator tries to optimize for locality (reuse recently freed memory).
9. The allocator gives back memory to the kernel (using `brk`) when a large portion of the allocated memory has been freed up.
10. The allocation functions work correctly without the `my` prefix.

## Execution

The file `test/tests.c` contains a number of (automated) test cases that evaluate the different aspects of your allocator. It can be invoked manually via .`/test <test name>`. Running `make check` will run all test cases.

## Notes

* If you want to add support for replacing the system allocator (i.e., by adding non my prefixed functions) you can use your allocator for any existing program on your system. You can do this by prefixing any command with `LD_PRELOAD=/path/to/libmyalloc.so. For example, LD_PRELOAD=./libmyalloc.so ls` will run `ls` with the allocator.
* Naming the functions such as `malloc` instead of `mymalloc` not only redirects all calls inside the code to this `malloc`, but will also cause all internal `libc` calls to go to this allocator instead of the built-in `libc` `malloc`. Many `libc` functions, such as `printf`, internally make calls to malloc, and as such using `printf` inside the allocation code would cause an infinite loop. Therefore I prefix the allocator functions with `my`, but you are free to remove it.
* MacOS is currently not supported but you could set up a Linux environment if you do not already have one (e.g., with a VM using virtualbox).

[travisci-badge]: https://travis-ci.com/HongyuHe/memory-allocator.svg?branch=main
[travisci-builds]: https://travis-ci.com/HongyuHe/memory-allocator
[maintain-badge]: https://img.shields.io/badge/Maintained%3F-yes-green.svg
[maintain-act]: https://github.com/HongyuHe/memory-allocator/graphs/commit-activity
[pr-badge]: https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square
[pr-act]: http://makeapullrequest.com