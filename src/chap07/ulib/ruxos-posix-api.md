
# ruxos-posix-api

The ruxos-posix-api layer provides APIs similar to the [posix](https://en.wikipedia.org/wiki/POSIX) standard, to support [ruxlibc](./ruxlibc.md) and [ruxmusl](./ruxmusl.md). The ruxos-posix-api layer features the following characteristics:

* APIs are designed and implemented based on POSIX standards.

* APIs are implemented by Rust, providing both rust and C APIs with the same name.

* Paying attention to the characteristics of the unikernel operating system, some posix APIs have been specially treated and implemented.

* `ruxos-posix-api` provides a variety of Rust features to reduce the kernel size.

In this chapter, RuxOS Book will explain the purpose of `ruxos-posix-api` and the difference from the standard posix API.

## Purpose

In order to provide support for user libraries and standard C libraries, the POSIX standard defines a set of common APIs for compatibility between different systems. 

`ruxos-posix-api` provides the implementation of posix system calls. The number of parameters and parameter types strictly follow the posix system call standards, and is used to support the upper-layer ruxlibc and ruxmusl user libraries. Decoupling the user library from the system call layer facilitates system developers to better locate bugs and errors, making the hierarchical relationship more obvious.

## Difference

`ruxos-posix-api` differs from the standard posix system call API in the following ways:

* [syscall API name](#syscall-api-name)

* [detailed syscall](#detailed-syscall)

* [architecture ignored](#architecture-ignored)

### syscall API name

The system call name starts with *sys_* (see [syscall list](#syscall-list)), followed by the specific function name. 

### detailed syscall

However, some syscall APIs in RuxOS are specifically used for [ruxlibc](./ruxlibc.md) and are not in the posix standard library. These APIs are specially pointed out in [syscall list](#syscall-list), like `sys_pthread_create`. 

By adding more detailed syscalls and specializing the pthread library in particular, the purpose of this design is to shorten the function call path and parameter parsing time.

Due to the characteristics of the unikernel operating system, user programs and the kernel run at the same privilege level. Therefore, direct function calls instead of long system calls can improve program performance and reduce debugging time for programmers and developers.

Some specially treated APIs are explained in detail [below](#special-apis).

### architecture ignored

In the standard POSIX syscall specification, different architectures have different system call numbers and system call names. 

However, in ruxos-posix-api, we have not yet made a distinction. There may be mutual calls between system calls for simplicity.

## Special APIs

All APIs customized by RuxOS are marked in italics in the [list](#syscall-list).

Some APIs are described and explained below.

### sys_clone VS sys_pthread_create

In libc, `sys_clone` is used to create threads or processes. The kernel determines whether to create a thread or a process by parsing the incoming flag, and sets the shared attributes of the new thread or new process. 

In the unikernel operating system, RuxOS only supports one process and does not support the `fork` function. Therefore, RuxOS's `sys_clone` semantics only need to satisfy the creation of new threads, and new threads always share resources such as address space, file descriptor tables, signals, etc. with the parent thread.

By simplifying the semantics, RuxOS can complete thread creation faster and maintain the semantics of thread creation that are basically consistent with Linux. In [ruxlibc](./ruxlibc.md), syscalls become function calls, so RuxOS does not need to support the creation of new threads through `sys_clone`, but directly supports the implementation of `pthread_create` and just throws new tasks into the scheduling queue.

### sys_futex VS sys_mutex_xxx

In kernels such as Linux, futex is used to support mutex, etc., and synchronization is completed by monitoring the value of an address. 

Since the kernel has provided various lock implementations, when supporting ruxlibc, RuxOS implemented the lock API starting with `pthread_mutex` to directly call the lock API implemented in kernel and is compatible with the lock definition in musl libc. 

When supporting musl libc, RuxOS implemented a simplified version of futex semantics. When the value of the address monitored by the thread remains unchanged, the `yield `function will be called to actively give up the CPU.

### sys_mmap

`mmap` maps a virtual address to the user program and allocates a specific physical address to the user program by processing page fault. 

In RuxOS, since the user program has only one process and owns the entire physical address space, the current `mmap` is to directly allocate content of a given size to the user program and release it in `munmap`. At the same time, RuxOS does not support file mapping, so it does not handle fd.

### sys_pthread_key_xxx

The `pthread_key` related APIs provide thread-local key-value pairs. The implementation of this part should belong to libc. 

In order to support these APIs more simply and directly, RuxOS puts the corresponding key-value into the `task_inner` structure, simplifying the implementation of ruxlibc. 

In musl libc, this part is placed into a thread-related memory managed by musl libc, so RuxOS does not need to provide corresponding syscalls.

### sys_freeaddrinfo, sys_getaddrinfo

These two APIs in libc are to parse addresses and release corresponding data structures. Its implementation belongs to the libc layer, since it needs to read the `/etc/hosts` file on the local machine to send DNS queries, RuxOS has specially processed it to simplify the implementation.

## syscall list

| Module Name | Syscall Name | Description |
| --- | --- | --- |
| **io_mpx** | sys_epoll_create | Create an epoll fd |
| | sys_epoll_ctl | Control interface for an epoll file descriptor |
| | sys_epoll_pwait, sys_epoll_wait | Wait for an I/O event on an epoll file descriptor |
| | sys_poll, sys_ppoll | Wait for some event on a file descriptor |
| | sys_pselect6, sys_select | Monitor multiple file descriptors |
| **io** | sys_read, sys_write | Read/Write from a fd |
| | sys_readv, sys_writev | Read/Write data from/into multiple buffers |
| **resources** | sys_getrlimit, sys_prlimit64, sys_setrlimit | Get/Set resource limits |
| **rt_sig** | sys_rt_sigaction | Examine and change a signal action |
| | sys_rt_sigprocmask | Examine and change blocked signals |
| **stat** | sys_umask | Set file mode creation mask |
| **sys** | sys_sysinfo | Return system information |
| **task** | sys_exit | |
| | sys_getpid | |
| | sys_sched_yield | |
| **time** | sys_clock_gettime, sys_clock_settime | |
| | sys_nanosleep | |
| **fd_ops** | sys_close | |
| | sys_dup, sys_dup2, sys_dup3 | |
| | sys_fcntl | |
| **fs** | sys_fdatasync, sys_fsync | |
| | sys_fstat, sys_lstat, sys_newfstatat, sys_stat | |
| | sys_getcwd | |
| | sys_open, sys_openat | |
| | sys_mkdir, sys_mkdirat | |
| | sys_rename, sys_renameat | |
| | sys_unlink, sys_unlinkat | |
| | sys_lseek | |
| | sys_rmdir | |
| **ioctl** | sys_ioctl | |
| **mmap** | sys_mmap, sys_munmap | |
| | sys_mprotect | |
| **net** | sys_accept, sys_bind, sys_connect, sys_listen | |
| | sys_socket | |
| | sys_shutdown | |
| | *sys_freeaddrinfo, sys_getaddrinfo* | |
| | sys_recv, sys_recvfrom | |
| | sys_send, sys_sendmsg, sys_sendto | |
| | sys_getsockname | |
| | sys_setsockopt | |
| | sys_getpeername | |
| **pipe** | sys_pipe, sys_pipe2 | |
| **signal** | sys_getitimer, sys_setitimer | |
| | sys_sigaction | |
| **pthread** | *sys_pthread_getspecific/setspecific* | |
| | *sys_pthread_key_create/delete* | |
| | *sys_pthread_create, sys_pthread_exit* | |
| | *sys_pthread_join* | |
| | *sys_pthread_self* | |
| | sys_clone | |
| | sys_set_tid_address | |
| **pthread::futex** | sys_futex | |
| **pthread::condvar** | *sys_pthread_cond_timedwait, sys_pthread_cond_wait* | |
| | *sys_pthread_cond_init/destroy* | |
| | *sys_pthread_cond_signal/broadcast* | |
| **pthread::mutex** | *sys_pthread_mutex_destroy/init* | |
| | *sys_pthread_mutex_lock/trylock/unlock* | |

