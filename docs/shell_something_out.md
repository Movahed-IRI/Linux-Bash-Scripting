### introduction

When  a shell is started ,it initially execute a set of commands are read from a shell script at `~/.bashrc` or `~/bash_profile` for login shell .

the bash shell also maintains a history of commands run by the user it's available in the `~/bash_history` or command `history`

## Printing in the terminal

- `echo Hello World !`  <b>we cannot use a semicolon , as it acts as a delimiter between commands in the bash shell.</b>
- `echo 'Hello World !'` <b>Variable subtitution , will not work </b>
- `echo "Hello World \!"` <b>Escape character \ prefixed</b>
- `printf "%-5s %-10s %-4s\n" No Name Mark` 
    - `%-5s` can be described as a sting subtituation with left alignment 
    -  if `-` was not specified , the string would have been aligned to the right.
    - `%f` , `%c` , and `%d` are other format subtitution

### Escaping newline in echo
By default, echo has a newline appended at the end of its output text. This can be avoided by using the `-n` flag.

echo can also accept escape sequences in double-quoted strings as an argument. When using escape sequences, use echo as echo -e "string containing escape sequences" .

For example:
``` bash
echo -e "1\t2\t3"
1   2   3
```

### Printing a colored output

<table>
    <thead>
        <th>color Name</th>
        <th>color Code</th>
        <th>Background Color Code</th>
    </thead>
    <tbody>
        <tr>
            <td>reset</td>
            <td>0</td>
            <td>0</td>
        </tr>
        <tr>
            <td>black</td>
            <td>30</td>
            <td>40</td>
        </tr>
        <tr>
            <td>red</td>
            <td>31</td>
            <td>41</td>
        </tr>
        <tr>
            <td>green</td>
            <td>32</td>
            <td>42</td>
        </tr>
        <tr>
            <td>yellow</td>
            <td>33</td>
            <td>43</td>
        </tr>
        <tr>
            <td>blue</td>
            <td>34</td>
            <td>44</td>
        </tr>
        <tr>
            <td>magenta</td>
            <td>35</td>
            <td>45</td>
        </tr>
        <tr>
            <td>cyan</td>
            <td>36</td>
            <td>46</td>
        </tr>
        <tr>
            <td>white</td>
            <td>37</td>
            <td>47</td>
        </tr>
    </tbody>
</table>

example usage
``` bash 
echo -e "\e[1;31m this is red text \e[0m"
```
## Playing with variables and environment variables

in bash every variable is string regardless of wheter we assign variables with quotes or without quotes. furthermore  , there are variables used by the shell environment and the operating environment variables .
``` bash
cat /proc/$pid/environ
```
set `pid` with a process id of the process with pgrep command as example : 
``` bash 
pgrep gedit
```
the aformentioned command return a list of environment variables and their value each value is represented as a name = value pair and are seprated bu a null char(\0)

``` bash
cat /proc/12501/environ | tr '\0' '\n'
```

variable can be assigned as follows :
``` bash
var = value
```
var is the name of a variable and value is the value to be assigned.

!> if value does not contain space it need not be enclosed in quotes.

?> Note that `var = value` and `var=variable` are different . the later one is an equality operation .

printing contents of a variable is done using by prefixing `$` with the variable name as follow : 
``` bash
echo $var 
```
or
``` bash
echo ${var}
```

?> environment variables are variable that are not defined in current process , but are recieved from parent process.

> the `export` command is used to set the env variable.

to illustrate consider we need to add a new path to the PATH environment variable , we use :
``` bash
export PATH="$PATH:/home/usr/bin"
```

- these are some most used environment variable
 - $HOME
 - $PWD
 - $USER
 - $UID
 - $SHELL

### finding length of a string 
``` bash
length=${#var}
```

### identify the current shell
``` bash
echo $shell
```
or
``` bash
echo $0
```
### Checking for super user
UID is an important environment variable that can be used to check whether the current script has been run as a root user or regular user 

?> the UID value for the root user is 0 .
### Modifying the Bash prompt string (username@hostname:~$)

We can Customize the promp text using `ps1` environment variable which is set using a line in the `~/.bashrc` file.

- `\u` expands to username
- `\h` expands to hostname
- `\w` expands to the current working directory

## Math with the shell
the bash shell environment can perform basic arithmetic operation using the command `let` , `(( ))` , and `[ ]` .

The two utilities `expr` and `bc` are also helpfull in performing advanced operations.
``` bash
let result=no1+no2
let no+=6
result=$[ no1 + no2 ]
result=$(( no1 + 50 ))
result=`expr 3 + 4`
result=$(expr $no1 + 5)
```
All of the preceding method do not support floating point numbers , and operate on integer only.

- `bc` , the precision calculator is an advanced utility for mathematical operations.
``` bash
no=54;
result=`echo "$no * 1.5" | bc`
```
 - Base conversion with `bc`
 ``` bash
 no=1100100 echo "obase=10; ibase=2; $no" | bc
 ```

## Playing with file descriptors and redirection
file describtors are integers associated with an opened file or data stream.

file describtors 0 , 1 and 2 are reserved as follow :
- 0 : stdin (standard input)
- 1 : stdout (standard output)
- 2 : stderr (standard error)

?> you can redirect stderr exclusively to a file and stdout to another file like follows
``` bash
    cmd 2>stderr.txt 1>stdout.txt
```

### redirecting or saving output text to a file
``` bash
    echo "This is a sample Text" > temp.txt
```
### To append text to file
``` bash
    echo "another sample text" >> temp.txt
```
### View The content of file 
``` bash
    cat temp.txt
```
### redirect stderr and stdout to a single file
``` bash
cmd 2>&1 outout.txt
```
or
``` bash
cmd &> output.txt
```
### redirect data to a file as well as provide redirected data to pipe

!> When redirection is performed stderr or stdout , the redirect text flows into a file , so no text remains to flow to the next command through pipe . 

?> There is a way to redirect data to a file as well as provide a copy of redirected data

``` bash
cat a* | tee output.txt | cat -n
```
`-n` puts a line number for each line recieved from stdin.

The `tee` command can read from stdin only .

!> By default , the `tee` command overwrites the file , but it can be used with appended options by providing the `-a` options

### recieve data from stdin

- A command that reads stdin for input can recieve data in multiple ways
 - redirection from a file to a command `cmd < file`
 - redirecting from a text block enclosed within a script
    ``` bash
    #!/bin/bash
    cat <<EOF> log.txt
    LOG FILE HEADER
    This is a test log file
    Finction: System statistics
    EOF
    ```

### custom file descriptors
A file descriptors is an abstract indicator for accessing a file .

We can create out own custom file descriptors using the `exec` command.

create a file descriptors for reading a file :
``` bash
exec 3<input.txt
```
create a file descriptors for writing as follows:
``` bash
exec 4>output.txt
```
create a file descriptors for write (Append mode):
``` bash
exec 5>>input.txt
``` 

## Arrays and associative arrays
regular arrays can use only integers as their array index . On the other hand , bash  also support associative arrays that can take a string as their index.

- An Array can be defined in many ways.
``` bash
array_var=(1 2 3 4 5 6)
```
 - Alternately , define an array as a set of index-value pairs as follow :
 ``` bash 
 array_var[0]="test1"
 array_var[1]="test2"
 ```
- print the content of an array at a given index
``` bash
echo ${array_var[0]}
```
- print all of the values in an array as a list using the following commands:
``` bash
echo ${array_var[*]}
```
or 
``` bash
echo ${array_var[@]}
```
- print the length of an array
``` bash
echo ${#array_var[*]}
```
### defining associative arrays
to use associative arrays , you must have bash version 4 or higher .
``` bash
declare -A ass_array
```
after the declaration , elements can be added to the ass array using two methods.
``` bash
ass_array=([index1]=val1 [index2]=val2)
```
or
``` bash
ass_array[index1]=val1
ass_array[index2]=val2
```

- listing of array indexes
``` bash 
echo ${!array_var[*]}
```
or
```bash
echo ${!array_var[@]}
```

## visiting aliases
an `alias` is basically a shortcut that takes the place of typing a long command sequence.

- an `alias` can be created as follows:
``` bash
alias new_command='command sequence'
```
- the `alias` command is temporary . To keep these shortcuts permanent , add this statement to the `~/.bashrc`
``` bash
echo 'alias cmd="command seq"' >> ~/.bashrc
```
- to remove an alias , remove its entry from `~/.bashrc` or use the `unalias` command


?> as an example , we can create an alias for rm so that it will delete the original and keep a copy in backup directory
``` bash
alias rm='cp $@ ~/backup && rm $@'
```

### escaping aliases
``` bash 
\command
```
!>while running privilaged commands on an untrusted env it is always good security practice to ignore aliases by prefixing command with `\` .

## Grabbing information about the terminal
`tput` and `stty` are utilities that can be used for terminal manipulations. Let us see how tp use them.
- get the number of columns and rows in a terminal:
``` bash
tput cols
tput lines
```
- To print the current terminal name , use the following command
``` bash
tput longname
```
- To move the cursor to a position
``` bash 
tput cup 100 100
```
- Set the background color for the terminal
``` bash
tput setab n
```
?> `n` can be a value in the range of 0 to 7.

- Set the foreground color for text using the follow command
``` bash
tput setaf n
```

- To make text bold use `tput bold`

- To start and end underlining use these commands
``` bash 
tput smul #start underline
tput rmul #remove underline
```

- To delete from the cursor to the end of the line 
``` bash 
tput ed
```
- While typing a password , we should not display the char typed . 
``` bash
#!/bin/bash
echo -e "Enter password:"
stty -echo
read password
stty echo
echo
echo password read
```
?> the `-echo` option in the preceding command disables the output to the terminal , whereas echo enables output

## Getting and setting dates and delays
- You can read the date as follow
``` bash 
date
```
- The epoch time can be printed as follows
``` bash
date +%s
```
- convert the date string into epoch as follows :
``` bash
date --date "Thu Nov 18 08:07:21 IST 2010" +%s
```
?> The `--date` option is used to provide a date string as input. 

- use a combination of format string prefixed with `+` as an argument for the date command. 
``` bash
date "+%d %B %Y"
```

- We can set the time and date as `date -s "Formatted date string"` for example :
``` bash
date -s "21 June 2009 11:01:32"
```
- Check the time taken by a set of commands . 
We can display taken time using the following code :
``` bash
#!/bin/bash
#Filename: time_take.sh
start=$(date +%s)
commands;
statements;
end=$(date +%s)
difference=$(( end - start))
echo Time taken to execute commands is $difference seconds.
```
?> an alternate method would be to use `time <scriptpath>` to get the time that it took to execute the script.

> To Write a date format to get the output as required , use the following table :
<table>
    <thead>
        <th>Date component</th>
        <th>Format</th>
    </thead>
    <tbody>
        <tr>
            <td>Weekday</td>
            <td>
                %A (for e.g:Saturday)
                <br>
                %a (for e.g:Sat)
            </td>
        </tr>
        <tr>
            <td>Year</td>
            <td>
                %Y (for example, 2010)
                <br>
                %y (for example, 10)
            </td>
        </tr>
        <tr>
            <td>Year</td>
            <td>
                %Y (for example, 2010)
                <br>
                %y (for example, 10)
            </td>
        </tr>
        <tr>
            <td>Month</td>
            <td>
                %b (for example, Nov)
                <br>
                %B (for example, November)
            </td>
        </tr>
        <tr>
            <td>Day</td>
            <td>
                %d (for example, 31)
            </td>
        </tr>
        <tr>
            <td>Date in format(mm/dd/yy)</td>
            <td>
                %D
            </td>
        </tr>
        <tr>
            <td>Hour</td>
            <td>
                %I or %H
            </td>
        </tr>
        <tr>
            <td>Minute</td>
            <td>
                %M
            </td>
        </tr>
        <tr>
            <td>Second</td>
            <td>
                %S
            </td>
        </tr>
        <tr>
            <td>NanoSecond</td>
            <td>
                %N
            </td>
        </tr>
    </tbody>
</table>

### producing delays in a scripts
`sleep <sleepno_of_seconds>`
For example . the following script counts from 0 to 40 by using `tput` and `sleep` :
``` bash
#!/bin/bash
#Filename: sleep.sh
echo -n Count:
tput sc
count=0;
while true;
do
    if [ $count -lt 40 ];
    then
        let count++;
        sleep 1;
        tput rc
        tput ed
        echo -n $count;
    else exit 0;
    fi
done
```

## Debugging the script
bash provides certain debugging options that every sysadmin should know .
- Add the `-x` option to enable debug tracing of a shell script as follows :
``` bash
bash -x script.sh
``` 
or
``` bash
sh -x script
```
?> runnig the script with the `-x` flag will print each source line with the current status.

- Debug only portions of the script using `set -x` and `set +x`

- We can setup custom debugging style by passing the `-DEBUG` environment variable as follows
``` bash 
#!/bin/bash
function DEBUG()
{
    [ "$_DEBUG" == "on" ] && $@ || :
}
for i in {1..10}
do
    DEBUG echo $i
done
```
We can  run the above script with debugging set to "on" as follows `_DEBUG=on ./script.sh`

?> In Bash, the command `:` tells the shell to do nothing.

- The `-x` flag output every line of script as it is executed to stdout.
 - `set -x` : This display arguments and commands upon their exec.
 - `set +x` : This disable debugging
 - `set -v` : This displays input when they are read
 - `set +v` : This disables printing input


### shebang hack 
we can make use of shebang in a trickier way to debug scripts.

The shebang can be chnaged from `#!/bin/bash` to `#!/bin/bash -xv` to enable debugging without any additional flags

## Functions And Arguments
- A function can be defined as follows :
``` bash 
function fname()
{
    statements;
}
```
or
``` bash 
fname()
{
    statements;
}
```
- A function can be invoked just by using it's name 
```bash 
fname ;
```

- Arguments can be passed to functions and can be accessed by our script:
``` bash
fname arg1 arg2 ;
fname()
{
    echo $1 , $2 ; #accessing arg1 and arg2
    echo "$@" ; #printing all arguments as list
    echo "$*" ; #Similar to $@ but arguments taken as single entity
    return 0 ; #return value
}
```
 - `$n` : nth argument
 - `$@` : expands as "$1" "$2" "$3" and so on.
 - `$*` : expands as "$1c$2c$3" where c is the first character of IFS

 ?> `$@` : is used more often than `$*` since the former provides all arguments as a single string

### The recursive function
function in bash also support recursion ( The function that call itself)

For example : 
``` bash
F() {echo $1;sleep 1; F hello;}
```
!> We can write a recursive function , which is basically a function that calls itself:
``` bash
:(){ :|:& };:
```
!> It infinitely spawns processes and ends up in a denial-of-service attack. `&` is postfixed with the function call to bring the subprocess into the background. This is a dangerous code as it forks processes and, therefore, it is called a fork bomb.

?> It can be prevented by restricting the maximum number of processes that can be spawned from the config file at `/etc/security/limits.conf`.

### exporting functions
A function can be exported -Like env variables- using export , such that the scope of the function can be extended to subprocesses , as follows :
``` bash 
export -f fname
```

### reading the return value(status) of a command
``` bash 
echo $?;
```
`$?`, will give the return value of the last command.
?> if the command exits successfully , the exit status will be zero , otherwise it will be a nonzero value.

### passing arguments to commands
``` bash 
command -p -v -k 1 file
command -pv -k 1 file
command -vpk 1 file
command file -pvk 1
```

## reading the output of a sequence of commands 
one of the best designed features of shell scripting is the ease of combining many commands or utilities to produce output .
While we combine multiple commands , we usually use stdin to give input and stdout to provide an output.

- we typically use pipes and use the, with the subshell method for combining outputs of multiple files.

For example :
``` bash 
ls | cat -n > out.txt
cmd_output = $(ls | cat -n) # This is called subshell method
cmd_output = `commands` #This method called back quotes  or back tick
```
### spawing a seprate process with subshell
A subshell can be defined using `()`.
``` bash 
pwd ; (cd /bin; ls); pwd;
!> changes are restricted to the subshell
```

## Reading n characters without pressing the return key
- the following statement will read n characters from input into the variable_name variable:
``` bash 
read -n number_of_chars variable_name
```
- Read a password in the nonechoed mode as follows :
``` bash 
read -s var
```

- Display a message with read as follows:
``` bash 
read -p "Enter input:" var
```
- Read the input after a timeout as follows :
``` bash 
read -t timeout var
```
- Use a delimiter char to end the input line using `-d delim_char` as follows:
``` bash 
read -d ":" var
```

## Running a command until it succeeds
define a function in the following way :
``` bash
repeat()
{
    while true
    do
        $@ && return
    done
}
```
or , add this to your shell's rc file for ease of use 
``` bash 
repeat() {while true; do $@ && return; done }
```
### a faster approach
On the most modern systems , true is implemented as a binary in /bin. This means that each time the aforementioned while loop runs, the shell has to spawn a process . To avoid this , we can use the `:` shell built-in, which always return an exit code 0 :
``` bash
repeat() {while :; do $@ && return; done }
```

## Field separators and iterators
The internal field separator(IFS) is an important concept in shell scripting . It is very useful while manipulating text data.

The default value of IFS is a space component (newline , tab , or a space character).

when IFS is set as `,` the shell interprets the comma as a delimiter character , therefore, the variables takes substring seprated by a comma as its value during the iteration.

- Loops are very useful in iterating through a sequence of values . bash provide many types  of loops 
 - using a for loop :
 ``` bash 
 for var in list;
 do
    commands;
 done
 ```
 We can generate different sequences easily , as follows :
 ``` bash 
 for i in {a..z}; do actions; done;
 ```
 The for loop can also take the format of the for loop in C. For example :
 ``` bash 
 for((i=0;i<10;i++))
 {
    commands; # Use $i
 }
 ```
 - using a while loop :
 ``` bash 
 while condition
 do
    commands;
 done
 ```
 > For an infinitive loop , use true as the condition.

 - Using a until loop:
 This execute the loop until the given condition becomes true. For example :
 ``` bash 
 X=0;
 until [ $X -eq 9 ];
 do
   let X++ ; echo $X;
 done
 ```

## Comparisons and test
We will have a look at all the different methods used for compariosons and performing tests:
- Using an if condition:
``` bash
if condition;
then
   commands;
fi
```
- Using else if and else :
``` bash 
if condition;
then
   commands;
else if condition;then
   commands;
else 
   commands;
fi
```
- The if conditions can be lengthy, to make them shorter we can use logical operators as follows:
``` bash 
[ condition ] && action;  #action execute if condition is true
[ condition ] || action;  #action execute if condition is false
```

- Performing methematical comparisons
 ?> Usually conditions are enclosed in square brackets 

 For example :
 ``` bash 
 [ $var -eq 0 ]
 ```
 Other important operators are as follows:
   - `-ne`
   - `-gt`
   - `-lt`
   - `-ge`
   - `-le`

- Multiple test conditions can be combined as follows :
``` bash 
[ $var1 -ne 0 -a $var2 -gt 2 ] # -a represent and
[ $var1 -ne 0 -o $var2 -gt 2 ] # -o represent or
```
- Filesystem related test 
 - `[ -f $file_var ]` : If the given value holds a regular filepath or name
 - `[ -x $var ]` : A file path or file that is executable
 - `[ -d $var ]` : directory path or directory name
 - `[ -e $var ]` : hold existing file
 - `[ -c $var ]` : hold the path of a char dev file
 - `[ -b $var ]` : hold the path of a block device file
 - `[ -w $var ]` : hold the path of a file that is writable
 - `[ -r $var ]` : hold the path of a readable file
 - `[ -L $var ]` : hold the path of a symlink

?> test command can be used for perform condition check . For example : `if test $var -eq 0; then echo "true";fi`

- String comparisons
!> while using string comparison ; it is best to use double square brackets ,since the use of single brackets can sometimes lead to errors.
 - `[[ $str1 = $str2 ]]`
 - `[[ $str1 == $str2 ]]` similar to previous comparison
 - `[[ $str != $str2 ]]` We can check whether strings are not the same
 - `[[ $str1 > $str2 ]]` We can find out the alphabetically smaller or larger
 - `[[ -z $str1 ]]` If str1 holds empty string
 - `[[ -n $str1 ]]` If holds a nonempty string