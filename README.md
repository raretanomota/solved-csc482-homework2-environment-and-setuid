Download Link: https://assignmentchef.com/product/solved-csc482-homework2-environment-and-setuid
<br>
The learning objective of this lab is for students to understand how environment variables affect program and system behaviors. Environment variables are a set of dynamic named values that can affect the way running processes will behave on a computer. They are used by most operating systems, since they were introduced to Unix in 1979. Although environment variables affect program behaviors, how they achieve that is not well understood by many programmers. As a result, if a program uses environment variables, but the programmer does not know that they are used, the program may have vulnerabilities. In this lab, students will understand how environment variables work, how they are propagated from parent process to child, and how they affect system/program behaviors. We are particularly interested in how environment variables affect the behavior of Set-UID programs, which are usually privileged programs.

This lab covers the following topics. Detailed coverage of these topics can be found in Chapters 1 and 2 of the SEED book, <em>Computer Security: A Hands-on Approach</em>, by Wenliang Du.

<ul>

 <li>Environment variables</li>

 <li>Set-UID programs</li>

 <li>Securely invoke external programs</li>

 <li>Capability leaking</li>

 <li>Dynamic loader/linker</li>

</ul>

Readings and related topics. Detailed coverage of the Set-UID mechanism and its security problems can be found in Chapter 1 of the SEED book, <em>Computer Security: A Hands-on Approach</em>, by Wenliang Du. Environment variables and their related attacks are covered in Chapter 2 of the book. A topic related to environment variables is the Shellshock attack, which was discovered in 2014. This relatively new attack is covered in Chapter 3, as well as in a separate SEED lab.

Lab environment. This lab has been tested on our pre-built Ubuntu 12.04 VM and Ubuntu 16.04 VM, both of which can be downloaded from the SEED website.

<h1>1          Lab Tasks</h1>

<h2>1.1        Task 1: Manipulating Environment Variables</h2>

In this task, we study the commands that can be used to set and unset environment variables. We are using Bash in the seed account. The default shell that a user uses is set in the /etc/passwd file (the last field of each entry). You can change this to another shell program using the command chsh (please do not do it for this lab). Please do the following tasks:

<ul>

 <li>Use printenv or env command to print out the environment variables. If you are interested in some particular environment variables, such as PWD, you can use “printenv PWD” or “env | grep PWD”.</li>

 <li>Use export and unset to set or unset environment variables. It should be noted that these two commands are not seperate programs; they are two of the Bash’s internal commands (you will not be able to find them outside of Bash).</li>

</ul>

<h2>1.2        Task 2: Passing Environment Variables from Parent Process to Child Process</h2>

In this task, we study how a child process gets its environment variables from its parent. In Unix, fork() creates a new process by duplicating the calling process. The new process, referred to as the child, is an exact duplicate of the calling process, referred to as the parent; however, several things are not inherited by the child (please see the manual of fork() by typing the following command: man fork). In this task, we would like to know whether the parent’s environment variables are inherited by the child process or not.

Step 1. Please compile and run the following program, and describe your observation. Because the output contains many strings, you should save the output into a file, such as using a.out &gt; child (assuming that a.out is your executable file name).

<table width="624">

 <tbody>

  <tr>

   <td width="624">#include &lt;unistd.h&gt;#include &lt;stdio.h&gt; #include &lt;stdlib.h&gt;extern char **environ;void printenv(){int i = 0;while (environ[i] != NULL) { printf(“%s
”, environ[i]); i++;} }void main(){ pid_t childPid;switch(childPid = fork()) { case 0: /* child process */ printenv();              Àexit(0);default: /* parent process *///printenv();                           Áexit(0);}}</td>

  </tr>

 </tbody>

</table>

Step 2. Now comment out the printenv() statement in the child process case (Line À), and uncomment the printenv() statement in the parent process case (Line Á). Compile and run the code again, and describe your observation. Save the output in another file.

Step 3.      Compare the difference of these two files using the diff command. Please draw your conclusion.

<h2>1.3        Task 3: Environment Variables and execve()</h2>

In this task, we study how environment variables are affected when a new program is executed via execve(). The function execve() calls a system call to load a new command and execute it; this function never returns. No new process is created; instead, the calling process’s text, data, bss, and stack are overwritten by that of the program loaded. Essentially, execve() runs the new program inside the calling process. We are interested in what happens to the environment variables; are they automatically inherited by the new program?

Step 1. Please compile and run the following program, and describe your observation. This program simply executes a program called /usr/bin/env, which prints out the environment variables of the current process.

<table width="624">

 <tbody>

  <tr>

   <td width="313">#include &lt;stdio.h&gt; #include &lt;stdlib.h&gt;extern char **environ;int main(){ char *argv[2];argv[0] = “/usr/bin/env”; argv[1] = NULL;execve(“/usr/bin/env”, argv, NULL);return 0 ;}</td>

   <td width="311">À</td>

  </tr>

 </tbody>

</table>

Step 2. Change the invocation of execve() in Line À to the following; describe your observation. execve(“/usr/bin/env”, argv, environ);

Step 3.   Please draw your conclusion regarding how the new program gets its environment variables.

<h2>1.4        Task 4: Environment Variables and system()</h2>

In this task, we study how environment variables are affected when a new program is executed via the system() function. This function is used to execute a command, but unlike execve(), which directly executes a command, system() actually executes “/bin/sh -c command”, i.e., it executes /bin/sh, and asks the shell to execute the command.

If you look at the implementation of the system() function, you will see that it uses execl() to execute /bin/sh; execl() calls execve(), passing to it the environment variables array. Therefore, using system(), the environment variables of the calling process is passed to the new program /bin/sh. Please compile and run the following program to verify this.

<table width="624">

 <tbody>

  <tr>

   <td width="624">#include &lt;stdio.h&gt; #include &lt;stdlib.h&gt;int main(){ system(“/usr/bin/env”);return 0 ;}</td>

  </tr>

 </tbody>

</table>

<h2>1.5        Task 5: Environment Variable and Set-UID Programs</h2>

Set-UID is an important security mechanism in Unix operating systems. When a Set-UID program runs, it assumes the owner’s privileges. For example, if the program’s owner is root, then when anyone runs this program, the program gains the root’s privileges during its execution. Set-UID allows us to do many interesting things, but it escalates the user’s privilege when executed, making it quite risky. Although the behaviors of Set-UID programs are decided by their program logic, not by users, users can indeed affect the behaviors via environment variables. To understand how Set-UID programs are affected, let us first figure out whether environment variables are inherited by the Set-UID program’s process from the user’s process.

Step 1. Write the following program that can print out all the environment variables in the current process.

<table width="624">

 <tbody>

  <tr>

   <td width="624">#include &lt;stdio.h&gt; #include &lt;stdlib.h&gt;extern char **environ;void main(){int i = 0;while (environ[i] != NULL) { printf(“%s
”, environ[i]); i++;}}</td>

  </tr>

 </tbody>

</table>

Step 2.       Compile the above program, change its ownership to root, and make it a Set-UID program.

// Asssume the program’s name is foo

$ sudo chown root foo

$ sudo chmod 4755 foo

Step 3. In your shell (you need to be in a normal user account, not the root account), use the export command to set the following environment variables (they may have already exist):

<ul>

 <li>PATH</li>

 <li>LDLIBRARYPATH</li>

 <li>ANYNAME (this is an environment variable defined by you, so pick whatever name you want).</li>

</ul>

These environment variables are set in the user’s shell process. Now, run the Set-UID program from Step 2 in your shell. After you type the name of the program in your shell, the shell forks a child process, and uses the child process to run the program. Please check whether all the environment variables you set in the shell process (parent) get into the Set-UID child process. Describe your observation. If there are surprises to you, describe them.

<h2>1.6        Task 6: The PATH Environment Variable and Set-UID Programs</h2>

Because of the shell program invoked, calling system() within a Set-UID program is quite dangerous. This is because the actual behavior of the shell program can be affected by environment variables, such as PATH; these environment variables are provided by the user, who may be malicious. By changing these variables, malicious users can control the behavior of the Set-UID program. In Bash, you can change the PATH environment variable in the following way (this example adds the directory /home/seed to the beginning of the PATH environment variable):

$ export PATH=/home/seed:$PATH

The Set-UID program below is supposed to execute the /bin/ls command; however, the programmer only uses the relative path for the ls command, rather than the absolute path:

int main() { system(“ls”); return 0;

}

Please compile the above program, and change its owner to root, and make it a Set-UID program. Can you let this Set-UID program run your code instead of /bin/ls? If you can, is your code running with the root privilege? Describe and explain your observations.

Note (Ubuntu 16.04 VM only): The system(cmd) function executes the /bin/sh program first, and then asks this shell program to run the cmd command. In both Ubuntu 12.04 and Ubuntu 16.04 VMs, /bin/sh is actually a symbolic link pointing to the /bin/dash shell. However, the dash program in these two VMs have an important difference. The dash shell in Ubuntu 16.04 has a countermeasure that prevents itself from being executed in a Set-UID process. Basically, if dash detects that it is executed in a Set-UID process, it immediately changes the effective user ID to the process’s real user ID, essentially dropping the privilege. The dash program in Ubuntu 12.04 does not have this behavior.

Since our victim program is a Set-UID program, the countermeasure in /bin/dash can prevent our attack. To see how our attack works without such a countermeasure, we will link /bin/sh to another shell that does not have such a countermeasure. We have installed a shell program called zsh in our Ubuntu 16.04 VM. We use the following commands to link /bin/sh to zsh (there is no need to do these in Ubuntu

12.04):

$ sudo rm /bin/sh

$ sudo ln -s /bin/zsh /bin/sh

<h2>1.7        Task 7: The LDPRELOAD Environment Variable and Set-UID Programs</h2>

In this task, we study how Set-UID programs deal with some of the environment variables. Several environment variables, including LDPRELOAD, LDLIBRARYPATH, and other LD* influence the behavior of dynamic loader/linker. A dynamic loader/linker is the part of an operating system (OS) that loads (from persistent storage to RAM) and links the shared libraries needed by an executable at run time.

In Linux, ld.so or ld-linux.so, are the dynamic loader/linker (each for different types of binary). Among the environment variables that affect their behaviors, LDLIBRARYPATH and LDPRELOAD are the two that we are concerned in this lab. In Linux, LDLIBRARYPATH is a colon-separated set of directories where libraries should be searched for first, before the standard set of directories. LDPRELOAD specifies a list of additional, user-specified, shared libraries to be loaded before all others. In this task, we will only study LDPRELOAD.

Step 1. First, we will see how these environment variables influence the behavior of dynamic loader/linker when running a normal program. Please follow these steps:

<ol>

 <li>Let us build a dynamic link library. Create the following program, and name it mylib.c. It basically overrides the sleep() function in libc:</li>

</ol>

<table width="588">

 <tbody>

  <tr>

   <td width="588">#include &lt;stdio.h&gt; void sleep (int s){/* If this is invoked by a privileged program, you can do damages here! */ printf(“I am not sleeping!
”);}</td>

  </tr>

 </tbody>

</table>

<ol start="2">

 <li>We can compile the above program using the following commands (in the -lc argument, the second character is <em>`</em>):</li>

</ol>

% gcc -fPIC -g -c mylib.c

% gcc -shared -o libmylib.so.1.0.1 mylib.o -lc

<ol start="3">

 <li>Now, set the LDPRELOAD environment variable:</li>

</ol>

% export LD_PRELOAD=./libmylib.so.1.0.1

<ol start="4">

 <li>Finally, compile the following program myprog, and in the same directory as the above dynamic link library libmylib.so.1.0.1:</li>

</ol>

<table width="588">

 <tbody>

  <tr>

   <td width="588">/* myprog.c */ int main(){ sleep(1); return 0;}</td>

  </tr>

 </tbody>

</table>

Step 2. After you have done the above, please run myprog under the following conditions, and observe what happens.

<ul>

 <li>Make myprog a regular program, and run it as a normal user.</li>

 <li>Make myprog a Set-UID root program, and run it as a normal user.</li>

 <li>Make myprog a Set-UID root program, export the LDPRELOAD environment variable again in the root account and run it.</li>

 <li>Make myprog a Set-UID user1 program (i.e., the owner is user1, which is another user account), export the LDPRELOAD environment variable again in a different user’s account (not-root user) and run it.</li>

</ul>

Step 3. You should be able to observe different behaviors in the scenarios described above, even though you are running the same program. You need to figure out what causes the difference. Environment variables play a role here. Please design an experiment to figure out the main causes, and explain why the behaviors in Step 2 are different. (Hint: the child process may not inherit the LD* environment variables).

<h2>1.8        Task 8: Invoking External Programs Using system() versus execve()</h2>

Although system() and execve() can both be used to run new programs, system() is quite dangerous if used in a privileged program, such as Set-UID programs. We have seen how the PATH environment variable affect the behavior of system(), because the variable affects how the shell works. execve() does not have the problem, because it does not invoke shell. Invoking shell has another dangerous consequence, and this time, it has nothing to do with environment variables. Let us look at the following scenario.

Bob works for an auditing agency, and he needs to investigate a company for a suspected fraud. For the investigation purpose, Bob needs to be able to read all the files in the company’s Unix system; on the other hand, to protect the integrity of the system, Bob should not be able to modify any file. To achieve this goal, Vince, the superuser of the system, wrote a special set-root-uid program (see below), and then gave the executable permission to Bob. This program requires Bob to type a file name at the command line, and then it will run /bin/cat to display the specified file. Since the program is running as a root, it can display any file Bob specifies. However, since the program has no write operations, Vince is very sure that Bob cannot use this special program to modify any file.

<table width="624">

 <tbody>

  <tr>

   <td width="624">#include &lt;string.h&gt;#include &lt;stdio.h&gt; #include &lt;stdlib.h&gt;int main(int argc, char *argv[]){ char *v[3]; char *command;if(argc &lt; 2) { printf(“Please type a file name.
”); return 1;}v[0] = “/bin/cat”; v[1] = argv[1]; v[2] = NULL;</td>

  </tr>

  <tr>

   <td width="624">command = malloc(strlen(v[0]) + strlen(v[1]) + 2); sprintf(command, “%s %s”, v[0], v[1]);// Use only one of the followings. system(command);// execve(v[0], v, NULL);return 0 ;}</td>

  </tr>

 </tbody>

</table>

Step 1: Compile the above program, make it a root-owned Set-UID program. The program will use system() to invoke the command. If you were Bob, can you compromise the integrity of the system? For example, can you remove a file that is not writable to you?

Step 2: Comment out the system(command) statement, and uncomment the execve() statement; the program will use execve() to invoke the command. Compile the program, and make it a root-owned Set-UID. Do your attacks in Step 1 still work? Please describe and explain your observations.

<h2>1.9        Task 9: Capability Leaking</h2>

To follow the Principle of Least Privilege, Set-UID programs often permanently relinquish their root privileges if such privileges are not needed anymore. Moreover, sometimes, the program needs to hand over its control to the user; in this case, root privileges must be revoked. The setuid() system call can be used to revoke the privileges. According to the manual, “setuid() sets the effective user ID of the calling process. If the effective UID of the caller is root, the real UID and saved set-user-ID are also set”. Therefore, if a Set-UID program with effective UID 0 calls setuid(n), the process will become a normal process, with all its UIDs being set to n.

When revoking the privilege, one of the common mistakes is capability leaking. The process may have gained some privileged capabilities when it was still privileged; when the privilege is downgraded, if the program does not clean up those capabilities, they may still be accessible by the non-privileged process. In other words, although the effective user ID of the process becomes non-privileged, the process is still privileged because it possesses privileged capabilities.

Compile the following program, change its owner to root, and make it a Set-UID program. Run the program as a normal user, and describe what you have observed. Will the file /etc/zzz be modified? Please explain your observation.

<table width="624">

 <tbody>

  <tr>

   <td width="624">#include &lt;stdio.h&gt;#include &lt;stdlib.h&gt; #include &lt;fcntl.h&gt;void main() { int fd;/* Assume that /etc/zzz is an important system file, * and it is owned by root with permission 0644.*       Before running this program, you should creat*       the file /etc/zzz first. */ fd = open(“/etc/zzz”, O_RDWR | O_APPEND); if (fd == -1) {</td>

  </tr>

  <tr>

   <td width="624">printf(“Cannot open /etc/zzz
”); exit(0);}/* Simulate the tasks conducted by the program */ sleep(1);/* After the task, the root privileges are no longer needed, it’s time to relinquish the root privileges permanently. */setuid(getuid()); /* getuid() returns the real uid */if (fork()) { /* In the parent process */ close (fd); exit(0);} else { /* in the child process *//* Now, assume that the child process is compromised, malicious attackers have injected the following statements into this process */write (fd, “Malicious Data
”, 15); close (fd);}}</td>

  </tr>

 </tbody>

</table>