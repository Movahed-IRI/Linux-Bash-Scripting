## Gathering information about processes
each process is assigned a unique identification number called a process ID (PID)

`ps` is an important tool for gathering information about the processes.
- when `ps` is run without any parameter ,`ps` will display processes that are running on the current terminal

- In order to show more columns consisting of more information ,use `-f`(stands for full) as follows : `ps -f`

- In order to get information about every process runnig on the system ,add the `-e`(every) option .The `-ax`(all) option will also produce an identical output.

- We can specify the columns to be displayed using the `-o` flag and hence ,thereby print only the required columns.

?> Parameters for `-o` are delimited by using the comma(,) operator .It should be noted that there is no space in between the comma operator and the next param

!> Usage of `-e` with filter will nullify the filter and it will show all process entries.

For example 
``` bash 
ps -eo comm,pcpu | head
```

- The different parameters that can be used with the `-o` option and their description are as follows 
    - `pcpu`  -> percentage of CPU
    - `pid` -> process ID
    - `ppid` -> parent process ID
    - `pmem` -> percentage of memory
    - `comm` -> Executable filename
    - `cmd` -> simple command
    - `user` -> the user who started the process
    - `nice` -> the priority (niceness)
    - `time` -> cumulative CPU time
    - `etime` -> Elapsed time since the process started 
    - `tty` -> the associated TTY device
    - `euid` -> The effective user
    - `stat` -> process stat

- Sorting the ps output with respect to a parameter 
output of the `ps` command can be sorted according to specified columns with the `--sort` parameter.

The ascending or descending order can be specified by using `+`(ascending) or `-`(descending) prefix to the parameter as follow
``` bash 
ps [OPTIONS] --sort -parameter1,+parameter2
ps -eo comm,pcpu --sort -pcpu | head
```

### Finding the process ID when given command names 
This information can be found by using the `ps` or `pgrep` command .
- We can use `ps` as follows 
``` bash 
ps -C COMMAND_NAME
# Or
ps -C COMMAND_NAME -o pid=
```
In order to remove headers for each column ,append `=` to the parameters

- We can use `pgrep` as follows : `pgrep COMMAND`

?> pgrep requires only a portion of the command name as its input argument to extract a bash command ,for example ,`pgrep ash` or `pgrep bas` will also work .But `ps` requires you to type the exact command

- In order to specify a delimiter char for output rather than using a newline as the delimiter ,use 
``` bash 
pgrep COMMAND -d Delimiter_string
# For Example
pgrep bash -d ":"
```

- Specify a list of owners of the user for the matching processes as follow
``` bash 
pgrep -u root,sedgeek COMMAND
```

- return the count of matching process as follows : `pgrep -c COMMAND`

### Filters with ps for real user or ID,effective user or ID
- Specify an effective users'list by using `-u EUSER1,EUSER2`
- Specify a real users'list by using `-U RUSER1,RUSER2`

For example 
``` bash 
ps -u root -U root -o user,pcpu
```

### tty filter for ps 
use the `-t` option to specify the TTY list ,as follows :
``` bash 
ps -t pts/0,pts/1
```

### Information about process thread 
We can show information about threads in the `ps` output by adding the `-L` option .Then ,it will show two columns NLWP and NLP
- NLWP is the thread count for a process
- NLP is the thread ID for each entry in ps

For example :
``` bash 
ps -eLf
# Or 
ps -eLf --sort -nlwp | head
```

### Showing environment variable for a process
In order to list environment varibales with `ps` entries use `ps -eo cmd e`

For example :
``` bash 
ps -eo pid,cmd e | tail -n 3
```
Suppose ,if we want to open a GUI-windowed application at a given time .We schedule it using `crontab` at a specified time .A windowed application always depends on the DISPLAY environment variable .To figure out the environment varibales needed ,run windowsapp manually and then run `ps -C windowsapp -eo cmd e` .Find out the env variables and prefix them before a command name appears in crontab as follows
```
00 10 * * * DISPLAY=:0 /usr/bin/windowapp
```

### About which ,whereis ,file ,whatis ,and load average
- `which` : The which cmd is used to find the location of a command

?> The terminal looks for the command in a set of locations and execute the executable file if found at the location .These sets of locations are specified using an env variable __PATH__

?> We can export __PATH__ and can add our own locations to be searched when command names are typed .For example `export PATH=$PATH:/home/sedgeek/bin`

- `whereis` : The whereis is similar to the which command .But it not only returns the path of the command ,it will also print the location of the man page

- `file` : It is used for determining the file type
- `whatis` : The whatis command outputs a one-line description of the command given as the argument

?> Sometimes ,we need to search if some command related to a word exists .Then ,we can search the man pages for strings in the commands ,as follow `apropos COMMAND` or `man -k COMMAND`

- __Load average__ : Load average is specified by three values. 
    - The first value indicates the average in one minutes
    - The second indicates the average in five minutes
    - the third indicates the average in 15 minutes .

Load average can be obtained by running `uptime` 


## Killing processes and send or respond to signals
Signals are an inter-process mechanism available in Linux .When a process receives a signal ,it responds by executing a signal handler .The Ctrl+c and Ctrl+z also send two types of signals.

The `kill` command is used to send signals to processes and the `trap` command is used to handle the received signals

- In order to list all the signals available use `kill -l`
- Terminate a process as follows `kill PROCESS_ID_LIST`

?>The `kill`  command issues a __TERM__ signal by default.

The process ID list is to be specified with space as a delimiter between process IDs

- In order to specify a signal to be sent to a process via the `kill`  command ,use `kill -s SIGNAL PID`

- we frequently use only a few signals .They are as follows
    - `SIGHUP 1` : Hangup detection on death of the controlling process or terminal 
    - `SIGNINT 2` : Signal which is emitted when Ctrl+c is passed
    - `SIGNKILL 9` : Signal used to force kill the process
    - `SIGNTERM 15` : Signal used to terminate a process by default
    - `SIGTSTP 20` : Signal emmited when ctrl+z is passed

- The `killall` command terminate the process by name as follows `killall process_name`

- In order to send a signal to a process by name ,use : `killall -s SIGNAL process_name` or `killall -9 gedit`

- Specify the process by name ,which is specified by users who own it ,by using `killall -u USERNAME process_name`

- If you want `killall` to interactively confirm before killing processes ,use the `-i` argument.

- Thee `pkill` command is similar to the `kill` command but it ,by default ,accepts a process name instead of a process ID.

For example
``` bash 
pkill -s SIGNAL process_name
``` 
!> SIGNAL is the signal number .The signal name is not supported with `pkill`

### Capturing and responding to signals
The syntax is as follows :
``` bash 
trap 'signal_handler_function_name' SIGNAL_LIST
```
__SIGNAL_LIST__ is delimited by space .It can be a signal number or a singal name.

## Sending message to user terminals
In order to broadcast a message to all users and all logged in terminal ,use `cat message | wall`

?> It is noteworthy that the message gets displayed to the current terminal only it the write message option is enabled .In the most distros ,write message is enabled by default .If the sender of the message is root ,then the message get displayed on the screen irrespective of wheter the write message option is enabled or disabled by the user

- In order to enable write messages ,use : `mesg y`
- In order to disable write messages ,use `mesg n`

## Gathering system information 
- In order to print the hostname of the current system ,use :`hostname` or `uname -n`
- print long details about the linux kernel version `uname -a`
- In order to print the kernel release `uname -r`
- Print the machine type as follows `uname -m`
- In order to print details about the CPU ,use `cat /proc/cpuinfo`
- In order to extract the processor name ,use `cat /proc/cpuinfo | sed -n 5p`
- print details about the memory or ram `cat /proc/meminfo`
- Print the total memory available on the system `cat /proc/meminfo | head -1`
- In order to list out the partitions available on the system `cat /proc/partiotions` or `fdisk -l`
- Get the entire details about system ,as follows `lshw`

## using /proc for gathering information
Suppose bash is running with PID 4295 (pgrep bash) , /proc/4295 will exist .Each of the directories corresponding to the process will contain a lot of information regarding to that process .Few of the important files in /proc/PID are as follows
- `environ` : This contains environment variable associated with that process
- `cwd` : This is a symlink to working direcotry of the process
- `exe` : This is a symlink to the running executable for the current process ,For exmple 
``` bash 
readlink /proc/4295/exe
```
- `fd` : This is directory consisting of entries on file descriptors used by the process

## Scheduling with cron 
- cronjob to execute the test.sh script at the second minute of all hours on all days :
``` bash 
02 * * * * /home/mypc/test.sh
```
- In order to run the script at the fifth ,sixth ,and seventh hours on all days ,use 
``` bash 
00 5,6,7 * *
```
- Shutdown the computer at 2 A.M
``` bash 
00 02 * * * /sbin/shutdown -h
```
- Execute script.sh at every hour on sunday as follows :
``` bash
00 */12 * * 0 /home/mypc/test.sh
```
- Use the `-e` option to crontab to start editing the cron table ,as follows : `crontab -e`

- There are two methods we usually use when we invoke the `crontab` command inside a script for scheduling tasks.
    - create a text file with the cronjob in it ,and then run the `crontab` with this filename as the command argument.
    - Or ,specify the cronjob inline without creating a seprate file .For example 
    ``` bash 
    crontab << EOF
    02 * * * * /home/mypc/test.sh
    EOF
    ```

- Each cron table consists of six sections in the following order
    - Minute (0-59)
    - Hour (0-23)
    - Day (1-31)
    - Month (1-12)
    - Weekday (0-6)
    - COMMAND

For example use `*/5` in the minute field for runnig the command at every five minutes.

?> cronjobs are executed with privileges with which the crontab command was executed .

!> The command specified in a cronjob are written with the full path to the command

### Specifying environment variables 
For example ,if you are using a proxy server for connecting to the internet 
``` bash 
crontab << EOF
http-proxy=http://192.168.0.3:3128
00 * * * * /home/mypc/download.sh
EOF
```
### Running command at system start up/boot
To run a command at boot ,add the following line to your crontab `@reboot` command

- We can list these existing cronjobs using the -l option `crontab -l`
The `crontab -l` lists the existing entries in the cron table for the current user.

- We can also view the cron table for other users by specifying a username with the `-u` option as follows :
``` bash 
crontab -l -u sedgeek
```

- We can remove the cron table for the current user using the `-r` option. `crontab -r`

- In order to remove crontab for another user ,use : `crontab -u user -r`
?> Run as root to get higher privilege

## Writing and reading the MySQL database from Bash
``` bash 
mysql -u $user -p$pass << EOF 2>/dev/null
CREATE DATABASE students;
EOF
```
?> set `@i=0` is an SQL construct used to set the variable i=0

## User administration 
- `useradd` : the useradd command can be used to create a new user.It has the following syntax : `useradd USER -p PASSWORD`
- The `-m` option used to create the home directory .It is possible to provide fullname of the user by using the `-c FULLNAME` option .
- `deluser` : The deluser command can be used to remove the user `deluser USER`
- `--remove-all-files` is used to remove all file associated with the user including the home directory.
- `-shell` : The `chsh` command is used to change the default shell for the user .The syntax is `chsh USER -s SHELL`
- `-disable` and `-enable` : The `usermod` command  is used to manipulate several attributes related to user account 
``` bash 
usermod -L USER #locks the user account
usermod -U USER #unlocks the user account
```
- `expiry` the `chage` command is used to manipulate user account expiry
    ``` bash 
    chage -E DATE
    ```
    - `-m` (MIN_DAYS) : set the minimum number of days between password changes to MIN_DAY
    - `-M` (MAX_DAYS) : set the maximum number of days during which a password is valid
    - `-W` (WARN_DAYS) : set the number of days of warning before a password change is required

- `-passwd` : The `passwd` command is used to chnage password for the users .The syntax is `passwd USER`
- `-newgroup` and `addgroup` : The `addgroup` command will add a new user group to the system the syntax is `addgroup GROUP`

- In order to add an existing user to group use : `addgroup USER GROUP`
- The `delgroup` command will remove a user group .The syntax is `delgroup GROUP`
- `-details` : The `finger USER` command will display the user information
- The `chage -l` command will display the user account expiry information.

## Bulk image resizing and format conversion 
- We can resize an image size to a specified image size either by specifying the scale percentage ,or by specifying the width or height as follows 
```bash 
convert img.png -resize WIDTHxHEIGHT image.png
```
?> It is required to provide either WIDTH or HEIGHT ,so that the other will be automatically calculated and resized so as to preserve the image size ratio

-resize the image by specifying the percentage scale factor as follows :
``` bash 
convert image.ong -resize "50%" image.png
```

### A script for image management
``` bash 
#!/bin/bash
#Filename: image_help.sh
#Description: A script for image management

if [ $# -ne 4 -a $# -ne 6 -a $# -ne 8 ];
then
    echo Incorrect number of arguments
    exit 2
fi

while [ $# -ne 0 ];
do
    case $1 in
    -source) shift; source_dir=$1 ; shift ;;
    -scale) shift; scale=$1 ; shift ;;
    -percent) shift; percent=$1 ; shift ;;
    -dest) shift ; dest_dir=$1 ; shift ;;
    -ext) shift ; ext=$1 ; shift ;;
    *) echo Wrong parameters; exit 2 ;;
    esac;
done

for img in `echo $source_dir/*` ;
do
    source_file=$img
    if [[ -n $ext ]];
    then
        dest_file=${img%.*}.$ext
    else
        dest_file=$img
    fi

    if [[ -n $dest_dir ]];
    then
        dest_file=${dest_file##*/}
        dest_file="$dest_dir/$dest_file"
    fi

    if [[ -n $scale ]];
    then
        PARAM="-resize $scale"
    elif [[ -n $percent ]];
    then
        PARAM="-resize $percent%"
    fi

    echo Processing file : $source_file
    convert $source_file $PARAM $dest_file
done
```
This is some usage exmaple 
``` bash 
./image_help.sh -source sample_dir -percent 20%
./image_help.sh -source sample_dir -scale 1024x
./image_help.sh -source sample -scale 50% -ext png -dest newdir
```

## Taking screenshots from the terminal 
- Taking the screenshot of the whole screen : `import -window root screenshot.png`

- Take a screenshot of a specific window : `import -window window-id screenshot.jpg`
?> To find out window-id ,run the command `xwininfo` and click on the window you want

## Managing Multiple termianl from one 
To achieve this ,we will be using a utility called GNU screen .
- Creating screen windows 
    - run the command `screen` from your shell
    - To create a new window ,press Ctrl+A and then C.

- Viewing a list of open windows __Ctrl+a__ and then __"__

- Switching between windows 
    - For the next window __Ctrl+a__ and then __n__
    - For the previous window __Ctrl+a__ and then __p__

- Attaching to and detaching screens
    - detaching : __ctrl+a__ and then __d__
    - attach : `screen -r -d` or `screen -r id`

- screen id can obtain using `screen -list` command
