---
layout: post
title: A trick of building multithreaded application on Solaris
---
Firstly, Let's see a simple multithreaded application:  

	#include <stdio.h>
	#include <pthread.h>
	#include <errno.h>

	void *thread1_func(void *p_arg)
	{
			   errno = 0;
			   sleep(3);
			   errno = 1;
			   printf("%s exit, errno is %d\n", (char*)p_arg, errno);
	}

	void *thread2_func(void *p_arg)
	{
			   errno = 0;
			   sleep(5);
			   printf("%s exit, errno is %d\n", (char*)p_arg, errno);
	}

	int main(void)
	{
			pthread_t t1, t2;

			pthread_create(&t1, NULL, thread1_func, "Thread 1");
			pthread_create(&t2, NULL, thread2_func, "Thread 2");

			sleep(10);
			return;
	}
What output do you expect from this program? Per my understanding, the `errno` should be a thread-safe variable. Though The `thread1_func` function changes the `errno`, it should not affect `errno` in `thread2_func` function.   

Let's check it on Solaris 10:

	bash-3.2# gcc -g -o a a.c -lpthread
	bash-3.2# ./a
	Thread 1 exit, errno is 1
	Thread 2 exit, errno is 1
Oh! The `errno` in `thread2_func` function is also changed to `1`. Why does it happen? Let's find the root cause from the `errno.h` file:  

	/*
	 * Error codes
	 */

	#include <sys/errno.h>

	#ifdef  __cplusplus
	extern "C" {
	#endif

	#if defined(_LP64)
	/*
	 * The symbols _sys_errlist and _sys_nerr are not visible in the
	 * LP64 libc.  Use strerror(3C) instead.
	 */
	#endif /* _LP64 */

	#if defined(_REENTRANT) || defined(_TS_ERRNO) || _POSIX_C_SOURCE - 0 >= 199506L
	extern int *___errno();
	#define errno (*(___errno()))
	#else
	extern int errno;
	/* ANSI C++ requires that errno be a macro */
	#if __cplusplus >= 199711L
	#define errno errno
	#endif
	#endif  /* defined(_REENTRANT) || defined(_TS_ERRNO) */

	#ifdef  __cplusplus
	}
	#endif

	#endif  /* _ERRNO_H */

We can find the `errno` can be a thread-safe variable(`#define errno (*(___errno()))`) only when the following macros defined:  

    defined(_REENTRANT) || defined(_TS_ERRNO) || _POSIX_C_SOURCE - 0 >= 199506L

Let's try it:  

	bash-3.2# gcc -D_POSIX_C_SOURCE=199506L -g -o a a.c -lpthread
	bash-3.2# ./a
	Thread 1 exit, errno is 1
	Thread 2 exit, errno is 0
Yes, the output is right! 

From [Compiling a Multithreaded Application](http://docs.oracle.com/cd/E19455-01/806-5257/6je9h033k/index.html#compile-4), we can see:  

    For POSIX behavior, compile applications with the -D_POSIX_C_SOURCE flag set >= 199506L. For Solaris behavior, compile multithreaded programs with the -D_REENTRANT flag.
So we should pay more attentions when building multithreaded application on Solaris.

Reference:  
(1) [Compiling a Multithreaded Application](http://docs.oracle.com/cd/E19455-01/806-5257/6je9h033k/index.html);  
(2) [What is the correct way to build a thread-safe, multiplatform C library?](http://stackoverflow.com/questions/15944664/what-is-the-correct-way-to-build-a-thread-safe-multiplatform-c-library)