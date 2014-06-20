---
layout: post
title: An experience of fixing a memory-corruption bug
---
During the last 4 months, I was disturbed by a memory-corruption bug, and this bug will cause program crash. Until last Monday, I found the root cause and fixed it. This debug process is a difficult but memorable experience, so I will share it in this article.

My program works as a SMS Hub. When it receives a SMS, it will allocate a structure in heap memory like this:  

    typedef struct
    {
    ......
    int *a[8];
    ......
    } info;
After processing the SMS, the program will free the memory, and send the SMS to the next Hub or Operator. 

Since last November, the program will crash sometimes, and the cause is the second element in array `a (a[1]) `will be changed from a valid value to NULL.

(1)  Checking the logs and reproduced the bug 
Firstly, I checked the commercial logs, but there were no clues can be found. And the SMS which killed program also seemed no different than others. I also tried to use this SMS to reproduce the bug in testbed, but also failed.

(2) Using libumem
Because our program runs in Solaris, I linked the libumem and hoped it can help me. After a few days, the program crashed again. But the tags before and after the corrupted memory are all OK, so it is not a memory off-bound bug. I also checked the memory before and after the corrupted memory, but nothing valuable can be found.

(3) Adding more logs
Until then, the only thing I can think is adding more logs. After adding enough logs, I found the variable is modified between functions in the same thread: when leaving the last function, the variable is OK, but entering the next function, the variable is changed. So I can make sure the variable is destroyed by another thread. But how can I find the murderer?

(4) Asking for help from other geeks
I began to ask for help from other geeks. I put this question on the stackoverflow: [How to debug the memory is changed randomly issue](http://stackoverflow.com/questions/20737582/how-to-debug-the-memory-is-changed-randomly-issue), and received a very good and comprehensive answer. I recommended every one should read this post. I also sent emails to other geeks, most of them replied and gave some suggestions, such as exchange some variables definition sequences, etc. I also wanted to get the root cause from the core dump file directly: [Can I know which thread change the global variable's value from core dump file?](http://stackoverflow.com/questions/21643736/can-i-know-which-thread-change-the-global-variables-value-from-core-dump-file), but at the end, I am failed.
Until one day, I found an [article](http://www.cnblogs.com/djinmusic/archive/2013/02/04/2891753.html) (Aha, written in Chinese!)by accident, this article describes the bug is very similar to mine except his programming language is C++ and mine is C. I began to follow the steps he provided to find the root cause.

(5) Porting the program on Linux and use valgrind
I wanted to use valgrind to help find the bug, but my program runs on Solaris, and valgrind can't be used on Solaris, so another colleague helped to ported it on Linux. After running valgrind, I did find some memory-related bugs, but these bugs can't explain the cause of this issue. I still can't find the root cause.

(6) Using electric-fence
I tried to use electric-fence, but the [home page](http://perens.com/FreeSoftware/ElectricFence/) of electric-fence was closed. I found some packages and source codes from the internet, and tried to integrated them into my program. But this time, I also failed.  

(7) Using mprotect function
Because the second element of the array is always changed to NULL, I used mprotect instead of malloc to allocate the memory of the array, and set read-only attribute of the memory. So if the memory is changed, the program will crash. But the mprotect will allocate a page size of the memory, and this may cause the program change the behavior, I am not sure whether the bug can occur again.

（8） Finding the cause
About 2 weeks later, when a colleague stopped the application, the application crashed again. The cause was a global variable was changed. I checked all the old core dumps immediately, and found the global variable was always changed, and pointed to the address of the array! It seemed I was very clear to the truth.

After about 2 days analysis, the cause was found:  When a thread calls pthread_create to create another thread, it will changes a global variable. But in few cases, the child thread will execute firstly, and it will also changes the global variable. The code assumes the parent thread always execute firstly, and this will cause the program crash.

When looking back the 4-month experience, I have studied a lot of things for debugging this bug: libumem, valgrid, libefence, etc. It is a really memorable and cool experience! 