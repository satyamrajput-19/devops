# Bash / Linux Practice Scripts

These are practice Bash scripts from Linux shell scripting work.

## 1. satyamm.sh — Search a Word in a File

~~~bash
#!/bin/bash

if [ $# -ne 2 ]; then
        echo "Usage: $0 <word> <filename>"
        exit 1
fi

word=$1
file=$2

if [ ! -f "$file" ]; then
        echo "File does not exist"
        exit 2
fi

grep -n "$word" "$file"

if [ $? -eq 0 ]; then
        echo "Word found"
else
        echo "Word not found"
fi
~~~

## 2. multipledirectories.sh — Create Multiple Directories

~~~bash
#!/bin/bash

if [ $# -eq 0 ]; then
        echo "Usage: $0 <directory1> <directory2> ..."
        exit 1
fi

for dir in "$@"
do
        if [ -d "$dir" ]; then
                echo "$dir already exists"
        else
                mkdir "$dir"
                echo "$dir created"
        fi
done
~~~

## 3. managementprogram.sh — User Management Practice

~~~bash
#!/bin/bash

if [ $# -eq 0 ]; then
        echo "Usage: $0 <username1><username2> ..."
        exit 1
fi

for username in "$0"
do
        if id "$username" &>/dev/null; then
                echo "user $username already exists"
        else
                useradd "username"
                echo "user $username created"
        fi
done
~~~

## 4. backupwithadataargument.sh — Directory Backup with Arguments

~~~bash
#!/bin/bash

if [ $# -ne 3 ]; then
        echo "Usage: $0 <source> <destination> <date>"
        exit 1
fi

source=$1
destination=$2
date=$3

if [ ! -d "$source" ]; then
        echo "$source directory does not exist"
        exit 2
fi

mkdir -p "$destination"
backup_name="backup_$date.tar.gz"

tar -czf "$destination/$backup_name" "$source"

if [ $? -eq 0 ]; then
        echo "Backup successful"
        echo "Backup file: $destination/$backup_name"
else
        echo "Backup failed"
        exit 3
fi
~~~

## Practice Areas

- Bash script arguments: `$#`, `$0`, `$1`, `$2`, `$@`
- Conditional statements with `if`
- File and directory tests
- `for` loops
- Linux user management with `id` and `useradd`
- Searching files with `grep`
- Directory creation with `mkdir`
- Creating compressed backups with `tar`
- Exit codes and basic error handling