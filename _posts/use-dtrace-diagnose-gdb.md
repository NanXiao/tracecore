---
layout: post
title: Use DTrace to diagnose gdb issues
---

A few days ago, I installed the newest 64-bit gdb program (version 7.7.1) on Solaris 10 (X86_64 platform) to debug programs. After playing with the gdb a day, I found 2 issues about gdb:  
(1) The "`set follow-fork-mode child`" command doesn't take effect. By default, after the parent process forks the child process, the gdb will track the parent process, so this command can make gdb begin to follow the child process. But this command works OK on Linux.  
(2) The gdb can't parse the 32-bit application core dump file. Per my understanding, the 64-bit gdb should parse both 32-bit and 64-bit core dump files.

I think these are 2 bugs, so I reported them to the gdb:https://sourceware.org/bugzilla/show_bug.cgi?id=17044 and https://sourceware.org/bugzilla/show_bug.cgi?id=17045. But unfortunately, there is no any responses from the gdb. Suddenly I hit upon an idea: since I worked on Solaris, why not use DTrace? Yes, DTrace. DTrace is a very cool application which can trace application, and tell you the function flow of the application. So I start working on it at once!

(1) The "set follow-fork-mode child" command doesn't take effect.  
Firstly, I wrote a simple C test program:

    #include <stdio.h>
    #include <sys/types.h>
    #include <unistd.h>
    
    int main(void) {
        pid_t pid;
    
        pid = fork();
        if (pid < 0)
        {
            exit(1);
        }
        else if (pid > 0)
        {
            exit(0);
        }
        printf("hello world\n");
        return 0;
    }
Then I wrote the DTrace script (set_follow_fork_mode.d) to diagnose gdb. Because I hadn't read gdb source code before and I didn't know the function flow of the gdb, I would like to know how the functionw were called and executed.  
The script was like the following, and it recorded every entry and return of the function:

    #!/usr/sbin/dtrace -Fs
    pid$target:gdb::entry,  
    pid$target:gdb::return
    {
    }
The "-F" option's effect is "Coalesce trace output by identifying function entry  and return. Function  entry  probe reports are indented and their output is prefixed with ->. Function return  probe reports are unindented and their output is prefixed with <- . " so it can tell us how the gdb was executed.

I began to use gdb to debug the program:

    bash-3.2# ./gdb -data-directory ./data-directory /data/nan/a
    GNU gdb (GDB) 7.7.1
    Copyright (C) 2014 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
    and "show warranty" for details.
    This GDB was configured as "x86_64-pc-solaris2.10".
    Type "show configuration" for configuration details.
    For bug reporting instructions, please see:
    <http://www.gnu.org/software/gdb/bugs/>.
    Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.
    For help, type "help".
    Type "apropos word" to search for commands related to "word"...
    Reading symbols from /data/nan/a...done.
Before executing "`set follow-fork-mode child`" command, I ran the DTrace script (the 4408 is the gdb process ID):  

    `./set_follow_fork_mode.d -p 4408 > a.txt`

Then I execute the "`set follow-fork-mode child`" command in gdb:

    (gdb) set follow-fork-mode child
Oh, the output of DTrace script was very large, more than 900 lines, but I found the following function entries and returns:

      CPU FUNCTION        
      ......
      5                      -> do_set_command    
      5                        -> do_sfunc        
      5                          -> empty_sfunc   
      5                          <- empty_sfunc   
      5                        <- do_set_command
      ......

From the name of `empty_sfunc`, it seemed that gdb did nothing, right? I began to search the code in gdb:  
a) The `empty_sfunc` is like this:

    static void
    empty_sfunc (char *args, int from_tty, struct cmd_list_element *c)
    {
    }

It really did nothing!

and it was called in `add_set_or_show_cmd`:

    static struct cmd_list_element *
    add_set_or_show_cmd (const char *name,
    		     enum cmd_types type,
    		     enum command_class class,
    		     var_types var_type,
    		     void *var,
    		     char *doc,
    		     struct cmd_list_element **list)
    {
      struct cmd_list_element *c = add_cmd (name, class, NULL, doc, list);
    
      gdb_assert (type == set_cmd || type == show_cmd);
      c->type = type;
      c->var_type = var_type;
      c->var = var;
      /* This needs to be something besides NULL so that this isn't
         treated as a help class.  */
      set_cmd_sfunc (c, empty_sfunc);
      return c;
    }
and the `set_cmd_sfunc` was like this:

    void
    set_cmd_sfunc (struct cmd_list_element *cmd, cmd_sfunc_ftype *sfunc)
    {
      if (sfunc == NULL)
        cmd->func = NULL;
      else
        cmd->func = do_sfunc;
      cmd->function.sfunc = sfunc; /* Ok.  */
    }
so from above, I knew the `cmd`'s `func` and `function.sfunc` both set to `empty_sfunc`.  
b) Then I searched "`follow-fork-mode`" in source code, it appeared in `add_setshow_enum_cmd` function:

    add_setshow_enum_cmd ("follow-fork-mode", class_run,
			follow_fork_mode_kind_names,
			&follow_fork_mode_string, _("\
    Set debugger response to a program call of fork or vfork."), _("\
    Show debugger response to a program call of fork or vfork."), _("\
    A fork or vfork creates a new process.  follow-fork-mode can be:\n\
      parent  - the original process is debugged after a fork\n\
      child   - the new process is debugged after a fork\n\
    The unfollowed process will continue to run.\n\
    By default, the debugger will follow the parent process."),
    			NULL,
    			show_follow_fork_mode_string,
    			&setlist, &showlist);
The fourth input parameter from bottom was `NULL`, and it was the `set_func` of the `cmd`. The `add_setshow_enum_cmd` is like this:

    void
    add_setshow_enum_cmd (const char *name,
    		      enum command_class class,
    		      const char *const *enumlist,
    		      const char **var,
    		      const char *set_doc,
    		      const char *show_doc,
    		      const char *help_doc,
    		      cmd_sfunc_ftype *set_func,
    		      show_value_ftype *show_func,
    		      struct cmd_list_element **set_list,
    		      struct cmd_list_element **show_list)
    {
      struct cmd_list_element *c;
    
      add_setshow_cmd_full (name, class, var_enum, var,
    			set_doc, show_doc, help_doc,
    			set_func, show_func,
    			set_list, show_list,
    			&c, NULL);
      c->enums = enumlist;
    }

`add_setshow_enum_cmd` called `add_setshow_cmd_full`, and what did `add_setshow_cmd_full` do?

    static void
    add_setshow_cmd_full (const char *name,
    		      enum command_class class,
    		      var_types var_type, void *var,
    		      const char *set_doc, const char *show_doc,
    		      const char *help_doc,
    		      cmd_sfunc_ftype *set_func,
    		      show_value_ftype *show_func,
    		      struct cmd_list_element **set_list,
    		      struct cmd_list_element **show_list,
    		      struct cmd_list_element **set_result,
    		      struct cmd_list_element **show_result)
    {
        ......
        set = add_set_or_show_cmd (name, set_cmd, class, var_type, var,
			     full_set_doc, set_list);
        set->flags |= DOC_ALLOCATED;
        
        if (set_func != NULL)
            set_cmd_sfunc (set, set_func);
        ......
    }

Form the above code analysis, I got the following conclusion: by default, the `cmd`'s `function.sfunc` was `empty_sfunc`, it would be set only if the `set_func` was not `NULL`. Because the default value of "`follow-fork-mode`" command's `set_func` was `NULL`, this command wouldn't take effect by default.  

Another question, why did it work OK on Linux? After further investigating the code, I found the Linux had a customized function for supporting this command: `linux_child_follow_fork`.

So the conclusion is: by default, the gdb doesn't support "`set follow-fork-mode child`" command, it needs the diffent platforms implement the function themselves.