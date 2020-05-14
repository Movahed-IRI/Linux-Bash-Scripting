## Monitoring disk usage

- To find the disk space used by a file (or files) use `du FILENAME1 FILENAME2`
?> The result is ,by default ,shown as size in bytes .

- `-a` outputs results for all files in the specified direcotry or directories recursively

- In order to print the disk usage in a display-friendly format ,use `-h` format.

- `du` has an option `-c` such that it will output the total disk usage of all files and directories given as an argument . It appends a line `SIZE total` with the result

- There is another option `-s`(summerize) , which will print only the grand total as the output

- We can force `du` to print the disk usage in specified units
    - print the size in bytes(by default) by using `du -b FILE(S)`
    - print the size in kilobytes by using `du -k FILES(S)`
    - print the size in megabyte by using `du -m FILE(S)`
    - print the size in the given BLOCK size specified by using `du -B BLOCK_SIZE FILE(S)`
    ?> Here ,__BLOCK_SIZE__ is specified in bytes.

- Excluding file from the disk usage calculation 
    - We can specify a wildcard as follows `de --exclude "*.txt" DIRECTORY`
    - Exclude list : We can specify a list of files to be excluded from a file as follows : `du --exclude-from Exclude.txt DIRECTORY`

- With the `--max-depth` parameter ,We can specify the maximum depth of the hierarchy `du` should traverse while calculating disk usage
``` bash 
du --max-depth 2 DIRECTORY
```
- `du` can be restricted to traverse only one filesystem by using the `-x` argument .For example `du -x /` will exclude all mount points in /mnt/ for the disk usage calculation.

> Exercise ! : Finding the 10 largest size files from a given directory ?
``` bash 
du -ak SOURCE_DIR | sort -nrk 1 | head
```
when we need to find only the largest files and not directories ,We can improve the one-liner to output only the large files as follows
``` bash 
find . -type f -exec du -k {} \; | sort -nrk 1 | head
```

- Disk free information 
use `-h` with `df` to print the disk space in human-readable format `df -h`

## Calculating the execution time for a command
- To measure the execution time ,just prefix `time` to the command you want to run .It will show real ,user ,and system times for execution.

?> An executable binary of the time command is available at `/usr/bin/time` ,as well as a shell built-in named `time` exists .When we run time ,it calls the shell built-in by default .The shell built-in time has limited options

- We can write these time statistics to a file using the `-o` filename option as follows:
``` bash 
/usr/bin/time -o output.txt COMMAND
```
!> The filename should always appear after the `-o` flag

- In order to append the time statistics ,use the `-a` flag along with the `-o` option as follows 
``` bash 
/usr/bin/time -a -o output.txt COMMAND
```

- We can also format the time outputs using format strings with `-f` option .String consists of parameters corresponding to specific options prefixed with `%`
    - Real time : `%e`
    - User : `%U`
    - Sys : `%S`
    For example :
    ``` bash 
    /usr/bin/time -f "time:%U" -a -o timing.log uname
    ```
    > When a formatted output is produced ,the formatted output of the command is written to the standard output and the output of COMMAND ,which is timed ,is written to standard error

- To show the page size ,use the `%Z` parameters as follows :
``` bash 
/usr/bin/time -f "Page size :%Z bytes " ls > /dev/null
```

- The three different times can be defined as follows 
    - Real : is wall clock time _the time from start to finish of the call .This is included time slices used by other processes and time that the process spends when blocked . 

    - User : is the amount of cpu time spent in user-mode code(outside the kernel) within the process .Other processes ,and the time that the process spends when blocked do not count towards this figure.

    - Sys : is the amount of cpu time spent in the kernel within the process .This means executing the CPU time spent in system calls within the kernel ,as opposed to the library code ,which is still running in the user space .Like user time this is only the CPU time used by the process

- The following table shows some of the interesting parameters that can be used by using format string .
    - `%C` : Name and command-line arguments of the command being timed 
    - `%D` : Average size of the process unshared data area ,in kilobyte
    - `%E` : Elapsed real time used by the process in [hours:minutes:seconds]
    - `%X` : Exit status of the command
    - `%k` : Number of signals delivered to process
    - `%W` : Number of times the process was swapped out of the main memory
    - `%Z` : System's page size in bytes .This is a per-system constant ,but varies between systems .
    - `%P` : Percentage of the CPU that this job got . This is just user+system times divided by the total running time.
    - `%K` : Average total (data + stack + text) memory usage of the process ,in kilobytes.
    - `%w` : Number of times that the program was context-switched voluntarily ,for instance while waiting for an I/O operation to complete
    - `%c` : Number of times the process was context-switched involuntarily (because the time slice expired)

## Collecting information about logged in users ,boot logs ,and boot failures 
- To obtain information about users currently logged into the machine `who`

?> TTY (the term comes from TeleTYpewriter) is the device file associated with a text terminal which is created in /dev when a terminal in newly spawned by the user .The device path for the current terminal can be found out by typing and executing the command `tty`

- To obtain more detailed information about the logged in users ,use `w`

    - `w` present current-time , system uptime , number of users currently logged on ,and the system load averages for the past 1 ,5 ,and 15 minutes in first line of output 
    - Each line contaning the login name ,the TTY name ,the remote host ,login time ,idle time ,total cpu time used by the user since login ,Cpu time of the currently runnning process and the command line of their current process .

- In order to list only the usernames of the users currently logged into the machine use `users`

- In order to see how long the system has been powered on `uptime`

- In order to get information about previous boot and user logged sessions ,use `last`
 
    ?> The `last` command uses the logfile `/var/log/wtmp` for the input log data

    - It is also possible to explicity specify the log file for the `last` command using the `-f` option .For example :
    ``` bash 
    last -f /var/log/wtmp
    ```

- In order to obtain information about login sessions for a single user ,use `last USER`

- Get information about reboot sessions as follows : `last reboot`

- In order to get information about failed user login sessions ,use `lastb`
?> You should run `lastb` as the root user.

## Listing the top 10 CPU consuming processes in an hour 
`ps` command can be used to gather details ,such as CPU usage ,commands under execution ,memory usage ,Status of processes ,and so on.
``` bash 
#!/bin/bash
#Name: pcpu_usage.sh
#Description: Script to calculate cpu usage by processes for 1 hour

SECS=3600 #Change the SECS to total seconds for which monitoring is to be performed.
UNIT_TIME=60 #UNIT_TIME is the interval in seconds between each sampling

STEPS=$(( $SECS / $UNIT_TIME ))

echo Watching CPU usage... ;

for((i=0;i<STEPS;i++))
do
    ps -eocomm,pcpu | tail -n +2 >> /tmp/cpu_usage.$$
    sleep $UNIT_TIME
done

echo
echo CPU eaters :

cat /tmp/cpu_usage.$$ | \
awk '
{ process[$1]+=$2; }
END{
    for(i in process)
    {
        printf("%-20s %s\n",i, process[i]) ;
    }
    }' | sort -nrk 2 | head

rm /tmp/cpu_usage.$$ #Remove the temporary log file
```
In the preceding script ,the major input source is `ps -eocomm,pcpu`
- `comm` stands stands for command name
- `pcpu` stands for the CPU usage in percent
- `$$` in `cpu-usage.$$` signifies that it is the process ID of the current script

## Monitoting command outputs with watch 
the `watch` command can be used to monitor the output of command on the terminal at regular intervals .
- The syntax of watch command is as follows : `watch COMMAND`
?> This command will output at a default interval of two seconds 
- We can also specify the time interval at which the output needs to be updated ,by using `-n SECONDS` .For exmaple :
``` bash 
watch -n 5 'ls -l'
```
- Highlighting the differences in the watch output `watch -d 'COMMANDS'`

## Logging access to files and direcotories 
The `inotifywait` command can be used to gather information about file access.
``` bash 
inotifywait -m -r -e create,move <PATH> -q
```
The previous command will log events ,create ,move from the given path
- The `-m` option is given for monitoring the changes continously
- The `-r` enables a recursive watch of the directories (symbolic links are ignored)    
- The `-e` specifies the list of events to be watched .
- The `-q` is to reduce the verbose messages and print only the required ones .

- We can add or remove the event list .Important events available are as follows :
    - `access` -> when a read happens to a file
    - `modify` -> when file contents are modified 
    - `attrib` -> when metadata is changed
    - `move` -> when a file undergoes a move operation.
    - `create` -> when a new file is created 
    - `open` -> when a file undergoes an open operation.
    - `close` -> when a file undergoes a close operation.
    - `delete` -> when a file is removed 

## Logfile management with logrotate
Management of logfile is required because as time passes ,the size of a logfile gets bigger and bigger .Therefore ,we use techniques called rotation such that we limit the size of the logfile and if the logfile reaches a size beyond the limit ,it will strip the logfile with that size and store the older entries in the logfile archived in log directories .

- `logrotate` has the configuration directory at `/etc/logrotate.d` 
We can write our custom configuration for our logfile (say /var/log/program.log) as follows :
``` bash 
$ cat /etc/logrotate.d/program
/var/log/program.log {
missingok
notifempty
size 30k
  compress
weekly
  rotate 5
create 0600 root root
}
```
- Let's see what each of the parameters in the configuration mean
    - `missingok` -> ignore if the logfile is missing and return without rotating the log
    - `notifempty` -> Only rotate the log if the source logfile is not empty
    - `size 30k` -> limit the size of the logfile fir which the rotation is to be mage .It can be 1M for 1MB 
    - `compress` -> Enable compression with gzip for older log
    - `weekly` -> specify the interval at which the rotation is to be perforemd .It can be __weekly__, __yearly__ ,or __daily__
    - `rotate 5` -> It is the number of older copies of logfile archives to be kept
    - `create 0600 root root` -> specify the mode ,user ,and the group of the logfile archive to be created


## Logging with syslog 
In linux ,creating and writing log information to logfiles at `/var/log` are handled by a protocol called syslog ,handled by the __syslogd__ daemon .We will learn the command `logger` to log into log files with __syslogd__.

- a list  of important logfiles used in Linux:
    - `/var/log/boot.log` Boot log information
    - `/var/log/httpd` Apache web server log
    - `/var/log/messages` Post boot kernel information
    - `/var/log/auth.log` User authentication log
    - `/var/log/dmesg` System boot up messages 
    - `/var/log/mail.log` Mail server log
    - `/var/log/Xorg.0.log` X server log


- In order to log to the syslog file /var/log/messages ,use `logger <LOG_MESSAGE>`
- In order to log to the syslog with a specified tag ,use `logger -t <TAG> <LOG_MESSAGE>`

__syslogd__ decides to which file the log should be made by using __TAG__ associated with the log .You can see the tag strings and associated logfiles from the configuration files located in the `/etc/rsyslog.d` directory

- In order to log in to the system log with the last line from another logfile ,use `logger -f /var/log/source.log`

## Monitoring user logins to find intruders 
Intruders are defined as users who are trying to log in with multiple attempts for more than two minutes and whose attempts are all failing .

Let's write the intruder detection script that can generate a report to intruders by using a authentication logfile as follows :
``` bash 
#!/bin/bash
#Filename: intruder_detect.sh
#Description: Intruder reporting tool with auth.log input
AUTHLOG=/var/log/auth.log

if [[ -n $1 ]];
then
    AUTHLOG=$1
    echo Using Log file : $AUTHLOG
fi

LOG=/tmp/valid.$$.log
grep -v "invalid" $AUTHLOG > $LOG
users=$(grep "Failed password" $LOG | awk '{ print $(NF-5) }' | sort | uniq)

printf "%-5s|%-10s|%-10s|%-13s|%-33s|%s\n" "Sr#" "User" "Attempts" "IP address" "Host_Mapping" "Time range"

ucount=0;

ip_list="$(egrep -o "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" $LOG | sort | uniq)"
for ip in $ip_list;
do
    grep $ip $LOG > /tmp/temp.$$.log

    for user in $users;
    do
        grep $user /tmp/temp.$$.log> /tmp/$$.log
        cut -c-16 /tmp/$$.log > $$.time
        tstart=$(head -1 $$.time);
        start=$(date -d "$tstart" "+%s");

        tend=$(tail -1 $$.time);
        end=$(date -d "$tend" "+%s")

        limit=$(( $end - $start ))

        if [ $limit -gt 120 ];
        then
            let ucount++;

            IP=$(egrep -o "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" /tmp/$$.log | head -1 );
            TIME_RANGE="$tstart-->$tend"
            ATTEMPTS=$(cat /tmp/$$.log|wc -l);
            HOST=$(host $IP | awk '{ print $NF }' )
            printf "%-5s|%-10s|%-10s|%-10s|%-33s|%-s\n" "$ucount" "$user" "$ATTEMPTS" "$IP" "$HOST" "$TIME_RANGE";
        fi
    done
done

rm /tmp/valid.$$.log /tmp/$$.log $$.time /tmp/temp.$$.log 2> /dev/null
```

## Remote disk usage health monitor
We need to collect the disk usage statistics from each machine on the network ,individually ,and write a logfile in the central machine .The script can be scheduled to run every day at a particular time .

We assume that there is a user test in all remote machines configured with auto-login
``` bash 
#!/bin/bash
#Filename: disklog.sh
#Description: Monitor disk usage health for remote systems

logfile="diskusage.log"

if [[ -n $1 ]]
then
    logfile=$1
fi

if [ ! -e $logfile ]
then
    printf "%-8s %-14s %-9s %-8s %-6s %-6s %-6s %s\n" "Date" "IP address" "Device" "Capacity" "Used" "Free" "Percent" "Status" > $logfile
fi

IP_LIST="127.0.0.1 0.0.0.0" #provide the list of remote machine IP addresses

(
for ip in $IP_LIST;
do
    #slynux is the username, change as necessary
    ssh slynux@$ip 'df -H' | grep ^/dev/ > /tmp/$$.df

    while read line;
    do
        cur_date=$(date +%D)
        printf "%-8s %-14s " $cur_date $ip
        echo $line | awk '{ printf("%-9s %-8s %-6s %-6s %-8s",$1,$2,$3,$4,$5); }'

    pusg=$(echo $line | egrep -o "[0-9]+%")
    pusg=${pusg/\%/};
    if [ $pusg -lt 80 ];
    then
        echo SAFE
    else
        echo ALERT
    fi

    done< /tmp/$$.df
done
) >> $logfile
``` 
?> We can use the `cron` utility to run the script at regular intervals .

## Finding out active user hours on system
The log data is stored in the `/var/log/wtmp`
``` bash
#!/bin/bash
#Filename: active_users.sh
#Description: Reporting tool to find out active users

log=/var/log/wtmp

if [[ -n $1 ]];
then
    log=$1
fi

printf "%-4s %-10s %-10s %-6s %-8s\n" "Rank" "User" "Start" "Logins" "Usage hours"

last -f $log | head -n -2 > /tmp/ulog.$$

cat /tmp/ulog.$$ | cut -d' ' -f1 | sort | uniq> /tmp/users.$$


(
while read user;
do
    grep ^$user /tmp/ulog.$$ > /tmp/user.$$
    minutes=0

    while read t
    do
    s=$(echo $t | awk -F: '{ print ($1 * 60) + $2 }')
    let minutes=minutes+s
    done< <(cat /tmp/user.$$ | awk '{ print $NF }' | tr -d ')(')

    firstlog=$(tail -n 1 /tmp/user.$$ | awk '{ print $5,$6 }')
    nlogins=$(cat /tmp/user.$$ | wc -l)
    hours=$(echo "$minutes / 60.0" | bc)

    printf "%-10s %-10s %-6s %-8s\n" $user "$firstlog" $nlogins $hours
done< /tmp/users.$$

) | sort -nrk 4 | awk '{ printf("%-4s %s\n", NR, $0) }'
rm /tmp/users.$$ /tmp/user.$$ /tmp/ulog.$$
```

## measuring and optimizing power usage
- to measaure and optimize power consumption we can use `powertop` utility .Running the `powertop` is pretty easy just run `powertop`

It will show a screen which will have detailed information about power usage ,the process using the most power ,and so on.

- For generating HTML reports ,use : `powertop --html`

- For optimizing power usage ,use the arrow keys to switch to the tunables tab ,just choose whichever ones you want ,press Enter to toggle from Bad to Good

?> If you want to monitor the power consumption from a portable device's battery ,it is required to remove the charger and use the battery for powertop to make measurements

## Monitoring disk activity 
The tool to monitor disk I/O is called `iotop` 
- For interactive monitoring ,use `iotop -o`
    - The `-o` option to iotop tells it to show only those processes which are doing active I/O while it is running

- For non-interactive use from shell scripts ,use : `iotop -b -n 2`
This will tell `iotop` to print the statistics two times and then exit

- Monitor a specific process using the following `iotop -p PID`

?> In most modern distros ,instead of finding PID and supplying it to `iotop` ,you can use the `pidof` command and write the preceding command as ``iotop -p `pidof cp` ``

## Checking disks and filesystems for errors 
Naturally ,it is important to monitor the consistency of data stored on physical media .We will use the standard tool ,`fsck` to check for errors in the filesystems.
- To check for errors on a partition or filesystem `fsck /dev/sdb3`
- To check all the filesystem configured in `/etc/fstab` : `fsck -A`
The fstab file basically configures a mapping between disks and mount points which makes it easy to mount filesystems.

- Instruct `fsck` to automatically attemp fixing errors ,instead of interactively asking us wheter or not to repair `fsck -a /dev/sda2`

- To simulate the actions ,fsck is going to perform `fsck -AN`
- If we run fsck on an ext4 filesystem ,it will end up calling the `fsck.ext4` command
