---
layout: post
title: A awful Project
---
A simple server program whose function is just receiving protocol packets from clients, then paring them, and inserting the paring results into database. Why is the implementation so complicated and awful?  

(1) For every protocol message, the server need listen to a specified port. Now, there are only 5 protocols, how if we need support 100 or 10000 protocol? Do we need to listen 100 or 10000 ports? Yes, We can't wait for that day. Because if we need support 10000 protocols, we have became Bill Gates.  

(2) every protocol message has a different header.  

(3) A lot of copy-paste code; a lot of dead code.

(4) Many functions are so long (more than 1000 lines) that I don't know whether they can be maintained after 5 years. 

(5) Depend on 32-bit/64-bit architecture. 

(6) The client and server is coupled so closely that upgrading one program must consider the other. 

I think we can use a unified header for every protocol and server only listen to one port. Every message arrives the server with the same header, so this can avoid many duplicated codes. What the server needs to do is use "switch ... case" to parse every message according to the header. This can also decouple client-server programs.

A good architecture is the base of the house. If there is a serious defection   in base, I am not sure whether the house can be stable.