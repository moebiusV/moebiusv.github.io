# moebiusv

[Defensive coding](#defensive-coding) | [Computer archaeology](#computer-archaeology) | [AI](#ai) | [Business and productivity](#business-and-productivity) | [Home automation, robotics, and device control](#home-automation-robotics-and-device-control) | [Networking](#networking) | [Writings](#writings)

I work in open source on Linux and BSD. Over the years I've contributed to Debian, the Linux kernel, the FreeBSD kernel, OpenBSD, and the newLisp programming language. Lately I've been working on Alpine Linux.

## Defensive coding

**Bringing military grade into the modern age.**

Ada was written for the US Department of Defense (MIL-STD-1815). These projects bring that discipline to Linux, BSD, and macOS.

- [imp72](https://moebiusv.github.io/imp72/): Edgar "Ned" Irons's extensible systems language, developed at the NSA and used for their Folklore time-sharing system. Walter Bilofsky's 1973 PDP-10 implementation, ported to LLVM for Linux, BSD, and macOS.
- [cruntime](https://moebiusv.github.io/cruntime/): the headers, start files, C library and libm of Linux (glibc and musl), Windows (mingw-w64) and the BSDs, one directory per OS, version and architecture, laid out as a `--sysroot`: everything LLVM needs to cross compile for them, from any project. Begun for imp72, not specific to it.
- [ada-on-alpine](https://moebiusv.github.io/ada-on-alpine/): a complete Ada development toolchain packaged for Alpine Linux.
- [imsg-ada](https://moebiusv.github.io/imsg-ada/): Ada implementation of the OpenBSD imsg wire protocol and API for inter-process communication.
- [soja](https://moebiusv.github.io/soja/) *(wip)*: an IRC bouncer in Ada 2022, translated from soju. *We are all sojas now.*
- [libsodium-ada](https://moebiusv.github.io/libsodium-ada/): Ada wrapper for the libsodium cryptography library.
- [mustache-ada](https://moebiusv.github.io/mustache-ada/): Ada implementation of the Mustache template language and API.

## Computer archaeology

- [pdp64](https://moebiusv.github.io/pdp64/): the PDP-11 architecture extended to 64 bits and SMP.
- [pcc](https://moebiusv.github.io/pcc/) (fork): the Portable C Compiler, with a PDP-11 focus.
- [v7unix-toolchain](https://moebiusv.github.io/v7unix-toolchain/): the Research Unix V7 cross-compiler (cc, as, ld, lex, yacc, make) for modern systems.
- [prebsd](https://moebiusv.github.io/prebsd/): fetch and run Research Unix V4, V5, V6, V7, and 32V on the SIMH emulator (PDP-11/70, VAX-11/780).
- [filsys](https://moebiusv.github.io/filsys/): a FUSE driver for Unix disk images from 1969 to 1989, Version 0 to Version 10, with full read/write.

## AI

- [hbnf](https://moebiusv.github.io/hbnf/): a parser generator and compiler-compiler supporting many programming languages.
- [cngpt](https://moebiusv.github.io/cngpt/): nanoGPT in C, with OpenBLAS matmul, full backprop, and a compiler benchmark harness.
- [cugpt](https://moebiusv.github.io/cugpt/): C translations of microgpt, one with tape autograd and one with BLAS manual backprop.

## Business and productivity

CSV is the universal business interchange format. The csv tool lets you use it securely, safely, and easily from the Unix command line and in scripts.

- [winless](https://moebiusv.github.io/winless/): a less pager for Windows, with annotation and editing based on vim.
- [bepsi](https://moebiusv.github.io/bepsi/) *(wip)*: automated payments for real-world goods. Pay crypto, get a beverage.
- [buildabook](https://moebiusv.github.io/buildabook/): build a printable book from Markdown and Typst files, with cover, table of contents, and footnotes.
- [ytran](https://moebiusv.github.io/ytran/): fetch YouTube transcripts, clean them up with Claude, and store searchable summaries in a local SQLite database.
- [bs3](https://moebiusv.github.io/bs3/): an interactive terminal browser for SQLite databases, with vi-style navigation and CGA colors.
- [csv](https://moebiusv.github.io/csv/): a Unix-style CSV processor with head, tail, cut, grep, sort, and more. It is CSV-aware, handles headers and comments, and works in pipes.
- [wsl-dns-forward](https://moebiusv.github.io/wsl-dns-forward/): makes .local printer and computer names resolve inside WSL2 on Windows 10, so ssh, ping, and CUPS find them. Includes a Windows 11 upgrade path.

## Home automation, robotics, and device control

- [bepsi](https://moebiusv.github.io/bepsi/) *(wip)*: code for the DCTRL Bepsi machine. Pay crypto, get a beverage.
- [seatuya](https://moebiusv.github.io/seatuya/): a C library with a stable ABI for local Tuya smart-device control, usable from any language with FFI.

## Networking

- [9Frank](https://moebiusv.github.io/9Frank/): a network filesystem protocol that is a superset of all 9P2000 dialects, plus improvements.
- [wsl-dns-forward](https://moebiusv.github.io/wsl-dns-forward/): a local DNS forwarder that answers .local (mDNS) questions inside WSL2 through Windows and passes every other question to the normal DNS server.

## Writings

- [Ada functional style guidelines](ada-style-guidelines.html): a style guide for functional-style Ada. Every example is compiled with GNAT.
- [What the Viking Poets Knew](https://moebiusv.github.io/papers/papers/viking-poets/): a book arguing that traditions which have survived for millennia already knew what AI researchers are only now rediscovering.
- [12 Hz Power Grid for National Energy Sovereignty](https://moebiusv.github.io/papers/papers/12hz-power-grid/): six papers on running a grid at 12 Hz, so transformer cores can use commodity steel instead of specialty grain-oriented steel from a handful of foreign mills.
