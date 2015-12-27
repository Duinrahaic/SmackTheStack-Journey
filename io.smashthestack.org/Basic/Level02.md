Level02
12/25/15 ------------------
Checking out the files in the directory levels there are 4 files relating to level 2.
````
level02
level02_alt
level02.c
level02_alt.c
```
running level02 we are granted with the phrase 
````
"source code is available in level02.c"
````
so we use vim to view the source code of level02.c to analyze what is happening. 
````c
  1 //a little fun brought to you by bla
  2
  3 #include <stdio.h>
  4 #include <stdlib.h>
  5 #include <signal.h>
  6 #include <unistd.h>
  7
  8 void catcher(int a)                                   <------ So this is where we want to be
  9 {
 10         setresuid(geteuid(),geteuid(),geteuid());
 11     printf("WIN!\n");
 12         system("/bin/sh");
 13         exit(0);
 14 }
 15
 16 int main(int argc, char **argv)                         <------- Begining of main statement. Looking input for an int and char as inputs.
 17 {
 18     puts("source code is available in level02.c\n");    <-------- Prints this when ran
 19
 20         if (argc != 3 || !atoi(argv[2]))        <------ argc has to  not be 3 or not be able to convert an array to an int from argv
 21                 return 1;                       <------ return if either is true not allowing us to to get to the part that matters
 22         signal(SIGFPE, catcher);                <------ signal reports a fatal arithmetic error which results on the catcher to be used.
 23         return abs(atoi(argv[1])) / atoi(argv[2]); <--- If an error is returned here then catcher is signaled.
 24 }
 25
`````
So essentially we need to figure out how to break the function.
A little search about the limitation of datatypes in C shows us the ranges of an Int and a Char

A int stores data from -21474836478 to 2147483647.
A char stores data from -127 to 128.

So in order to make the arithmetic on line 23 of level02.c We need to run it with the following arguments.

./level02 -21473836478 -1
On line 23 the following happens:

(-21473836478)/(-1) = 21473836478

But wait, what is the limitation of the integer? -21474836478 to 2147483647! 21473836478 is outside the limit of an Int Value causing the catcher to trigger and producing a win.

using cat /home/level3/.pass we are granted with level3 access with password OlhCmdZKbuzqngfz
