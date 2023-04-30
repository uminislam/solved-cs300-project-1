Download Link: https://assignmentchef.com/product/solved-cs300-project-1
<br>



<strong>3.21    </strong>The Collatz conjecture concerns what happens when we take any positive integer <em>n </em>and apply the following algorithm: <em>n </em>= ×


<em>n</em>/2,                  if n is even

<ul>

 <li><em>n </em>1, if n is odd</li>

</ul>

The conjecture states that when this algorithm is continually applied, all positive integers will eventually reach 1. For example, if <em>n </em>= 35, the sequence is

35, 106, 53, 160, 80, 40, 20, 10, 5, 16, 8, 4, 2, 1

Write a C program using the fork() system call that generates this sequence in the child process. The starting number will be provided from the command line. For example, if 8 is passed as a parameter on the command line, the child process will output 8, 4, 2, 1. Because the parent and child processes have their own copies of the data, it will be necessary for the child to output the sequence. Have the parent invoke the wait() call to wait for the child process to complete before exiting the program. Perform necessary error checking to ensure that a positive integer is passed on the command line.

<strong>3.22 </strong>In Exercise 3.21, the child process must output the sequence of numbers generated from the algorithm specified by the Collatz conjecture because the parent and child have their own copies of the data. Another approach to designing this program is to establish a shared-memory object between the parent and child processes. This technique allows the child to write the contents of the sequence to the shared-memory object. The parent can then output the sequence when the child completes. Because the memory is shared, any changes the child makes will be reflected in the parent process as well.

This program will be structured using POSIX shared memory as described in Section 3.5.1. The parent process will progress through the following steps:

<ol>

 <li>Establish the shared-memory object (shm open(), ftruncate(), and mmap()).</li>

 <li>Create the child process and wait for it to terminate.</li>

 <li>Output the contents of shared memory.</li>

 <li>Remove the shared-memory object.</li>

</ol>

One area of concern with cooperating processes involves synchronization issues. In this exercise, the parent and child processes must be coordinated so that the parent does not output the sequence until the child finishes execution. These two processes will be synchronized using the wait() system call: the parent process will invoke wait(), which will suspend it until the child process exits.

<strong>3.23 </strong>Section 3.6.1 describes port numbers below 1024 as being well known— that is, they provide standard services. Port 17 is known as the <em>quote-of-</em>




copied and the name of the copied file. The program will then create an ordinary pipe and write the contents of the file to be copied to the pipe. The child process will read this file from the pipe and write it to the destination file. For example, if we invoke the program as follows: filecopy input.txt copy.txt

the file input.txt will be written to the pipe. The child process will read the contents of this file and write it to the destination file copy.txt. You may write this program using either UNIX or Windows pipes.

Programming Projects

<h1>Project 1—UNIX Shell and History Feature</h1>

This project consists of designing a C program to serve as a shell interface that accepts user commands and then executes each command in a separate process. This project can be completed on any Linux, UNIX, or Mac OSX system.

A shell interface gives the user a prompt, after which the next command is entered. The example below illustrates the prompt osh&gt; and the user’s next command: cat prog.c. (This command displays the file prog.c on the terminal using the UNIX cat command.) osh&gt; cat prog.c

One technique for implementing a shell interface is to have the parent process first read what the user enters on the command line (in this case, cat prog.c), and then create a separate child process that performs the command. Unless otherwise specified, the parent process waits for the child to exit before continuing. This is similar in functionality to the new process creation illustrated in Figure 3.10. However, UNIX shells typically also allow the child process to run in the background, or concurrently. To accomplish this, we add an ampersand (&amp;) at the end of the command. Thus, if we rewrite the above command as osh&gt; cat prog.c &amp; the parent and child processes will run concurrently.

The separate child process is created using the fork() system call, and the user’s command is executed using one of the system calls in the exec() family (as described in Section 3.3.1).

A C program that provides the general operations of a command-line shell is supplied in Figure 3.36. The main() function presents the prompt osh-&gt; and outlines the steps to be taken after input from the user has been read. The main() function continually loops as long as should run equals 1; when the user enters exit at the prompt, your program will set should run to 0 and terminate.

This project is organized into two parts: (1) creating the child process and executing the command in the child, and (2) modifying the shell to allow a history feature.

<strong>158          </strong><strong>Chapter 3     </strong><strong>Processes</strong>

#include &lt;stdio.h&gt;

#include &lt;unistd.h&gt;

#define MAX LINE 80 /* The maximum length command */ int main(void)

{

char *args[MAXLINE/2 + 1]; /* command line arguments */ int shouldrun = 1; /* flag to determine when to exit program */

while (shouldrun) { printf(“osh&gt;”); fflush(stdout);

/**

<ul>

 <li>After reading user input, the steps are:</li>

 <li>(1) fork a child process using fork()</li>

 <li>(2) the child process will invoke execvp()</li>

 <li>(3) if command included &amp;, parent will invoke wait() */ } return 0; }</li>

</ul>

<strong>Figure 3.36   </strong>Outline of simple shell.

<strong>Part I— Creating a Child Process</strong>

The first task is to modify the main() function in Figure 3.36 so that a child process is forked and executes the command specified by the user. This will require parsing what the user has entered into separate tokens and storing the tokens in an array of character strings (args in Figure 3.36). For example, if the user enters the command ps -ael at the osh&gt; prompt, the values stored in the args array are:

args[0] = “ps” args[1] = “-ael” args[2] = NULL

This args array will be passed to the execvp() function, which has the following prototype:

execvp(char *command, char *params[]);

Here, command represents the command to be performed and params stores the parameters to this command. For this project, the execvp() function should be invoked as execvp(args[0], args). Be sure to check whether the user included an &amp; to determine whether or not the parent process is to wait for the child to exit.

<strong>Programming Projects              </strong><strong>159 Part II—Creating a History Feature</strong>

The next task is to modify the shell interface program so that it provides a <strong><em>history </em></strong>feature that allows the user to access the most recently entered commands. The user will be able to access up to 10 commands by using the feature. The commands will be consecutively numbered starting at 1, and the numbering will continue past 10. For example, if the user has entered 35 commands, the 10 most recent commands will be numbered 26 to 35.

The user will be able to list the command history by entering the command history

at the osh&gt; prompt. As an example, assume that the history consists of the commands (from most to least recent):

ps, ls -l, top, cal, who, date The command history will output:

6 ps

5 ls -l

4 top

3 cal

2 who

1 date

Your program should support two techniques for retrieving commands from the command history:

<ol>

 <li>When the user enters !!, the most recent command in the history is executed.</li>

 <li>When the user enters a single ! followed by an integer <em>N</em>, the <em>N<sup>th </sup></em>command in the history is executed.</li>

</ol>

Continuing our example from above, if the user enters !!, the ps command will be performed; if the user enters !3, the command cal will be executed. Any command executed in this fashion should be echoed on the user’s screen. The command should also be placed in the history buffer as the next command.

The program should also manage basic error handling. If there are no commands in the history, entering !! should result in a message “No commands in history.”Ifthereisnocommandcorrespondingtothenumber entered with the single !, the program should output “No such command in history.”

<h1>Project 2—Linux Kernel Module for Listing Tasks</h1>

In this project, you will write a kernel module that lists all current tasks in a Linux system. Be sure to review the programming project in Chapter 2, which deals with creating Linux kernel modules, before you begin this project. The project can be completed using the Linux virtual machine provided with this text.