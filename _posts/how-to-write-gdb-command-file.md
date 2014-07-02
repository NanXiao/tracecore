---
layout: post
title: 如何写gdb命令脚本（command file）
---
作为UNIX/Linux下使用广泛的调试器，gdb不仅提供了丰富的命令，还引入了对脚本的支持：一种是对已存在的脚本语言支持，比如python，用户可以直接书写python脚本，由gdb调用python解释器执行；另一种是是命令脚本（command file），用户可以在脚本中书写gdb已经提供的或者自定义的gdb命令，再由gdb执行。在这篇文章里，我会介绍一下如何写gdb的命令脚本。  

(一)自定义命令  
gdb支持用户自定义命令，格式是：  

	define commandName  
		statement  
		......  
	end  
其中`statement`可以是任意gdb命令。此外自定义命令还支持最多10个输入参数：$arg0，$arg1 ......$arg9，并且还用$argc来标明一共传入了多少参数。  
下面结合一个简单的C程序（test.c），来介绍如何写自定义命令：  

	#include <stdio.h>

	int global = 0;
	
	int fun_1(void)
	{
		return 1;
	}
	
	int fun_a(void)
	{
		int a = 0;
		printf("%d\n", a);
	}
	
	int main(void)
	{
		fun_a();
		return 0;
	}

首先编译成可执行文件：  

	gcc -g -o test test.c
接着用gdb进行调试：  

	[root@linux:~]$ gdb test
	GNU gdb (GDB) 7.6
	Copyright (C) 2013 Free Software Foundation, Inc.
	License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
	This is free software: you are free to change and redistribute it.
	There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
	and "show warranty" for details.
	This GDB was configured as "x86_64-unknown-linux-gnu".
	For bug reporting instructions, please see:
	<http://www.gnu.org/software/gdb/bugs/>...
	Reading symbols from /data2/home/nanxiao/test...done.
	(gdb) b fun_a
	Breakpoint 1 at 0x4004d7: file test.c, line 12.
	(gdb) r
	Starting program: /data2/home/nanxiao/test
	
	Breakpoint 1, fun_a () at test.c:12
	12              int a = 0;
	(gdb) bt
	#0  fun_a () at test.c:12
	#1  0x0000000000400500 in main () at test.c:18
可以看到使用`bt`（`backtrace`缩写）命令可以打印当前线程的调用栈。我们的第一个自定义命令就是也实现一个`backtrace`功能：  

	define mybacktrace
		bt
	end

怎么样？简单吧，纯粹复用gdb提供的命令。下面来验证一下：  

	(gdb) define mybacktrace
	Type commands for definition of "mybacktrace".
	End with a line saying just "end".
	>bt
	>end
	(gdb) mybacktrace
	#0  fun_a () at test.c:12
	#1  0x0000000000400500 in main () at test.c:18
功能完全正确！  
接下来定义一个赋值命令，把第二个参数的值赋给第一个参数：  

	define myassign
		set var $arg0 = $arg1
	end
执行一下：  

	(gdb) define myassign
	Type commands for definition of "myassign".
	End with a line saying just "end".
	>set var $arg0 = $arg1
	>end
	(gdb) myassign global 3
	(gdb) p global
	$1 = 3
可以看到`global`变量的值变成了`3`。  
对于自定义命令来说，传进来的参数只是进行简单的文本替换，所以你可以传入赋值的表达式，甚至是函数调用：  

	(gdb) myassign global fun_1()
	(gdb) p global
	$2 = 1
可以看到`global`变量的值变成了`1`。  
除此以外，还可以为自定义命令写帮助文档，也就是执行`help`命令时打印出的信息：  

	document myassign
		assign the second parameter value to the first parameter
	end
执行`help`命令：  

	(gdb) document myassign
	Type documentation for "myassign".
	End with a line saying just "end".
	>assign the second parameter value to the first parameter
	>end
	(gdb) help myassign
	assign the second parameter value to the first parameter
可以看到打印出了`myassign`的帮助信息。  
