# cryptolab_job
Lab Project: Exploiting Buffer Overflows in C

Exploiting Buffer Overflows in C
This project will give you some hands-on experience to understand buffer overflows and how to exploit them. You will carry out the project using a virtual machine, on your own computer. In carrying it out, you will have to answer specific questions, given at the bottom, to show that you have followed each of the necessary steps.

Installing and running the Virtual Machine
This project will use a virtual machine in the VirtualBox format. 

The first step is to install VirtualBox on your computer; our VM images have been tested with version 4.3.18. There are specific instructions for installing VirtualBox for computers running Windows, Linux, and Mac 
OSX. 

The next step is to download the virtual machine image, in OVF format, that we will use for the project. It has extension .ova meaning it is an archive with all of the relevant materials in it. The file is 1.3 GB in size. 

The file can be found direct download here https://d396qusza40orc.cloudfront.net/softwaresec/virtual_machine/mooc-vm3.ova

Or a torrent download here https://spark-public.s3.amazonaws.com/softwaresec/virtual_machine/mooc-vm3.ova?torrent

The virtual machine runs a version of Ubuntu Linux.

The SHA1 of the file is 6fb69359d595d81599713c4d02dc8e947d3d6cef. 

Finally, you must import this OVF file, which is called mooc-vm.ova (or mooc-vm2.ova for the smaller version), and run it. To import it, it should be as simple as double-clicking the .ova file. Doing so will start VirtualBox and ask you whether to import it the image. You should then click "import". Alternatively, rather than double clicking the archive file, you can select "File" -> "Import appliance" from the Manager window and select the file. 

Note that if using an earlier version of Virtual Box (4.2.x and earlier), the VM may not import; some have had success importing by disabling the DVD and USB flags.

Having imported the VM, you should see it in your list of VMs. Select it and click "Start". This will open a window running the virtual machine, starting up Ubuntu Linux. When you get to a login screen, use username "seed" and password is "dees" (but without quotes). Then start up a terminal window -- there is an icon in the menu bar at the top for doing so (it looks like a computer monitor).

The vulnerable program
We have placed a C program wisdom-alt.c in the projects/1 directory in the virtual machine. Type cd projects/1 to change into this directory. If you type ls you will see that also in this directory is a compiled version of the program, called wisdom-alt. This executable was produced by invoking gcc -fno-stack-protector -ggdb -m32 wisdom-alt.c -o wisdom-alt (in case you accidentally delete it and need to reproduce it).

Running the program
The program reads data from the stdin (i.e., the keyboard) and writes to stdout (i.e., the terminal). You can run the program by typing ./wisdom-alt on the command prompt. When we do this, we see the following greeting:


1 seed@seed-desktop:~/projects/1$ ./wisdom-alt
2 Hello there
3 1. Receive wisdom
4 2. Add wisdom
5 Selection >

At this point, it is waiting for the user to type something in. Typing the number 1 allows you to "receive wisdom" and typing 2 allows you to "add wisdom". Extending the interaction, suppose we type 1 (and a carriage return).

1 seed@seed-desktop:~/projects/1$ ./wisdom-alt
2 Hello there
3 1. Receive wisdom
4 2. Add wisdom
5 Selection >1
6 no wisdom
7 Hello there
8 1. Receive wisdom
9 2. Add wisdom
10 Selection >

Notice that it outputs no wisdom and then repeats the greeting. Now if we type 2 we can try to add some wisdom; here's what happens:

1 Selection >2
2 Enter some wisdom

Now the program is waiting for the user to type something in. Suppose we type in sleep is important and press return. Then we will get the standard greeting again. If we type 1 at that point we will get the following:

1 Selection >2
2 Enter some wisdom
3 sleep is important 
4 Hello there
5 1. Receive wisdom
6 2. Add wisdom
7 Selection >1
8 sleep is important
9 Hello there
10 1. Receive wisdom

We can continue to add wisdom, by typing 2. For example, if we did this sequence again, with the entry exercise is useful, we would get:

1 Selection >2
2 Enter some wisdom
3 exercise is useful
4 Hello there
5 1. Receive wisdom
6 2. Add wisdom
7 Selection >1
8 sleep is important
9 exercise is useful
10 Hello there

We can keep doing this as long as we like. We can terminate interacting with the program by typing control-D.

Crash!
This program is vulnerable to a buffer overflow. It is easy to see there is a problem, by typing in something other than 1 or 2. For example, type in 156.

1 seed@seed-desktop:~/projects/1$ ./wisdom-alt
2 Hello there
3 1. Receive wisdom
4 2. Add wisdom
6
7 Selection >156

Segmentation fault
In fact, the program has (at least) two vulnerabilities; the above is demonstrating one of them, but there is one other. Your job in this lab is to find and exploit both vulnerabilities. The lab will guide you through steps to do so, and you will answer questions in the on-line quiz to show that you took each of these steps.

Exploiting the program
We are now going to show you some tools you'll need to exploit this program.

Entering binary data
To exploit the program, you will need to enter non-printable characters, i.e., binary data. To input binary data to the program, use the following command line instead:

1 ./runbin.sh

Then we can type in binary-format strings (e.g., with hex escaping). For example:

1 seed@seed-desktop:~/projects/1$ ./runbin.sh
2 Hello there
3 1. Receive wisdom
4 2. Add wisdom
5 Selection >2
6 Enter some wisdom
7 \x41\x41
8 Hello there
9 1. Receive wisdom         
10 2. Add wisdom
11 Selection >1
12 AA

In the above, \x41\x41 represents two bytes, defined in hexadecial format. 41 in hex is 65 in decimal, which in ASCII is the character A. As a result, when we ask for wisdom, the program prints AA. Entering something like \x07 would be a byte 7. This is not a printable character, but is the "bell". So when it "prints," you would actually hear a sound (if sound were enabled on this VM).

To exploit the program, you will have to enter sequences of binary bytes that contain addresses, which are 4-byte (i.e., 32-bit) words on the VM. The x86 architecture is "little-endian", meaning that the bytes in a word are stored from least significant to most significant. That means that the hexadecimal address 0xabcdef00 would be entered as individual bytes in reverse order, i.e., \x00\xef\xcd\xab. 

Note: runbin.sh is a shell script that is just a wrapper around the following code:

1 while read -r line; do echo -e $line; done | ./wisdom-alt 

This is what is converting the hex digits into binary before passing them to the wisdom-alt program. When carrying out the lab, please use the runbin.sh program, and not the above code directly, or your answers may be slightly off, as discussed at the end.

Using GDB
To exploit the program, you will have to learn some information about how it is laid out in memory. You can find out this information using the gdb program debugger. You can attach gdb to your running program, and then use it to print information about the state of that program, and step through executions of that program.

To attach gdb to wisdom-alt, you should first invoke ./runbin.sh, and then, in a separate terminal, from the projects/1 directory invoke the following line:

1 gdb -p `pgrep wisdom-alt`

The -p option to gdb tells it to attach to a running program with the process ID (PID) given to the option. The command pgrep wisdom-alt searches the process table to find the PID of the wisdom-alt program; this PID is then fed as the argument to -p. Be warned: If you have multiple wisdom-alt programs running, you may not attach to the one you expect! Make sure they are all killed (perhaps by killing and restarting the terminals you started them in) if you run into trouble. Also, be sure you use backquotes around pgrep wisdom-alt and not forward quotes.

Once you have connected to the process, you can start using gdb commands to start examining its state and controlling it. For example:

1 seed@seed-desktop:~/projects/1$ gdb -p `pgrep wisdom-alt`
2
3 GNU gdb 6.8-debian
4 Copyright (C) 2008 Free Software Foundation, Inc.License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
5 This is free software: you are free to change and redistribute it.
6 There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
7 and "show warranty" for details.
8 This GDB was configured as "i486-linux-gnu".
9 Attaching to process 29727
10 Reading symbols from /home/seed/projects/1/wisdo-alt...done.
11 Reading symbols from /lib/tls/i686/cmov/libc.so.6...done.
12 Loaded symbols for /lib/tls/i686/cmov/libc.so.6
13 Reading symbols from /lib/ld-linux.so.2...done.
14 Loaded symbols for /lib/ld-linux.so.2
15 0xb7fe1430 in __kernel_vsyscall ()
16 (gdb) 
17

This shows starting gdb and attaching it to a running wisdom-alt process. Then the gdb command prompt comes up. At this point, the execution of that program is paused, and we can start entering commands. For example:

1 (gdb) break wisdom-alt.c:100
2 Breakpoint 1 at 0x80487ea: file wisdom-alt.c, line 100.
3 (gdb) cont
4 Continuing.

Here we enter a command to set a breakpoint at line 100 of wisdom-alt.c. Then we enter command cont (which is short for continue) to tell the program to resume its execution. In the other terminal, running wisdom-alt we enter 2 and press return. This causes execution to reach line 100, so the breakpoint fires, and the gdb command prompt comes up again, pausing the program in the process.

1 Breakpoint 1, main () at wisdom-alt.c:100
2 100            int s = atoi(buf);
3 (gdb) next
4 101            fptr tmp = ptrs[s];
5 (gdb) print s
6 $1 = 2
7 (gdb) print &r
8 $2 = (int *) 0xbffff530
9 (gdb) cont
10 Continuing.

Then, we control the program by stepping using "next", which executes the current line of code, proceeding to the next. Then we print the contents of variable s with "print", and it displays the value we entered in the other terminal. Then we continue execution by entering "cont". In the other terminal we see the prompt to enter some wisdom.

When you are done working with gdb (perhaps when you've terminated the other program), just type quit to exit.

The basic GDB commands you will want to use are those we have already demonstrated: setting break points, stepping through execution, and printing values. You would find it helpful to be familiar with the "print", "break", and "step" commands. 

Questions
You are now ready to start your process of developing an exploit. 

The first step is to identify where the vulnerabilities are. To do that you will have to look through the code of wisdom-alt.c. You can do this by using an editor on Linux virtual machine, like vi or emacs, both of which are installed. 

After looking over the code to see how it works, answer the following four questions.

There is a stack-based overflow in the program. What is the name of the stack-allocated variable that contains the overflowed buffer?

A: ptrs

Consider the buffer you just identified: Running what line of code will overflow the buffer? (We want the line number, not the code itself.)

A: 101

There is another vulnerability, not dependent at all on the first, involving a non-stack-allocated buffer that can be indexed outside its bounds (which, broadly construed, is a kind of buffer overflow). What variable contains this buffer?

A: 

Consider the buffer you just identified: Running what line of code overflows the buffer? (We want the number here, not the code itself.)

Now use GDB to examine the running the program and answer the following questions. These questions are basically going to walk you through constructing an exploit of the non-stack-based overflow vulnerability you just identified. We will do less "hand holding" when asking about exploiting the stack-allocated buffer.

Once you have answers for all of the questions here, you can enter them on the Coursera site for full credit.

Important: When carrying out the lab, you must follow the instructions given above exactly for running the program (using runbin.sh) and using GDB (attaching to wisdom-alt in a separate terminal, and not running gdb ./wisdom-alt) or else the answers you get may not match the ones we are expecting. In particular, the addresses of stack variables may be different. These addresses might also be different if you have altered any environment variables in the Ubuntu terminals. To confirm that things are as they should be, recall the GDB interaction above, where we print the address &r with the result being 0xbffff530 -- if you are not getting that result when you reproduce that interaction then something is wrong. You should restart fresh terminals and begin from scratch, following the instructions exactly.

On to the first exploit:

What is the address of buf (the local variable in the main function)? Enter the answer in either hexadecimal format (a 0x followed by 8 "digits" 0-9 or a-f, like 0xbfff0014) or decimal format. Note here that we want the address of buf, not its contents.

What is the address of ptrs (the global variable) ? As with the previous question, use hex or decimal format.

What is the address of write_secret (the function) ? Use hex or decimal.

What is the address of p (the local variable in the main function) ? Use hex, or decimal format.

What input do you provide to the program so that ptrs[s] reads (and then tries to execute) the contents of local variable p instead of a function pointer stored in the buffer pointed to by ptrs? You can determine the answer by performing a little arithmetic on the addresses you have already gathered above -- be careful that you take into account the size of a pointer when doing pointer arithmetic. If successful, you will end up executing the pat_on_back function. Enter your answer as an unsigned integer.

What do you enter so that ptrs[s] reads (and then tries to execute) starting from the 65th byte in buf, i.e., the location at buf[64]? Enter your answer as an unsigned integer.

What do you replace \xEE\xEE\xEE\xEE with in the following input to the program (which due to the overflow will be filling in the 65th-68th bytes of buf) so that the ptrs[s] operation executes the write_secret function, thus dumping the secret? (Hint: Be sure to take endianness into account.) 771675175\x00AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\xEE\xEE\xEE\xEE

Now let's consider the other vulnerability:

Suppose you wanted to overflow the wis variable to perform a stack smashing attack. You could do this by entering 2 to call put_wisdom, and then enter enough bytes to overwrite the return address of that function, replacing it with the address of write_secret. How many bytes do you need to enter prior to the address of write_secret?

To work out the answer here, you might find it useful to use the GDB backtrace command, which prints out the current stack, and the x command, which prints a "hex dump" of the bytes at a given address. For example, by typing x/48xw $esp you would print out 48 words (the w) in hexadecimal format (the x) starting at the address stored in register $esp.

Answer the following questions:

1. There is a stack-based overflow in the program. What is the name of the stack-allocated variable that contains the
overflowed buffer? (enter the answer in math expresion)

2. Consider the buffer you just identified: Running what line of code will overflow the buffer? (We want the line number, not
the code itself.)

3. There is another vulnerability, not dependent at all on the first, involving a non-stack-allocated buffer that can be indexed
outside its bounds (which, broadly construed, is a kind of buffer overflow). What variable contains this buffer? (enter the answer in math expression)

4. Consider the buffer you just identified: Running what line of code overflows the buffer? (We want the number here, not
the code itself.)

5. What is the address of buf (the local variable in the main function)? Enter the answer in either hexadecimal format (a Ox
followed by 8 "digits" 0-9 or a-f, like Oxbff f 0014) or decimal format. Note here that we want the address of buf, not
its contents.

6. What is the address of ptrs (the global variable) ? As with the previous question, use hex or decimal format.

7. What is the address of write _ secret (the function) ? Use hex or decimal.

8. What is the address of p (the local variable in the main function) ? Use hex, or decimal format.

9. What input do you provide to the program so that ptrs [s] reads (and then tries to execute) the contents of stack
variable p instead of a function pointer stored in the buffer pointed to by ptrs? As a hint, you can determine the answer
by performing a little arithmetic on the addresses you have already gathered. If successful, you will end up executing the
pat _on_back function. Provide the smallest positive integer.

10. What do you enter so that ptrs [s] reads (and then tries to execute) starting from the 65th byte in buf, i.e., the location
at buf [64] ? Enter your answer as an (unsigned) integer.

11. What do you replace \color{red}{\verbl\xEE\xEE\xEE\xEEl} with in the following input to the program (which due to the
overflow will be filling in the 65th-68th bytes of\color{red}{\verblbufl}) so that the \color{red}{\verblptrs[s)l} operation
executes the \color{red}{\verblwrite_secretl} function, thus dumping the secret? (Hint: Be sure to take endianness into
account.) \color{red}{\verb|771675175\x00AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\\xEE\xEE\xEE\xEE|}

12. Suppose you wanted to overflow the wis variable to perform a stack smashing attack. You could do this by entering 2 to
call put _ vi sdom, and then enter enough bytes to overwrite the return address of that function, replacing it with the
address of vrite_ secret. How many bytes do you need to enter prior to the address of write _ secret?
