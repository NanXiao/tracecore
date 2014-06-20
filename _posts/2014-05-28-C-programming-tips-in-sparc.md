---
layout: post
title: C programming tips in SPARC architecture
---

If you are a newbie of C programmers in SPARC architecture (For example, working on Solaris), you should pay attention to the following tips:

(1) By default, SPARC is big-endian (For Endianness, you can refer [http://en.wikipedia.org/wiki/Endianness](http://en.wikipedia.org/wiki/Endianness)). It means for an integer (short, int, long, etc), the MSB will be stored in the lower address, while the LSB will be stored in the higher address.

(2) SPARC requires byte-alignment. It means for a short (2 bytes long) variable, the start address of the variable must be the multiples of 2, while a int (4 bytes long) variable, the start address of the variable must be the multiples of 4. If the address can't satisfy this condition, the application will core dump, and a "Bus Error" will be reported. For this tip, you can refer [Expert C Programming: Deep C Secrets](http://www.e-reading.ws/bookreader.php/138815/Linden_-_Expert_C_Programming:_Deep_C_Secrets.pdf) (Bus Error section, page 163 ~ page 164).

For more SPARC information, you can refer:  
[http://en.wikipedia.org/wiki/SPARC](http://en.wikipedia.org/wiki/SPARC);  
[SPARC Processor Issues](http://docs.oracle.com/cd/E26505_01/html/E27000/hwovr-1.html).