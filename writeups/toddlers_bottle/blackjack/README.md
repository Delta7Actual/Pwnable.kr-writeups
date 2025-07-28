# Challenge #10 - Blackjack

## Challenge Description:
> Hey! check out this C implementation of blackjack game!
> I found it online
> * http://cboard.cprogramming.com/c-programming/114023-simple-blackjack-program.html
>
> I like to give my flags to millionares.
> how much money you got?
>
> ```bash
> ssh blackjack@pwnable.kr -p2222 (pw:guest)
> ```

## Pre-Analysis

Right, let's see what we have to work with...

```bash
blackjack@ubuntu:~$ ls -l
total 56
-r-xr-x--- 1 root root 22153 Apr  1 14:40 blackjack
-rw-r--r-- 1 root root 26095 Apr  1 14:40 blackjack.c
-rw-r--r-- 1 root root   106 Apr  1 14:40 readme
```

Ok so we can't execute the executable ourselves as we don't have the needed permissions, Let's see what the `readme` has to add:

```bash
blackjack@ubuntu:~$ cat readme
once you connect to port 9009, the "blackjack" binary will be executed under asm_pwn privilege. get flag.
```

So to run the executable we need to use `netcat`, good to know!

## Static Analysis

The `blackjack.c` file is quite long and has many irrelevant parts to our use. The part thats interesting to us is the handling of the users money as we want to get over 1 million.

```C
// ... (Start of file)

int cash = 500; /* this appears to be our money */

// ...

/* The randcard() function uses random correctly, no vulnerabilities I can see... */

// ...

int betting() //Asks user amount to bet
{
 printf("\n\nEnter Bet: $");
 scanf("%d", &bet);

 if (bet > cash) //If player tries to bet more money than player has
 {
        printf("\nYou cannot bet more money than you have.");
        printf("\nEnter Bet: ");
        scanf("%d", &bet);
        return bet;
 }
 else return bet;
} // End Function

// ...
```

There we go! the `betting()` function checks if our bet is larger than the amount of money we have but crucially it neglects to check if our bet is even positive at all...

## Plan

Due to the oversight in the `betting()` function we can input NEGATIVE million and then LOSE! Then, as the program goes to subtract the bet from our `cash` it will in turn add 1 million to it!

## Exploit

Let's go ahead and implement it:

```bash
blackjack@ubuntu:~$ nc 0 9009
// Enters game
// Input y to play
// Input 1 to begin game
// Input -999999999 (or some other large negative bet)
// Try your best to lose the match
// Input y to play again

<blackjack_flag_here>


Cash: $2000000498
-------
|C    |
|  2  |
|    C|
-------

Your Total is 2

The Dealer Has a Total of 7

Enter Bet:
```

## And we are successful!
