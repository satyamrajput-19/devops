# Day 2 — Linux Permissions, Users, Groups & sudo

## Objectives

- Understand Linux users and groups
- Read permissions such as rwxr-xr--
- Understand owner / group / others
- Use chmod and chown
- Understand sudo and least privilege
- Understand numeric permissions such as 755, 644, and 700
- Practice Linux permission troubleshooting

## Core Concepts

### Users and Groups

Linux is a multi-user operating system.

Useful commands:

```bash
whoami
id
```

Important concepts:
- UID — User ID
- GID — Group ID
- Groups — collections of users

Groups make permission management easier by allowing access to be managed for a set of users.

### File Permissions

Example:

```text
-rw-r--r--
```

The three permission sets represent:
- owner
- group
- others

Permissions:
- r — read
- w — write
- x — execute

For directories:
- r — list contents
- w — create/delete entries
- x — enter/traverse

Object types:
- `-` — file
- `d` — directory
- `l` — symbolic link

### Numeric Permissions

Permission values:

| Permission | Number |
|---|---:|
| r | 4 |
| w | 2 |
| x | 1 |

Examples:
- rwx = 7
- rw- = 6
- r-x = 5
- r-- = 4

Common permissions:

```text
755 = rwxr-xr-x
644 = rw-r--r--
700 = rwx------
```

### chmod

`chmod` changes permissions.

```bash
chmod 755 test.sh
chmod 644 file.txt
chmod 700 private.txt
```

Symbolic form:

```bash
chmod u+x test.sh
chmod g+w test.sh
chmod o-r test.sh
```

Where:
- u — owner/user
- g — group
- o — others
- a — all

### chown

`chown` changes ownership.

```bash
sudo chown alice test.txt
sudo chown alice:developers test.txt
```

### sudo

`sudo` allows an authorized user to execute a command with elevated privileges.

Production systems should follow the principle of least privilege: give users and processes only the permissions they need.

## Hands-on Lab

Create the workspace:

```bash
mkdir -p ~/devops-day2/{permissions,users,groups,scripts}
cd ~/devops-day2
```

Create test files:

```bash
cd ~/devops-day2/permissions
touch public.txt private.txt script.sh

echo "This is public information" > public.txt
echo "This is private information" > private.txt
echo '#!/bin/bash' > script.sh
echo 'echo "DevOps Day 2"' >> script.sh
```

Apply permissions:

```bash
chmod 644 public.txt
chmod 600 private.txt
chmod 755 script.sh
```

Run:

```bash
./script.sh
```

Expected output:

```text
DevOps Day 2
```

Practice symbolic permissions:

```bash
chmod 644 public.txt
chmod u+x public.txt
chmod o-r public.txt
ls -l
```

Create `~/devops-day2/scripts/system-info.sh`:

```bash
#!/bin/bash

echo "User: $(whoami)"
echo "Home: $HOME"
echo "Shell: $SHELL"
echo "Current directory: $(pwd)"
echo "Date: $(date)"
```

Make it executable and run it:

```bash
chmod 755 ~/devops-day2/scripts/system-info.sh
~/devops-day2/scripts/system-info.sh
```

## Interview Questions

1. What are Linux file permissions?
2. What does `chmod 755 file.sh` mean?
3. What does `chmod 644 file.txt` mean?
4. Difference between `chmod` and `chown`?
5. What is `sudo`?
6. Why should applications not run as root unless necessary?
7. What is a UID?
8. What is a GID?
9. Difference between a user and a group?
10. How would you design permissions for an application under `/opt/myapp` that needs to read configuration, write logs, and execute scripts?

## DSA — Second Largest Element

Given:

```text
[10, 5, 8, 20, 15]
```

Expected answer:

```text
15
```

Tasks:
1. Write the algorithm.
2. Explain why it works.
3. Give time complexity.
4. Give space complexity.
5. Implement it in Java.

## DevOps Principle

Think about permissions operationally:
- Who runs the process?
- Who owns the files?
- Which group needs access?
- What needs read access?
- What needs write access?
- What needs execute access?
- Can privileges be reduced?

This foundation will be used later for Linux security, SSH, servers, Docker, CI/CD runners, AWS EC2, Kubernetes, and production troubleshooting.

## Day 2 Checkpoint

Complete:
- Permission calculation
- chmod 640 explanation
- chmod vs chown
- sudo and least privilege
- UID/GID/user/group
- Permission lab
- Second-largest-element Java solution
- Learning report
