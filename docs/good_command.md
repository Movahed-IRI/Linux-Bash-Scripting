## Concatenating with cat
- The follows command concatenates data from the files specified as command line arguments.
``` bash 
cat file1 file2 file3 file4
```
- To read from the standard input , use a pipe operation as follow 
``` bash 
output_from_some_commands | cat
echo "Text through stdin" | cat - file.txt  # in This example , - acts as the filename for the stdin text
```
- get rid of extra blank line 
``` bash 
cat -s file
```
- For debugging indentation error use `-T` argument to find tabs as `^I`
``` bash 
cat -T file.py
```
- By using the `-n` flag for the cat command will output each line with a number prefixed . If you want to skip numbering blank lines use `-b` option.

## Recording and playing back of terminal sessions
We can start recording the terminal session like that :
``` bash 
script -t 2> timing.log -a output.session
#commands
exit
```
!> note that this recipe will not work with shells that do not support redirecting only __stderr__ to a file , such as the csh script.

?> By using the two files timing.log and output.session , we can replay the sequence of command execution as follows:
``` bash 
scriptreplay timing.log output.session
```

## Finding files and file listing
- To list all the files and folders from the base_path to the descending 
``` bash 
find base_path
```
- `-print0` argument specifies each matching filename printed with the delimating character `\0` . useful when a filename contains a space character.
- Search based on filename or regular expression match `-name` argument
``` bash 
find /home -name "*.txt" -print
```
- The find command has an option `-iname`(ignore case)
``` bash
find . -iname "example*" -print
```
- If we want to match either of the multiple criteria , We can use OR conditions as shown
``` bash 
find . \( -name "*.txt" -o -name "*.pdf" \)
```
- The `-path` argument can be used to match the file path for files that match the wildcards.
``` bash 
find /home/user -path "*/sedgeek/*" -print
```
- the `-regex` argument is similar to `-path` , but `-regex` matches the file paths based on regular expressions

?> Regular expressions are an advanced form of wildcard matching ,which enable us to specify text with patterns . For example , an email address takes the form __name@host.root__ So it can be generalized as [a-z0-9]+@[a-z0-9]+.[a-z0-9]+.

The `+` sign siginifies that the previous class of char can occur one or more times.

- Similary , using `iregex` ignores the case for the regular expression that are available.
``` bash 
find . -regex ".*\(\.py\|\.sh\)$"
```

- Find can also exclude things that match a pattern using `!`
``` bash 
find . ! -name "*.txt" -print
```

- Search based on the directory depth 
When the find command is used , it recursively wlaks through all the subdirectories as much as possible ,until it reaches the leaf of the subdirectories tree. We can restrict the depth to which the find command traverses.
 - For Specifying the maximum depth we use the `-maxdepth` level parameter. : `find . -maxdepth 1 -name "f*"`
 - To start searching from the second level onwards ,We can set the minimum depth using the `-mindepth` : `find . -mindepth 2 -name "f*" -print`

 !> `-maxdepth` and `-mindepth` should be specified as the third argument tp the find command . If they are specified as the forth or furthur argument ,It may affect the efficiency of find as it has to do unnecessary checks.

- Search based on file type
Unix-like operating system treat every object as a file . The file search can be filtered out using the `-type` option.
 - `f` : Regular file
 - `l` : Symbolic link
 - `d` : Directory
 - `b` : Block device
 - `s` : Socket
 - `p` : FIFO
 - `c` : Character special device

- Search on file times
Unix/Linux filesystems have three types of timestamps on each file.
 - __Access time__ : `-atime` modify last timestamp of when the file content was accessed by user.
 - __Modification Time__ : `-mtime` modify last timestamp of when the file content was modified.
 - __change Time__ : `-ctime` modify last timestamp of when the metadata for a file (Such as permission and ownership) was chnaged

 !> There is no such thing as creation time in unix .

 For Example :
 print all the files that were accessed within the last seven days as follow .
 ``` bash
 find . -type f -atime -7 -print
 ```
 print all the file that are having access time exactly seven day old as follows .
 ``` bash 
 find . -type f -atime 7 -print
 ```
 print all the files that have an access time older than seven days as follow .
 ``` bash 
 find . -type f -atime +7 -print
 ```

 `-atime` , `-mtime` , `-ctime` are time-based parameters that use the time metric in days.

 There are some other time-based parameters that use the `time` metric in minute 
 - __Access time__ `-amin` 
 - __Modification time__ `-mmin`
 - __change Time__ `-cmin`

- Another good feature available with find is the `-newer` parameter . By using `-newer` , we can specify a refrence  file to compare with the timestamp or specified file
``` bash
find . -type f -newer file.txt -print
```
- Search based on size of the file , a search can be performed as follows  `-size 2k`.

instead of `k` we can use different size units :
 - `b` -> byte blocks
 - `c` -> Bytes
 - `w` -> two-byte words
 - `k` -> kilobyte
 - `M` -> Megabyte
 - `G` -> Gigabyte

- The -delete flag can be used to remove file that are matched by find .
``` bash 
find . -type f -name "*.swp" -delete
```

- Match based on the file permissions and ownership
``` bash 
find . -type f -perm 644 -print
```
The file owned by a specific user can be found out using the `-user USER` option

- The find command can be coupled with many of the other commands using the `-exec` option.

For example :
``` bash
find . -type f -user root -exec chown movahed {} \;
```
in this command , `{}` is a specific string used with the -exec option 

!> You must run the find command as root if you want to chnage ownership of files or directories 

?>  Sometimes we dont want to run the command for each file ,instead ,we might want to run it a fewer times with a list of files as parameters . for this , use `+` instead of `;` in exec syntax

- We can  not use multiple commands along with the `-exec` parameter. It accept only a single command ,but we can write multiple commands in a shell script and use it with `-exec` as follow : `-exec ./commands.sh {}\;`

- Skipping specified directories when using the find command

The technique of excluding files and directories from the search is known as pruning . It can be performed as follow :
``` bash
find  devel/sourcepath \( -name ".git" -prune \) -o \( -type f -print \)
```

## playing with xargs 
Some of the commands accept data as command-line argument rather than a data stream through stream through stdin in that case , We cannot use pipes to supply data so we should try alternate methods `xargs`

!> When using the pipe operator ,the `xargs` command should always be the first thing to appear after the operator 

- converting multiple lines of input to a single-line output : `cat example.txt | xargs`
- converting single-line into lines of n argument . `cat example.txt | xargs -n 3`

?> Space is the default delimiter .

- To specify a custom delimiter for input use the `-d` option . 
For example : `echo "splitXsplitXsplitXsplit" | xargs -d x

- passing formated arguments to a command by reading stdin 
  - for executing a command with X arguments per each execution : ` INPUT | xargs -n X`
  - for executing the command at once with all the arguments : `cat args.txt | xargs ./ccat.sh`

- By using `-I` ,we can specify a replacement string that will be replaced while xargs expands .
``` bash
cat args.txt | xargs -I {} ./cecho.sh -p {} -l
```
 - `-I {}` specifies the replacement string. For each of the arguments supplied for the command.
 - The `{}` string will be replaced with arguments read through stdin .

- Using xargs with find 
Let's use find to match and list of all the .txt files and remove them using xargs :
``` bash 
find . -type f -name "*.txt" -print0 | xargs -0 rm -f
```
`xargs -0` interprets that the delimiting character is `\0`.

> as a little project create a script for counting the number of lines of C code in a source directory.
> ``` bash 
> find source_code_dir_path -type f -name "*.c" -print0 | xargs -0 wc -l
> ```
> utility which give more statistivs about source code `SLOCCount`

- while and subshell trick with stdin
    - cause xargs is restricted to providing arguments in limited ways
    - cause xargs cannot supply arguments to multiple set of commands.

``` bash
cat file.txt | ( while read arg; do cat $arg;done)
#Equivalent
cat files.txt | xargs -I {} cat {}
```

## translating with tr
!> `tr` accepts input only through stdin and cannot accept input through arguments .
``` bash
tr [option] set1 set2  #input char from stdin are mapped from set1 to set2.
```
- tr can be used to convert tab characters into space`tr '\t' ' ' < file.txt`
- tr has an option `-d` to delete a set of char.
``` bash
cat file.txt | tr -d '[set1]' #only set1 is used , not set2
```

- Complementing character set 
we can use a set to complement set1 by using the -c flag . The best usage example is to delete all the char from the input text except the one specified in the complement set . For example :
``` bash 
echo hello 1 char 2 next 4 | tr -d -c '0-9 \n'
#1 2 4
```
- Let's use tr in a tricky way to add a given list of numbers from a file and echo sum as follows :
``` bash
cat sum.txt
1
2
3
4
5
#### 
cat sum.txt | echo $[ $( tr '\n' '+' ) 0 ]
```
?> hence we form the string "__1+2+3+4+5+__" , but at the end of the string we have an extra __+__ operator in order to nullify the effect of the + operator , `0` is appended
- Squeezing char with tr
tr provide the `-s` option to squeeze repeating char from the input . It can be performed as follows :` tr -s [set]`
?> tr can use different character classes as sets .The different classes are as follow:
<table>
    <thead>
        <th>
            set
        </th>
        <th>
            Character Classes
        </th>
    </thead>
    <tbody>
        <tr>
            <td>
                alnum
            </td>
            <td>
                alphanumer char
            </td>
        </tr>
        <tr>
            <td>
                alpha
            </td>
            <td>
                alphabetic char
            </td>
        </tr>
        <tr>
            <td>
                cntrl
            </td>
            <td>
                Control (nonprinting) char
            </td>
        </tr>
        <tr>
            <td>
                digit
            </td>
            <td>
                numeric char
            </td>
        </tr>
        <tr>
            <td>
                graph
            </td>
            <td>
                graphic char
            </td>
        </tr>
        <tr>
            <td>
                lower
            </td>
            <td>
                lowercase alphabetic char
            </td>
        </tr>
        <tr>
            <td>
                print
            </td>
            <td>
                printable char
            </td>
        </tr>
        <tr>
            <td>
                punct
            </td>
            <td>
                punctuation  char
            </td>
        </tr>
        <tr>
            <td>
                space
            </td>
            <td>
                whitespace char
            </td>
        </tr>
        <tr>
            <td>
                upper
            </td>
            <td>
                uppercase char
            </td>
        </tr>
        <tr>
            <td>
                xdigit
            </td>
            <td>
                hexadecimal char
            </td>
        </tr>
    </tbody>
</table>

?> We can select the required class and use them as `tr [:class:] [:class:]`


## checksum and verfication
checksum programs are used to generate checksum key strings from the files and verify the integrity of the files later by using that checksum string.
Checksums are crucial while writing backup scripts or maintenance scripts that transfer file through the network.

- To compute the `md5sum` and redirect stdout to file_sum.md5 , use the following command : `md5sum filename > file_sum.md5`

- The integroty of a file can be verified by using the generated file as follow : `md5sum -c file_sum.md5`


- To use __sha-1__ instead of md5 simply replace `md5sum` with `sh1sum` in all the commands previously mentioned . instead of file_sum.md5 , change the output filename to file_sum.sha1

- Checksum for directories can be achieved by the `md5deep` and `sh1deep.

Calculating the shecksum for a directory would mean that we would need to calculate the checksum for all the files in the directory
``` bash
md5deep -rl directory_path > direcotry.md5
```

## Cryptographic tools and hashes 
- The crypt command is a simple and relatively insecure cryptographic utility that take a file from stdin and a cryptographic utility that take a file from stdin and a passphrase as input and output encrypted data into stdout.
``` bash
crypt passphrase <input-file >encrypted-file
```
or in order to __decrypt__ the file ,use :
``` bash
crypt passphrase -d <encrypted_file >output_file
```

- gpg signature that are also widely used in e-mail communication to sign email message ,proving the authenticity of the sender
``` bash
gpg -c filename #This command generate filename.gpg 
gpg filename.gpg #in order to decrypt a gpg file
```
- __Base64__ inorder to encode a binary file into the Base64 format, use :
``` bash
base64 filename > outputfile 
```
decode Base64 data as follows 
``` bash
base64 -d file >  outputfile
```
md5sum and sha1 are undirectional hash algorithm ,which cannot be reversed to form the original data.

?> its recommended to use tools such as `bcrypt` or `sha512sum` 

### Shadow-like hash (salted hash)
``` bash
opensslpasswd -1 -salt SALT_STRING PASSWORD
```
 - Replace __SALT_STRING__ with a random string and __PASSWORD__  with the password you want to use.


## sorting unique and duplicates
- We can easily sort a given set of files as follows :
``` bash 
sort file1.txt file2.txt > sorted.txt
```
or 
```bash
sort file1.txt file2.txt -o sorted.txt
```

- For a numerical sort , We can use `sort -n file.txt`

- For sorting in the reverse ,we can use `sort -r file.txt`

- For sorting by month , use `sort -M month.txt`

- To merge to already sorted files use `sort -m sorted1 sorted2`

- To find the unique lines from a sorted file , use : `sort file1.txt file2.txt | uniq`
?> uniq can be applied only for sorted data input 

- For checking if a file is already sorted or not, we explit the fact that sort return :
    - an exit code of `0` if the file is sorted 
    - and a `nonzero` otherwise

- Sorting according to the keys or columns 
    - `-k` key is the column number by which sort is to be done . For example sort reverse by column1  :
    ``` bash
    sort -nrk 1 data.txt
    ```
    - in order to specify numeric sort the `-n` option should be provided . 

- To use the first character instead of first column as the key ,use:
``` bash
    sort -nk 1,1 data .txt
```

- To make the sort's output `xrags` compatible with `\0` terminator ,use the following command
``` bash 
sort -z data | xargs -0
```

- To sort text in dictionary order , by ignoring punctuation and folds , use :
``` bash 
sort -bd unsorted.txt
```

### uniq
 - Display only unique lines `uniq -u sorted.txt`
 - to count how many times each of the lines appear in the file`sort unsorted.txt | uniq -c`
 - To find duplicate lines in the file `sort unsorted.txt | uniq -d`
 - To specify keys ,We can use the combination of the `-s` and `-w` arguments.
    - `-s` -> The number fot the first N char to be skipped 
    - `-w` -> The maximum number of char to be compared

    ?> This comparison key is used as the index for the uniq operation.
 - zero-byte-terminated output can be generated from the uniq command as follows : 
    ``` bash 
    uniq -z file.txt
    ```


## Temprorary file naming and random numbers
the most suitable location to store data is `/tmp` (which will be cleaned outby the system on reboot.)

- We can use 2 method to generate standard filename for temp data
    - ````filenae=`mktemp` ````

    - ```bash 
    mktemp -u  #dry-run - Just generate file name
    ```


- To create a temporary dir `mktemp -d `

- To create the temporary filename according to a template ,use `mktemp test.XXX` #X replaced with one rand char.

## Splitting file and data
- Let's say we have a test file called data.file ,which has a size of 100 KB .You can split this file into smaller files of 10k each as follow `split -b 10k data.file`

- The chunks will be named the manner __xab__ ,__xac__ ,__xad__ ,and so on .This mean it willl have alphabetic suffixes to use the numeric suffixes ,use an additional `-d` argument

- it's also possible to specify a suffix length using `-a` lingth : `split -b 10k data.file -d -a 4`

- instead of the `k`(kilobyte) suffix 
    - M -> MB
    - G -> GB
    - c -> byte
    - w -> word

- Specifying a filename prefix for the split files : `split [command-args] prefix`

let's run the previous command with the prefix filename for split files :
``` bash
split -n 10k data.file -d -a 4 split-file
```
- To split files based on the number of lines in each split rather than chuck size use  `-l no-of-lines` as follows :
``` bash
split -l 10 data.file
```

### csplit
There is another interesting utility called csplit . the `split` utility can only split file :
 - based on chunk size
 - based on the number of lines 

?> `csplit` makes the split based on context based split.

- For example split the files from the contents for each server in each file :
``` bash 
csplit server.log /SERVER/ -n 2 -s {*} -f server -b "%02d.log" ; rm server00.log
```
 - __/SERVER/__ : is the line used to match a line by which a split is to be carried out .
 - __/[REGEX]/__ : is the format . it copies from the current line up to the matching line that contain "SERVER" excluding the match line
 - __{*}__ is used to specify to repeat a split based on the match up to the end of the file .By using {integer} ,we can specify the number of times it is to be continued .
 - __-s__ flag , is the flag to make the command silent
 - __-n__ is used to specify the number of digits to be used as suffix. 01,02,03,and so on .
 - __-f__ is used for specifying the filename prefix for split files (server is the prefix in the previous example)
 - __-b__ is used to specify the suffix format . __%02d.log__ is similar to the printf argument format in C .Here ,the filename = prefix + suffix ,that is "server" + %02.log",

 ?> we remove server00.log since the first split file is an empty file (the math word is the first line of the file).

## slicing filenames based on extension
- The name from name.extension can be easily extracted using the `%` operator
``` bash 
file-jpg='sample.jpg'
name=${file-jpg%.*}
```
?> remove the string match from __file-jpg__ for the wirldcard pattern that appears to the right-hand side of `%`

 - The `%` operator perform a nongreedy match for .*

 - The `%%` operator perform a greedy match.

- extract the extension of a file from its filename using the `#` operator
``` bash 
extension=${file-jpg#*.}
```
?> `#` , we have used to extract the extension from the filename. it is similar to `%` .But it evalutes from left to right.

?> similary ,as in the case of `%%` we have another greedy operator for `#` ,which is `##` .


## Renaming and moving files in bulk

- To replace space in the filenames with the "_" char.
``` bash 
rename 's/ /_/g' *
```
?> `s/ /_/g` is the replacement part in the filename and * is the wildcard for the target files .

- To convert any filename of files from upercase to lowercase 
``` bash 
rename 'y/A-Z/a-z/' *
rename 'y/a-z/A-Z/' *
```

- To recursively move all the .mp3 files to a given directory 
``` bash
find path -type f -name "*.mp3" -exec mv {} target-dir \;
```

## Spell checking and dictionary manipulation
There is a command-line utility called aspell that function as spell checker

?> The `/usr/share/dict` directory contains some of the directory files .


> Creating dictionary tips : 
in grep , `^` is the word-start-marker char and the `$` char is the word-end-marker 
 - -q is used to suppress any output and to be silent
 - List all word in a file starting with a given word 
    - `look word filepath`
    - `grep "^word" filepath`


the `aspell` list command return text when the given input is not a dictionary word ,and does not output anything when the input is a dictionary word.

## Automating interactive input

``` bash
#!/bin/bash
#Filename: interactive.sh
read -p "Enter number:" no ;
read -p "Enter name:" name
echo You have entered $no, $name;
```
Let's automate the sending of input to the command as follows:
``` bash 
echo -e "1\nhello\n" | ./interactive.sh
```
?> We have used `echo -e` to produce the input sequence where -e signal to interpret escape sequences.


?> If you are a reverse engineer, you may have played with buffer overflow exploits. To exploit them we need to redirect a shellcode such as "\xeb\x1a\x5e\x31\xc0\x88\x46" , which is written in hex. These characters cannot be typed directly through the keyboard as keys for these characters are not present in the keyboard. Therefore, we should use: `echo -e \xeb\x1a\x5e\x31\xc0\x88\x46"` This will redirect the shellcode to a vulnerable executable.

### Automating with expect
The expect command supplies the correct input for the correct input promp by the program.
`expect` expects for a particular input promp and sends data by checking messages in the input prompt.

``` bash 
#!/usr/bin/expect
#Filename: automate_expect.sh
spawn ./interactive .sh
expect "Enter number:"
send "1\n"
expect "Enter name:"
send "hello\n"
expect eof
```

- the `spawn` param specifies which commands are to be automated .
- the `expect` param provides the expected message.
- the `send` is the message to be sent.
- the `expect eof` de defines the end of the command inteeraction.

## Making commands quicker by running parallel processes

the multiple cores are useless unless the software makes use of them.

For instance  ,we can run multiple instances of md5sum using a script like this : 
``` bash
#/bin/bash
#filename: generate_checksums.sh
PIDARRAY=()
for file in File1.iso File2.iso
do
    md5sum $file &
    PIDARRAY+=("$!")
done
wait ${PIDARRAY[@]}
```
?> When we run this , we get the output that will be same as running md5sum . However, as the md5sum command ran simultaneously  you'll get the result quicker .

- `&` intructs the shell to send the command to the background .
- `$!` We get the PIDs of the processes using `$!` , which in bash hold the PID of the last background process .

!> we apppend these PIDs to an array and then use the `wait` command to wait for these processes to finish . This is essential to prevent exit script as soon as loop completes. 

