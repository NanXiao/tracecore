---
layout: post
title: Why do I need a root privilege?
---
Last week, the support engineer told me that a strange issue had occurred on commercial system, and gave me an account to let me check. I used this account to log in the system, but when I wanted to use some commands, the system prompted me "Permission denied". I also wanted to use DTrace, but it also requires root privilege. So the following dialogue came out between I and administrator:

I: I need the root privilege, because I want to write some scrips and do some test.
Administrator: This is the commercial system, only operation team members have root privilege. You can send commands to them and let them execute the commands and send results back to you.
I: I need to do further investigation according to the previous results, and this may last a long time. So I think it is convenient for me to operate the system myself.
Administrator: No, it is not allowed for you to operate the commercial system. You can only send your scripts and commands to operation members, and they can send results back. 
I:......

Per my understanding, debugging is a tough progress which may last several days even months, and the engineer need to dig and analyse from previous output then decide what to do next. Sometimes, maybe a digit can spark engineer. So I need a root privilege and do debugging myself, and don't want to send mails back and forth. This disrupts me!

No root privilege, it really sucks!