Bash / Bourne Shell
====

## Built in Shell Variables
---

```
$# - the number of arguments
$* - all arguments to the shell
$@ - similar to $*; see Section 5.7
$- - options supplied to the shell
#? - return value of the last command executed
$$ - process-id (pid) of the shell
$! - process-id (pid) of the last command started with & (child process)
$HOME - default argument for cd command
$IFS - list of characters that separate words in arguments
$MAIL - file that, when changed, triggers "you have mail" message
$PATH - list of directories to search for commands in
$PS1 - prompt string, default '$ '
$PS2 - prompt string for continued command line, default '> '
$0 - The name of the Bash script.
$1 - $9 - The first 9 arguments to the Bash script. (As mentioned above.)
$# - How many arguments were passed to the Bash script.
$@ - All the arguments supplied to the Bash script.
$? - The exit status of the most recently run process.
$$ - The process ID of the current script.
$USER - The username of the user running the script.
$HOSTNAME - The hostname of the machine the script is running on.
$SECONDS - The number of seconds since the script was started.
$RANDOM - Returns a different random number each time is it referred to.
$LINENO - Returns the current line number in the Bash script.
```

### Examples

`**$1, $2, ...**`

The first, second, etc command line arguments to the script.

`**variable=value**`

To set a value for a variable. Remember, no spaces on either side of =

`**Quotes " '**`

Double will do variable substitution, single will not.

`**variable=$( command )**`

Save the output of a command into a variable

`**export var1**`

Make the variable var1 available to child processes.

## Debugging
---
Shell script with Debug command inside: Add set –xv inside the shell script now to debug the output as shown below.+

```bash
$ ./filesize.sh
```
Total file size in current directory: 652

```bash
$ cat filesize.sh
#!/bin/bash
set -xv
for filesize in $(ls -l . | grep "^-" | awk '{print $5}')
do
  let totalsize=$totalsize+$filesize
done
echo "Total file size in current directory: $totalsize"
```

Output of Shell script with Debug command inside:

```bash
$ ./fs.sh
++ ls -l .
++ grep '^-'
++ awk '{print $5}'
+ for filesize in '$(ls -l . | grep "^-" | awk '\''{print
$5}'\'')'
+ let totalsize=+178
+ for filesize in '$(ls -l . | grep "^-" | awk '\''{print
$5}'\'')'
+ let totalsize=178+285
+ for filesize in '$(ls -l . | grep "^-" | awk '\''{print
$5}'\'')'
+ let totalsize=463+189
+ echo 'Total file size in current directory: 652'
Total file size in current directory: 652
```

Execute Shell script with debug option: Instead of giving the set –xv inside the shell script, you can also provide that while executing the shell script as shown below.

`$ bash -xv filesize.sh`

## Exit Codes
---
I actually [cover exit codes under Concepts](/linux/concepts/exit_codes.md) as they are part of a standard.

## Useful Scripts
---
List of commands you use most often:

```bash
history | awk '{a[$2]++}END{for(i in a){print a[i] " " i}}' | sort -rn | head
How to archive all the files that are not modified in the last x number of days?
```

The following command finds all the files not modified in the last 60 days under `/home/jsmith` directory and creates an archive files under /tmp in the format of ddmmyyyy_archive.tar.

```bash
find /home/jsmith -type f -mtime +60 | xargs tar -cvf
/tmp/`date '+%d%m%Y'_archive.tar`
```

### Suppress standard output using > /dev/null

This will be very helpful when you are debugging shell scripts, where you don’t want to display the echo statement and interested in only looking at the error messages.
```bash
cat file.txt > /dev/null
./shell-script.sh > /dev/null
```

### Suppress standard error using 2> /dev/null

This is also helpful when you are interested in viewing only the standard output and don’t want to view the error messages.
```bash
cat invalid-file-name.txt 2> /dev/null
./shell-script.sh 2> /dev/null
```

Note: One of the most effective ways to use this is in the crontab, where you can suppress the output and error message of a cron task as shown below.
```bash
30 1 * * * command > /dev/null 2>&1
```

### Change the Case

Convert a file to all upper-case

```
$ tr a-z A-Z < employee.txt
100 JASON SMITH
200 JOHN DOE
300 SANJAY GUPTA
400 ASHOK SHARMA
```

1. When you are trying to delete too many files using rm, you may get error message: /bin/rm Argument list too long – Linux. Use xargs to avoid this problem.
`find ~ -name ‘*.log’ -print0 | xargs -0 rm -f`
2. Get a list of all the *.conf file under /etc/. There are different ways to get the same result. Following example is only to demonstrate the use of xargs. The output of the find command in this example is passed to the ls –l one by one using xargs.
`find /etc -name "*.conf" | xargs ls –l`
3. If you have a file with list of URLs that you would like to download, you can use xargs as shown below.
`cat url-list.txt | xargs wget –c`
4. Find out all the jpg images and archive it.
`find / -name *.jpg -type f -print | xargs tar -cvzf images.tar.gz`
5. Copy all the images to an external hard-drive.
`ls *.jpg | xargs -n1 -i cp {} /external-hard-drive/directory`

bash script that print cpu usage,diskusage,ram usage
```bash
#!/bin/bash     
echo CPU: `top -b -n1 | grep "Cpu(s)" | awk '{print $2 + $4}'`
FREE_DATA=`free -m | grep Mem`
CURRENT=`echo $FREE_DATA | cut -f3 -d' '`
TOTAL=`echo $FREE_DATA | cut -f2 -d' '`
echo RAM: $(echo "scale = 2; $CURRENT/$TOTAL*100" | bc)
echo HDD: `df -lh | awk '{if ($6 == "/") { print $5 }}' | head -1 | cut -d'%' -f1`
```
