
# ruxos-posix-api

The ruxos posix api layer provides APIs similar to the [posix](https://en.wikipedia.org/wiki/POSIX) standard, to support [ruxlibc](./ruxlibc.md) and [ruxmusl](./ruxmusl.md). The ruxos posix api layer features the following characteristics:

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

### sys_clone VS sys_pthread_create

### sys_futex VS sys_mutex_xxx

### sys_mmap

### sys_pthread_key_xxx

## syscall list

| Module Name | Syscall Name | Description |
| --- | --- | --- |
| **io_mpx** | sys_epoll_create | Create an epoll fd |
| | sys_epoll_ctl | Control interface for an epoll file descriptor |
| | sys_epoll_pwait, sys_epoll_wait | Wait for an I/O event on an epoll file descriptor |
| | sys_poll, sys_ppoll | |
| | sys_pselect6, sys_select | |
| **io** | sys_read, sys_write | |
| | sys_readv, sys_writev | |
| **resources** | sys_getrlimit, sys_prlimit64, sys_setrlimit | Get/Set resource limits |
| **rt_sig** | sys_rt_sigaction, sys_rt_sigprocmask | |
| **stat** | sys_umask | |
| **sys** | sys_sysinfo | |
| **task** | sys_exit, sys_getpid, sys_sched_yield | |
| **time** | sys_clock_gettime, sys_clock_settime, sys_nanosleep | |
| **fd_ops** | sys_close, sys_dup, sys_dup2, sys_fcntl, sys_dup3 | |
| **fs** | sys_fdatasync, sys_fstat, sys_fsync, sys_getcwd, sys_lseek, sys_lstat, sys_mkdir, sys_mkdirat, sys_newfstatat, sys_open, sys_openat, sys_rename, sys_renameat, sys_rmdir, sys_stat, sys_unlink, sys_unlinkat | |
| **ioctl** | sys_ioctl | |
| **mmap** | sys_mmap, sys_mprotect, sys_munmap | |
| **net** | sys_accept, sys_bind, sys_connect, sys_freeaddrinfo, sys_getaddrinfo, sys_getpeername, sys_getsockname, sys_listen, sys_recv, sys_recvfrom, sys_send, sys_sendmsg, sys_sendto, sys_setsockopt, sys_shutdown, sys_socket | |
| **pipe** | sys_pipe, sys_pipe2 | |
| **signal** | sys_getitimer, sys_setitimer, sys_sigaction | |
| **pthread** | sys_pthread_getspecific, sys_pthread_key_create, sys_pthread_key_delete, sys_pthread_setspecific, sys_clone, sys_set_tid_address, sys_pthread_create, sys_pthread_exit, sys_pthread_join, sys_pthread_self | |
| **pthread::futex** | sys_futex | |
| **pthread::condvar** | sys_pthread_cond_broadcast, sys_pthread_cond_destroy, sys_pthread_cond_init, sys_pthread_cond_signal, sys_pthread_cond_timedwait, sys_pthread_cond_wait | |
| **pthread::mutex** | sys_pthread_mutex_destroy, sys_pthread_mutex_init, sys_pthread_mutex_lock, sys_pthread_mutex_trylock, sys_pthread_mutex_unlock | |

