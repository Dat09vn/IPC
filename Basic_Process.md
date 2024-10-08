**I. What is process?**
**1.** What is a process? A process is a program in execution.

What is a program? A program is a file containing the information of a process and how to build it during run time. When you start execution of the program, it is loaded into RAM and starts executing.

Each process is identified with a unique positive integer called as process ID or simply PID (Process Identification number). The kernel usually limits the process ID to 32767

**II. What exactly is process image?**

**2.** What exactly is process image? Process image is an executable file required while executing the program :
- Code segment or text segment
- Data segment
- Stack segment
- Heap segment

![image](https://github.com/user-attachments/assets/931a48e1-5513-4d0b-8bc3-cee18c90c448)


**3. **Process creation is achieved through the fork() system call. The fork() system call returns either of the three values :
- Negative value to indicate an error, i.e., unsuccessful in creating the child process.
- Returns a zero for child process.
- Returns a positive value for the parent process. This value is the process ID of the newly created child process.

**4.** What happens if the parent process finishes its task early than the child process and then quits or exits? Now who would be the parent of the child process? The parent of the child process is init process, which is the very first process initiating all the tasks.

Observe that the parent process PID was 94 and the child process PID was 95. After the parent process exits, the PPID of the child process changed from 94 to 1 (init process)

The wait() system call would wait for one of the children to terminate and return its termination status in the buffer as explained below.

**5.** Process group, Job control

Process group is a collection of one or more processes. A process group constitutes of one or more processes sharing the same process group identifier (PGID)

Job control: This permits a shell user to simultaneously execute multiple commands (or jobs), one in the foreground and all remaining in the background. It is also possible to move the jobs from the foreground to the background and vice-versa.
It is also possible to terminate the process either with CTRL+C or kill command. 

Example: To stop the current running process, you need to enter CTRL+Z. This gives you a job number. The job can be resumed either in the foreground or the background: command

- show all jobs list: jobs
- run foreground of last stoped process: fg
- run background of last stoped process: bg
- specific process: fg % 1, fg % 2

**6.** Other process

- Orphan Process: In real systems, when the parent process exits before the child process, the child process is "adopted" by the init process (with PID 1 in Linux), which becomes its new parent. This situation is known as an orphaned process.
- Zombie Process: A zombie process is a process that has completed execution (via exit()), but still has an entry in the process table. This happens when the parent process hasn't yet read the exit status of the child process using a system call like wait() or waitpid(). In this state, the process is "dead" but still occupying an entry in the system's process table, hence the term "zombie."
- Daemon Process: A daemon process is a background process that runs independently of the controlling terminal and often provides system or application-level services. These processes are typically started during system boot or run continuously in the background without user interaction, performing essential system tasks such as handling requests for services, monitoring system events, or performing scheduled tasks. A daemon is used when you need a process to run in the background and perform tasks without user interaction. Daemons are typically employed to provide system or application services, monitor system events, or perform periodic tasks. https://notes.shichao.io/apue/ch13/

**7.** Overlaying Process Image

We are running a program and we want to run another program from the current program. If so, the second program will overlay the current program and current program will not continue execute. 

--> Solution: Create a child process, so that we have a parent process and a newly created child process. Already we are running the current program in the parent process, so run the newly created process in the child. (current program is still running in parent process, second program will run in child process.

**8.** A system call is a mechanism used by an application to request services from the operating system (OS). It acts as a bridge between user-mode applications and the kernel, allowing programs to access low-level functions such as hardware manipulation, file system operations, or process control.

Key points about system calls:
- Kernel Interaction: System calls allow user programs to communicate with the OS kernel, which operates in privileged mode.
- Functionality: They provide various services like file operations (open, read, write), process management (fork, exec), memory allocation (mmap), and communication (socket).

Types:
- Process Control: E.g., fork(), exec(), exit()
- File Operations: E.g., open(), read(), write()
- Device Management: E.g., ioctl()
- Information Maintenance: E.g., getpid(), alarm()
- Communication: E.g., pipe(),shmctl(), msgget(), socket()

**9.** System V and POSIX are two important standards related to Unix-based operating systems. They define different ways of interacting with the system, with POSIX being a more general, widely-adopted standard, and System V representing an earlier Unix lineage.

System V is an early Unix version that influenced many commercial Unix systems and is known for its init system, IPC mechanisms, and other Unix features.

POSIX is a standard designed to ensure portability across Unix-like systems by defining a standard API for **system calls, IPC, threads,** and other functionalities.

If you are writing code meant to run on a wide variety of Unix-based systems, you'll generally prefer using POSIX functions, as they offer broader portability than System V-specific features.

