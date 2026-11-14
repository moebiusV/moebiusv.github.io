# moebiusv

[Security](#defensive-coding) | [Computer History](#computer-archaeology) | [AI](#ai) | [Business and Productivity](#business-and-productivity) | [Home automation, robotics, and device control](#home-automation-robotics-and-device-control) | [Networking](#networking) | [Writings](#writings)

I work in open source on Linux and BSD. Over the years I've contributed to Debian, the Linux kernel, the FreeBSD kernel, OpenBSD, and the newLisp programming language. Lately Alpine Linux and the Ada language have my attention.

## Defensive coding

**Military grade for the modern age.**

The Ada language was designed for the US Department of Defense and became MIL-STD-1815 in 1980. These projects make it accessible for everyday programming on modern open source operating systems: Linux, BSD, macOS, and Windows.

- [imp72](https://moebiusv.github.io/imp72/): Edgar "Ned" Irons's extensible systems language, developed at the NSA and used for their Folklore time-sharing system. Walter Bilofsky's 1973 PDP-10 implementation, ported to LLVM for Linux, BSD, and macOS, and it makes programs for Windows too (mingw-w64, tested under Wine). Cross compiling is one command, for Linux (glibc and musl), the BSDs, macOS, and Windows: `imp --target=aarch64-linux-gnu prog.imp` fetches the target's C library and headers from [cruntime](https://moebiusv.github.io/cruntime/) and links.
- [cruntime](https://moebiusv.github.io/cruntime/): the headers, start files, C library and libm of Linux (glibc and musl), Windows (mingw-w64) and the BSDs, one directory per OS, version and architecture, laid out as a `--sysroot`: everything LLVM needs to cross compile for them, from any project. Begun for imp72, not specific to it.  Any cross compiler can use it.
- [ada-on-alpine](https://moebiusv.github.io/ada-on-alpine/): a complete Ada development toolchain packaged for Alpine Linux.
- [imsg-ada](https://moebiusv.github.io/imsg-ada/): Ada implementation of the OpenBSD imsg wire protocol and API for inter-process communication.
- [soja](https://moebiusv.github.io/soja/): a drop-in replacement for the soju IRC bouncer, written in Ada 2022: same configuration, database and BouncerServ commands, with OAuth 2, PAM, Web Push and SASL EXTERNAL certificates. *We are all sojas now.*
- [libsodium-ada](https://moebiusv.github.io/libsodium-ada/): Ada wrapper for the libsodium cryptography library.
- [mustache-ada](https://moebiusv.github.io/mustache-ada/): Ada implementation of the Mustache template language and API.

## Computer archaeology

- [pdp64](https://moebiusv.github.io/pdp64/): the PDP-11 architecture extended to 64 bits and SMP.
- [pcc](https://moebiusv.github.io/pcc/) (fork): the Portable C Compiler. Includes the optimizations from Dennis Ritchie's original PDP-11 compiler and Steve Johnson's VAX compiler, and now produces code even better than the originals. Until the fixes are merged, upstream produces code 60% larger than the original compilers did. Also adds AArch64 and pdp64 CPU backends for testing porting and cross compiling. The Ritchie and Johnson optimizations benefit all architectures.
- [v7unix-toolchain](https://moebiusv.github.io/v7unix-toolchain/): the Research Unix V7 cross-compiler (cc, as, ld, lex, yacc, make). Runs natively on modern platforms and makes V7 Unix binaries for the PDP-11, byte identical to what the original tools produce on a real PDP-11.
- [prebsd](https://moebiusv.github.io/prebsd/): fetch and run Research Unix V4, V5, V6, V7, and 32V on the SIMH emulator (PDP-11/70, VAX-11/780).  Plus a few more ancient 1970s operating systems like TOPS-10 on the PDP-10.  Handles all the configuration and setup for you.
- [filsys](https://moebiusv.github.io/filsys/): a FUSE driver for the Unix filesystem, from the very first version on the PDP-7 to the last release, Version 10, and others like Coherent and Xenix. Keeping the promise the Linux kernel made in the beginning: it supports existing filesystems. It covers all the different versions of the Unix filesystem with full autodetect and full read/write, plus filesystem creation and repair utilities.

## AI

- [hbnf](https://moebiusv.github.io/hbnf/): a parser generator and compiler-compiler that solves some of the major pain points of parsing.  Emits parsing code in the major languages (C, Rust, Zig, and Ada) to be included directly in your program. From small projects, to binary wire protocols, to complex languages like COBOL and C++, it does it all in a form that is compact, readable, and very efficient. No LR conflicts ever!
- [cngpt](https://moebiusv.github.io/cngpt/): nanoGPT translated to C, with full speed optimizations.  OpenBLAS matmul, full backprop, and a compiler benchmark harness.
- [cugpt](https://moebiusv.github.io/cugpt/): microgpt translated to C, with full speed optimizations.  One with tape autograd and one with BLAS manual backprop.

## Business and productivity

CSV is the universal business interchange format. The csv tool lets you use it securely, safely, and easily from the Unix command line and in scripts.

- [winless](https://moebiusv.github.io/winless/): a vi-like text editor for Windows. It can run remotely over ssh, and makes it very easy to annotate, edit, and correct documents directly or with the help of an AI.
- [bepsi](https://moebiusv.github.io/bepsi/) *(wip)*: automated payments for real-world goods. Pay crypto, get a beverage.
- [buildabook](https://moebiusv.github.io/buildabook/): builds a finished book, cover included, ready to send straight to the printer. Makes sure the page numbers, footnotes, table of contents and everything else are correct. Keywords: Markdown, Typst, PDF.
- [ytran](https://moebiusv.github.io/ytran/): fetch YouTube transcripts, clean them up with Claude, write an executive summary, and store them in a local database.
- [bs3](https://moebiusv.github.io/bs3/): an interactive terminal browser for SQLite databases, with vi-style navigation.  Makes it easy to explore and edit your databases with a few keystrokes.
- [csv](https://moebiusv.github.io/csv/): work with your CSV business data files from the Unix command line. Includes head, tail, cut, grep, sort, and more. It handles CSV headers and comments, and works in Unix pipelines.
- [wsl-dns-forward](https://moebiusv.github.io/wsl-dns-forward/): makes .local printer and computer names resolve inside WSL2 on Windows 10, so ssh, ping, and CUPS find them. It is for Windows 10; it is compatible with Windows 11, although not needed there.

## Home automation, robotics, and device control

- [bepsi](https://moebiusv.github.io/bepsi/) *(wip)*: code for the DCTRL Bepsi machine. Pay crypto, get a beverage.
- [seatuya](https://moebiusv.github.io/seatuya/): a C library with a stable ABI for local Tuya smart-device control. Makes it easy to connect to and control Tuya devices from any programming language. I use it for temperature control when making cheese.

## Networking

- [9Frank](https://moebiusv.github.io/9Frank/): a network filesystem protocol that is a superset of all 9P2000 dialects, plus improvements.
- [wsl-dns-forward](https://moebiusv.github.io/wsl-dns-forward/): makes .local printer and computer names resolve inside WSL2 on Windows 10, so ssh, ping, and CUPS find them. It is for Windows 10; it is compatible with Windows 11, although not needed there.

## Writings

- [Ada functional style guidelines](ada-style-guidelines.html): a style guide for writing Ada code in a functional style. Ada is famously verbose; this shows how to make it smaller and clearer, closer to modern programming languages.
- [What the Viking Poets Knew](https://moebiusv.github.io/papers/papers/viking-poets/): a book arguing that traditions which have survived for millennia have lessons for AI researchers, some of which have already been rediscovered since the book was released.
- [12 Hz Power Grid for National Energy Sovereignty](https://moebiusv.github.io/papers/papers/12hz-power-grid/): six papers on rapidly expanding power grid capacity, and making it safer, cheaper, and more secure at the same time by running it at 12 Hz instead of 60 Hz.
