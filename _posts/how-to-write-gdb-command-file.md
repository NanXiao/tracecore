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