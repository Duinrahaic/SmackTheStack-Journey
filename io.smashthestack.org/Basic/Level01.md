#Level01
12/24/15
Logged into the io.smasthstack.org server with putty. 
Started as a level01 user meaing I only have access as a generic user and cannot access level02 or higher levels.

cd/levels brought me to all the levels that are on this server
using ./ I am able to execute (run) a level. For example ./level01 gives me the first level

Executing ./level01 for the first time we are confronted with a three digit passcode.

Attemps 001,123,000, and 111 did not work. Was just trying luck of the draw on that one. 

After some research I was able to see that there was a debugger (gdb) where I can see the code run without modifying or being able to view the soruce code.

         gdb level01        <--- Runs GDB on specific file
         (gdb)       <----------- Now shows I'm in the debugger
         (gdb) run level01     <- Executes (runs) level01 as if I were outside the debugger and using ./level01
         (gdb) disass main     <- All programs have some form of a main loop so were are able to disassemble the main program.

Dump of assembler code for function main:
```
         0x08048080 <+0>:     push   $0x8049128
         0x08048085 <+5>:     call   0x804810f <puts>             <---- First command to put a phrase on the screen, assuming its "Enter the 3 digit passcode to enter:"
         0x0804808a <+10>:    call   0x804809f <fscanf>           <---- Fscanf is a c command to read the user input. There is no limitation so its just waiting till you hit enter.
         0x0804808f <+15>:    cmp    $0x10f,%eax                  <---- cmp is short for compare. Assuming its comparing the user input with the correct passphrase
         0x08048094 <+20>:    je     0x80480dc <YouWin>           <---- je is a jump command in assembly. Where if the solution is correct we are granted access.
         0x0804809a <+26>:    call   0x8048103 <exit>             <---- Exit program.
```
The gdb has a command called break point where is pauses the program, and we can view information on it. Taking note of the cmps address we add a breakpoint using the following command:
      (gdb) b *0x0804808f   <----- * is a pointing to the point where the program is comparing user input to that of the correct password
Result:
      
      Breakpoint 1 at 0x804808f
Now we run the program and we are confronted with:
      (gdb) run level01
      Starting program: /levels/level01 level01
      Enter the 3 digit passcode to enter: 999
      
      Breakpoint 1, 0x0804808f in main ()

Now we can figure out information on the info registers found at this point. Info registers displays the contents of general-purpose processor registers.

(gdb) i r <----- Also can use "info registers" and it will do the same thing.

      eax            0x3e7    999
      ecx            0xbfffed68       -1073746584
      edx            0x1000   4096
      ebx            0x3e7    999
      esp            0xbffffd6c       0xbffffd6c
      ebp            0x0      0x0
      esi            0xbfffed6c       -1073746580
      edi            0x0      0
      eip            0x804808f        0x804808f <main+15>
      eflags         0x282    [ SF IF ]
      cs             0x73     115
      ss             0x7b     123
      ds             0x7b     123
      es             0x7b     123
      fs             0x0      0
      gs             0x0      0

Things to note: 
*eip has the same address as our breakpoint/the cmp command. So I assume this means "Execution in progress".
*eax and ebx holds the value 999 which is what i entered as a test passcode.
*We can see the values 115 and 123... could those be the answer? Wait, 123 wasn't because I used that earlier tring to do brute force.

      (gdb) Quit <---- Quits the program (it asks y or n before exit)

Trying 115... doesn't work! Back to the gdb!

We can try and get a little more information from the processor to try and see what else is going on. 
      x/AAi BBBB  tells the debugger "Hey tell me the next #(AA) set of assembler instructions (i) at this address (BBBB)!"
so we want the next 20 instructions at the eip.
````
      (gdb) x/20i $eip              <--- $eip meaing the current value or address at eip. A dollar symbol "$" denotes current value of address counter in the currently active segment.
      => 0x804808f <main+15>: cmp    $0x10f,%eax            <--- Compare
         0x8048094 <main+20>: je     0x80480dc <YouWin>     <--- True go to... somewhere outside the 20 instructions
         0x804809a <main+26>: call   0x8048103 <exit>       <--- Else exit          
         0x804809f <fscanf>:  sub    $0x1000,%esp                                   
         0x80480a5 <fscanf+6>:        mov    $0x3,%eax                              
         0x80480aa <fscanf+11>:       mov    $0x0,%ebx                                 
         0x80480af <fscanf+16>:       mov    %esp,%ecx                              
         0x80480b1 <fscanf+18>:       mov    $0x1000,%edx                           
         0x80480b6 <fscanf+23>:       int    $0x80                                  
         0x80480b8 <fscanf+25>:       mov    %esp,%esi                              
         0x80480ba <fscanf+27>:       xor    %ebx,%ebx                              
         0x80480bc <skipwhite>:       xor    %eax,%eax      
         0x80480be <skipwhite+2>:     lods   %ds:(%esi),%al
         0x80480bf <skipwhite+3>:     cmp    $0x20,%al
         0x80480c1 <skipwhite+5>:     je     0x80480bc <skipwhite>
         0x80480c3 <doit>:    sub    $0x30,%al
         0x80480c5 <doit+2>:  cmp    $0x9,%al
         0x80480c7 <doit+4>:  ja     0x80480d3 <exitscanf>
         0x80480c9 <doit+6>:  imul   $0xa,%ebx,%ebx
         0x80480cc <doit+9>:  add    %eax,%ebx
````
So I believe the answer is contained in at %eax, so I can use p/ to find out what its using at that address. 
      (gdb) p/d $eax
      $1 = 999
Wait, isn't that what I put into it?
      (gdb) p/d 0x10f
      $2 = 271
Wow thats a new number lets try that...

      level1@io:/levels$ ./level01
      Enter the 3 digit passcode to enter: 271
      Congrats you found it, now read the password for level2 from /home/level2/.pass

Success!
````
         sh-4.2$ cat /home/level2/.pass granted me the passcode: ~~XNWFtWKWHhaaXoKI~~ which is the passcode to level02
````
