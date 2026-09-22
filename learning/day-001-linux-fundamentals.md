# Day 1 — Linux Fundamentals

## Goal

Start the DevOps journey from the operating-system layer. Today focuses on understanding what Linux is, how the filesystem is organized, and how to navigate it confidently from the terminal.

## Study target

- Total: ~5 hours
- Theory: 60 minutes
- Hands-on Linux lab: 2 hours
- Bash/terminal practice: 60 minutes
- Interview + CS fundamentals: 45 minutes
- Revision/notes: 15 minutes

## 1. What is Linux?

Linux is an operating-system kernel. A complete Linux distribution combines the Linux kernel with user-space tools, libraries, package managers, and other software.

For DevOps, Linux matters because cloud servers, containers, Kubernetes nodes, CI runners, and many production systems depend heavily on Linux.

### Kernel vs user space

- Kernel: manages CPU, memory, processes, devices, networking, and system calls.
- User space: applications and utilities that run outside the kernel.
- Shell: a user-space program that lets us interact with the operating system through commands.

Mental model:

User → Shell → System calls → Linux kernel → Hardware

## 2. Terminal and shell

The terminal is the interface in which we interact with a shell.

Common shells include Bash and Zsh.

Check your shell:
echo $SHELL

Check the current user:
whoami

Check the current directory:
pwd

## 3. Linux filesystem

Linux uses a single filesystem hierarchy beginning at /.

Important directories:

- / — filesystem root
- /home — normal users' home directories
- /etc — system configuration
- /var — variable data such as logs
- /tmp — temporary files
- /usr — many user-space programs and libraries
- /opt — optional software
- /dev — device files
- /proc — virtual filesystem exposing process/kernel information

Do not memorize these blindly. Learn what kind of information belongs in each location.

## 4. Navigation commands

Practice:

pwd
ls
ls -la
cd /
cd ~
cd ..
cd -

Create a practice directory:

mkdir -p ~/devops-day1/linux-lab
cd ~/devops-day1/linux-lab

Create files:

touch notes.txt
touch commands.txt

Write text:

echo "Linux Day 1" > notes.txt
echo "DevOps journey started" >> notes.txt

Read it:

cat notes.txt

## 5. Files and directories

Practice:

mkdir test
cp notes.txt test/
mv commands.txt test/
ls -la test/
rm test/commands.txt
rmdir test

Understand the difference between:

- file
- directory
- absolute path
- relative path
- hidden file

## 6. Useful inspection commands

Practice:

file notes.txt
wc -l notes.txt
wc -c notes.txt
head notes.txt
tail notes.txt

## 7. Permissions — first introduction

Linux permissions are represented for owner, group, and others.

Inspect:

ls -l

You may see something like:

-rw-r--r--

Conceptually:

- rw- = owner permissions
- r-- = group permissions
- r-- = others permissions

We will study permissions deeply on a later day. Today, understand the model.

## 8. PATH

The PATH environment variable tells the shell where to look for executable commands.

Check it:

echo $PATH

Find a command:

which bash
which python3

This concept becomes important later for Bash, Python, Docker, CI/CD, and Kubernetes tooling.

# Day 1 Lab

Complete these without copying a solution from elsewhere.

### Task 1

Create:

~/devops-day1/
├── linux/
│   ├── notes.txt
│   ├── commands.txt
│   └── logs/
└── scripts/

### Task 2

Put at least 10 Linux commands you learned into commands.txt.

### Task 3

Put a short explanation of Linux, kernel, shell, and filesystem into notes.txt.

### Task 4

Use ls -la and explain every column of the output.

### Task 5

Find your shell, current user, home directory, PATH, and current working directory.

### Task 6

Use find to locate all .txt files under ~/devops-day1.

### Task 7

Break something safely.

Create a file, remove it, recreate it, and explain what each command did.

## Interview questions

Answer these in your own words:

1. What is Linux?
2. What is the difference between the Linux kernel and an operating system distribution?
3. What is a shell?
4. What is the difference between an absolute and relative path?
5. What is /?
6. What is /etc used for?
7. What is /var generally used for?
8. What is /proc?
9. What does PATH do?
10. What is the difference between a terminal and a shell?

## DSA — Day 1

Today is not about solving a hard problem.

Learn:

- Array
- Index
- Traversal
- Time complexity

Practice one simple problem:

Find the maximum element in an array.

Before writing code, explain:

1. Your algorithm.
2. Why it works.
3. Time complexity.
4. Space complexity.

## OS — Day 1

Learn these terms today:

- kernel
- process
- thread
- system call
- user space
- kernel space

Do not go deep yet. We will build these concepts properly as the roadmap progresses.

## Deliverable

By the end of Day 1 you should have:

- Completed the Linux lab
- Created your ~/devops-day1 workspace
- Written your notes
- Answered the 10 interview questions
- Solved the DSA problem
- Understood the kernel/shell/user-space relationship

## LinkedIn learning post

Day 1 of my DevOps & Cloud Engineering journey.

Today I started from the foundation: Linux.

I worked through Linux filesystem structure, terminal navigation, paths, basic file operations, permissions, the shell, and the relationship between user space and the Linux kernel.

The key lesson for me was that DevOps is not just about learning cloud tools. Understanding the operating system underneath those tools is essential.

Hands-on practice today included navigating the filesystem, creating and manipulating files/directories, inspecting permissions, and working with environment variables such as PATH.

Next: deeper Linux administration and Bash fundamentals.

#DevOps #Linux #CloudComputing #DevOpsLearning #100DaysOfCode
