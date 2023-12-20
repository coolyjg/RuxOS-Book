
# ruxlibc

Inspired by [relibc](https://github.com/redox-os/relibc) and [nolibc](https://github.com/unikraft/unikraft/tree/staging/lib/nolibc), `ruxlibc` is a C user library implemented by Rust. Its main purposes are:

* **Small and concise**. It only contains some of the most commonly used API implementations in libc, greatly reducing the size of the image file. `ruxlibc` uses Rust `feature` and the characteristics of the unikernel operating system to implement conditional compilation according to the application, so that the generated C user library only contains the modules necessary for the application.

* **Based on Rust to ensure memory safety**. Drawing inspiration from `relibc`, `ruxlibc` uses Rust to ensure memory safety as much as possible from the language level. At the same time, reasonable setting of error types and corresponding processing facilitates debugging by OS developers and application developers.

## Features

The implementation of `ruxlibc` follows musl libc, and uses Rust `feature` to achieve tailoring. The features included are introduced below.

| Feature Name | Feature Description | 
| --- | --- |
| smp | Enable SMP (symmetric multiprocessing) support. |
| fp_simd | Enable floating point and SIMD support. |
| irq | Enable interrupt handling support. |
| alloc | Enable dynamic memory allocation. |
| tls | Enable thread-local storage. |
| multitask | Enable multi-threading support. |
| fs | Enable file system support. |
| net | Enable networking support. |
| signal | Enable signal support. |
| fd | Enable file descriptor table. |
| pipe | Enable pipe support. |
| select | Enable synchronous I/O multiplexing ([select]) support. |
| epoll | Enable event polling ([epoll]) support. |
| random-hw | Enable random number generation by hardware instruction. |

## How to use ruxlibc

See [Your App](../../chap03/your_app.md) to use `ruxlibc` for C applications.
