
# ruxmusl

This chapter introduces how [musl libc](https://www.musl-libc.org/) is integrated to RuxOS. `ruxmusl` is tested by libc-bench.

## From musl libc to syscall

### importance to import musl libc

`musl libc` provides corresponding C APIs to C programs and encapsulates the underlying system call API. At the same time, not all C functions require system calls. A large number of string processing functions do not require system calls. These functions that do not require system calls provide a relatively complete and efficient implementation in `musl libc`. 

Therefore, after integrating `musl libc` into RuxOS, the string and other related functions implemented by the original ryxlibc are no longer needed, and more APIs are provided without the need to implement them one by one.

### system call number

System calls are a set of portable APIs defined by the kernel. The system call numbers corresponding to system calls of different architectures may be different. In AARCH64, you can get the A64-defined system calls and their call numbers by looking at `path/to/musl-libc/arch/aarch64/bits/syscall.h.in`.

Comparing the system calls of different architectures, we can find that not only the system call numbers are different, but the types are also different. For example, A64 provides two system calls, `dup` and `dup3`, but not `dup2`. `dup2` is implemented through `dup + fcntl`; but x86_64 provides `dup/dup2/dup3`.

### trigger a synchronization exception

How to enter the kernel system call layer from the libc function? Take the `read()` function as an example.

* `read()` is a function provided by the C standard library and is implemented in `path/to/musl-libc/src/unistd/read.c`. You can see that `syscall_cp(SYS_read, fd, buf, count)` is called. Here, the system call is called through macro, defined in `path/to/musl/arch/aarch64/syscall_arch.h`. You can see that `svc 0` is passed here, and the relevant parameters are stored in `x0~x5`, and the system call number is stored in `x8`, which means that up to 6 parameters are supported.

* `svc 0` will trigger an interrupt and enter the kernel's interrupt exception vector table. It will trigger a synchronization exception, which will be taken over and processed by the kernel. 

## System call handling (A64)

### parse parameters

The synchronization exception will enter `handle_sync_exception()` in `modules/ruxhal/src/arch/aarch64/trap.rs` of RuxOS. The exception triggering cause is obtained by parsing the `ESR` exception status register, and the system call parameters and system call number are obtained by reading `x0~x5` and `x8`.Then enter the system call processing function.

The function `handle_syscall()` is added to the `TrapHandler` trait in `modules/ruxhal/src/trap.rs`, and is only available with `MUSL` feature. The implementation is in `ulib/ruxmusl/src/trap.rs`. System call number is parsed here and then distributed to corresponded handling function.

### system call distribution

The distribution process is in `ulib/ruxmusl/src/syscall/mod.rs`, and the system call number is defined in `ulib/ruxmusl/src/syscall/syscall_id.rs`. It is consistent with the standard system call number (A64), but it is not yet complete. The distribution process is matching system call numbers, parsing different numbers of parameters according to the system call number, and then calling the specific system call implementation in [`ruxos-posix-api`](./ruxos-posix-api.md).

## Use ruxmusl

The relevant makefile has been modified to integrate the `MUSL` feature. When compiling a C program, **add `MUSL=y` to compile the application** through `musl libc`.

Some of the features that `ruxmusl` must include have been added to the makefile, and no additional work is required:

* **fp_simd**: Since musl needs to compile all C standard library functions involving `double` types, floating point registers are needed.
* **fd**: Since musl needs to initialize stdin/stdout/stderr in the form of a file, this feature needs to be turned on.
* **tls**: `musl libc` manages tls by itself in the thread, and needs to get things like `pthread_self()` through `tls`, so it needs to be turned on.

RuxOS will automatically pull and compile the musl libc source code. Users must add `ARCH=aarch64` because it is currently only integrated for A64.

The compilation process is similar to the previous ruxlibc. The previous ruxlibc will compile a `.a` file under the `target/` directory, and `musl libc` will also generate a `libc.a` file under the `install/` directory after compilation. Use this `.a` file for linking (already integrated into makefile).

