# Challenge #2 - Collision

## Challenge Description:

> Daddy told me about cool MD5 hash collision today.
> 
> I wanna do something like that too!
> ```bash
> ssh col@pwnable.kr -p2222 (pw:guest)
> ```

---

## Pre-Analysis

First let's check what files we have here...

```bash
col@ubuntu:~$ ls -l
total 24
-r-xr-sr-x 1 root col_pwn 15164 Mar 26 13:13 col
-rw-r--r-- 1 root root      589 Mar 26 13:13 col.c
-r--r----- 1 root col_pwn    26 Apr  2 08:58 flag
```

Okay so we have an executable and a corresponding C file, as well as the flag we are after though we have no read permissions over it.

## Static Analysis

Let's look at `col.c`:
```C
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
        int* ip = (int*)p;
        int i;
        int res=0;
        for(i=0; i<5; i++){
                res += ip[i];
        }
        return res;
}

int main(int argc, char* argv[]){
        if(argc<2){
                printf("usage : %s [passcode]\n", argv[0]);
                return 0;
        }
        if(strlen(argv[1]) != 20){
                printf("passcode length should be 20 bytes\n");
                return 0;
        }

        if(hashcode == check_password( argv[1] )){
                setregid(getegid(), getegid());
                system("/bin/cat flag");
                return 0;
        }
        else
                printf("wrong passcode.\n");
        return 0;
}
```

OK! so the program does something like so:

1. It checks if the user put a command-line argument
2. If not, exit program
3. It then checks the length of the provided argument
4. If it's not 20 characters, exit program
5. Then it checks if our argument after the `check_password` function is equal to the value `0x21DD09EC` (Decimal: `568134124`)
6. If it is equal, we win!
7. If not, exit program

And let's look at the `check_password` function in more detail:

1. It creates `ip` which is the pointer to the start of the password casted to an `int *`
2. It creates `i` which serves as an index
3. It creates `res` which will be the hash value that's returned
4. Then, it goes through a loop 5 times in total
5. For each iteration it adds the value of `ip` at offset `i` to `res`
6. It returns `res`

### But what does it really do?

The thing that's really happening behind the scenes is a clever trick with pointers in C.

In C, pointers hold some memory address, and we know how many bytes to read by the pointer type!
For example:

```C
char * // Generally 1 byte
int *  // 4 bytes
double * // 8 bytes
```

So when the password's `char *` is casted to `int *` (On this line: `int* ip = (int*)p;`) it is treated as 5 ints which are then summed up inside the loop!

## Plan

Okay so essentially what we want is a payload of 20 characters that when read as 5 consecutive ints and summed up we get `0x21DD09EC` (Decimal: `568134124`).
So let's divide the hash by 5!

```
>>> 568134124 / 5
113626824.8 # Not a whole number!
```

Hmm, the value is not divisible by 5... Another route we can take is choose some identical 4 values and make the 5th one be the remainder!

```
>>> 568134124 - 4 * (0x01010101) # 0x01010101 is a simple number, any could work really
500762088
>>> hex(500762088)
'0x1dd905e8'
```

Great! So our payload will be a non-printable string which is: `"\x01\x01\x01\x01" * 4 + "\x1d\xd9\x05\xe8"`!

But we need to remember to use little endian! So let's switch it to little endian and we get: `"\x01\x01\x01\x01" * 4 + "\xe8\x05\xd9\x1d"`

## Exploit

Let's use python2 and try our payload:

```bash
col@ubuntu:~$ ./col $(python2 -c 'print("\x01\x01\x01\x01" * 4 + "\xe8\x05\xd9\x1d")')
<collision_flag_here>
```

### And we are successful!
