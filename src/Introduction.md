
# Welcome to RuxOS world!

This is RuxOS book, which covers following points:

- RuxOS overview.

- Getting started with RuxOS apps.

- RuxOS design.

RuxOS Book is still working in progress. 

# What is RuxOS?

RuxOS is a lightweight library operating system compatible with Linux applications, mainly following the [unikernel](https://en.wikipedia.org/wiki/Unikernel) design idea, maintained by [syswonder community](https://www.syswonder.org/#/).

Considering that in edge ubiquitous computing scenarios, the number of applications is usually limited and relatively fixed, so the operating system is simplified and designed to only support a single application and encapsulates the kernel functions. As a library, it is provided to applications in the form of system calls, and applications run directly in the kernel state. 

The performance of operating system applications in this library form will be greatly improved, and security issues will be mainly solved by the underlying Type 1 hypervisor ([hvisor](https://github.com/syswonder/hvisor)). Library-shaped operating systems require good tool support to facilitate users to generate and construct runnable binary images based on a single application, such as [unikraft](https://unikraft.org/).

RuxOS is still working in progress, and we appriciate your contributions.
