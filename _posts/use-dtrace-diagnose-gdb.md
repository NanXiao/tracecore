---
layout: post
title: Use DTrace to diagnose gdb issues
---

A few days ago, I installed the newest 64-bit gdb program (version 7.7.1) on Solaris 10 (X86_64 platform) to debug programs. After playing with the gdb a day, I found 2 issues about gdb:  
(1) The "set follow-fork-mode child" command doesn't take effect. By default, after the parent process forks the child process, the gdb will track the parent process, so this command can make gdb begin to follow the child process. But this command works OK on Linux.  
(2) The gdb can't parse the 32-bit application core dump file. Per my understanding, the 64-bit gdb should parse both 32-bit and 64-bit core dump files.

I think these are 2 bugs, so I reported them to the gdb:https://sourceware.org/bugzilla/show_bug.cgi?id=17044 and https://sourceware.org/bugzilla/show_bug.cgi?id=17045. But unfortunately, there is no any responses from the gdb. Suddenly I hit upon an idea: since I worked on Solaris, why not use DTrace? Yes, DTrace. DTrace is a very cool application which can trace application, and tell you the function flow of the application. So I start working on it at once!