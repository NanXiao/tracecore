---
layout: post
title: Socket programming tips on Solaris
---

I sponsored a topic in stackoverflow.com, and hoped the programmers can share the socket programming tips on different UNIX flavors. But unfortunately, the responders are few. So I can only share my socket programming tips on Solaris at here (the Chinese version can be found [there](http://blog.csdn.net/yigeshouyiren/article/details/16358991)):  

1. Use the following link options: "-lresolv -lnsl -lsocket";  
2. Solaris doesn't provide socket options: SO_SNDTIMEO and SO_RCVTIMEO([Why does Solaris OS define SO_SNDTIMEO and SO_RCVTIMEO socket options in header file which actually not support by kernel?](http://stackoverflow.com/questions/15264801/why-does-solaris-os-define-so-sndtimeo-and-so-rcvtimeo-socket-options-in-header));
3. In SCTP programming, must call bind() before calling sctp_bindx()([sctp_bindx(Solaris sctp library) always return "Invalid argument"](http://stackoverflow.com/questions/16476770/sctp-bindx-solaris-sctp-library-always-return-invalid-argument));
4. When calling shutdown() on a listen socket, it will cause ENOTCONN error([Why shutdown a socket can't let the select() return?](http://stackoverflow.com/questions/18352039/why-shutdown-a-socket-cant-let-the-select-return)).