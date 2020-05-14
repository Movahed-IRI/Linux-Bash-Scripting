## Generating files of any size 
Loopback files are files that can contain a filesystem itself and these files can be muted similary to a physical device usin the mount command.

- Use the `dd` command to create alarge-size file with th given size
    - input can be stdin
    - output can be stdout
    - `dd` operate on a very low level with the device so be careful while using the dd command.

``` bash
ff if=/dev/zero of=junk.data bs=1M count=1
```
the preceiding command will create a file called junk.data that is exactly 1 MB in size.
   - `if` stands for the input file
   - `of` stands for the output file
   - `bs` stands for bytes for a block
   - `count` stands for the number of blocks of `bs` specified to be copied.

we can use various unit for block size (`bs`)
<table>
    <thead>
        <th>
            unit size
        </th>
        <th>
            code
        </th>
    </thead>
    <tbody>
        <tr>
            <td>
            Byte(1B)
            </td>
            <td>
            c
            </td>
        </tr>
        <tr>
            <td>
            word(2B)
            </td>
            <td>
            w
            </td>
        </tr>
        <tr>
            <td>
            Block(512B)
            </td>
            <td>
            b
            </td>
        </tr>
        <tr>
            <td>
            kilobyte(1024B)
            </td>
            <td>
            k
            </td>
        </tr>
        <tr>
            <td>
            megabyte(1024KB)
            </td>
            <td>
            M
            </td>
        </tr>
        <tr>
            <td>
            Gigabyte(1024MB)
            </td>
            <td>
            G
            </td>
        </tr>
    </tbody>
</table>

?> the `dd` command can also be used to measure the speed of memory operations by transfering a large quantity of data and checking the command output

## The intersection and set difference on text files
the `comm` command is a utility to perform a comparioson between the two files .
We can perform intersection ,difference ,and set difference operations.

- intersection : the intersection operation will print the lines that the specified files have in common
- Difference : the difference operation will print the lines that specified files contain and that are not the same
- Set difference : the set difference operation will print the lines in file "A" that do not match these in all of the set of files specified("B" plus "C" for example)

!> Note that `comm` takes only sorted file as input

``` bash
sort A.txt -o A.txt ; sort B.txt -o B.txt
comm A.txt B.txt
```
>The first column of the output contains lines that are only in A.txt
>The second column contains lines that are only in B.txt
>The third column contains the common lines from A.txt and B.txt

each of the columns are delimited using the tab (\t) char.

print lines that are uncommon in two files and produce a unified output.
``` bash
comm A.txt B.txt -3 | sed 's/^\t//'
```
?> the cmd-line options for comm format the output as per our requirement.
- `-1` -> remove the first column from the output
- `-2` -> remove the second column
- `-3` -> remove the third column

## Fincding and deleting duplicate files
We can identify the duplicate files by comparing file content .Checksums are ideal for this task ,since files with exactly the same content will produce the same checksum values

The code for the script to remove the duplicate files is as follow
``` bash 
#!/bin/bash
#Filename: remove_duplicates.sh
#Description: Find and remove duplicate files and keep one sample of each file.

ls -lS --time-style=long-iso | awk 'BEGIN {
  getline; getline;
  name1=$8; size=$5
}
{
  name2=$8;
  if (size==$5)
  {
    "md5sum "name1 | getline; csum1=$1;
    "md5sum "name2 | getline; csum2=$1;
    if ( csum1==csum2 )
    {
      print name1; print name2
    }
  };
  size=$5; name1=name2;
}' | sort -u > duplicate_files

cat duplicate_files | xargs -I {} md5sum {} | sort | uniq -w 32 |
awk '{ print "^"$2"$" }' | sort -u > duplicate_sample

echo Removing..

comm duplicate_files duplicate_sample -2 -3 | tee /dev/stderr |
xargs rm
echo Removed duplicates files successfully.
```

## Working with file permissions ,ownership ,and the sticky bit
Permissions of a sile can be listed by usin the `ls -l` .

The first column of the output specifies the following ,with the first letter corresponding to :
- `-` -> if it is a regular file
- `d` -> if it is a directory
- `c` -> for a character device
- `b` -> for a block device
- `l` -> if it is a symbolic link
- `s` -> for a socket
- `p` -> for a pipe

- The user has one more special permission called setuid(`s`) which appears in the position of execute(`x`) .The setuid permission enables an executable file to be executed effectively as its owner ,even when the executable is run by another user.

?> instead of setuid ,the group has a setgid(s) bit.

- the read ,write ,and execute permissions are also applied to the directories .However ,the meaning of read ,write ,and execute permissions are slightly different in the context of directories .
> The read permission for the directories enables reading the list of files and subdirectories in the directory.
> The write permission for a directory enables creating or removing files and directories from a directory
> The execute permission(x) specifies whether the access to the files and directories in a directory is possible or not.


- Directories have a special permission called sticky bit. When a sticky bit is set for a directory ,only the user who created the directory can delete the files in the directory ;even if the group and others have write permissions .The sticky bit appears in the posiotion of execute char(x) in the others permission set . it represented as `T` # `-------rwt`

this could be set using chmod  ` chmod u=rwx g=rw o=r filename`

- use `+` to add permission to user,group,or others and use `-` to remove the permissions.

For example add the executable permission to all permission categories
``` bash 
chmod a+x filename
```

- Read ,write ,and execute permission have unique octal numbers as follow :
    - `r--` = 4
    - `-w-` = 2
    - `--x` = 1

we can get the required combination of permission by adding the octal values for the required permission set . For example __rw- = 4+2 = 6__ the command for setting the permissions using octal value is :
``` bash
chmod 764 filename
```

- In order to change ownership of files ,use chown command `chown user:group filename`

- Setting sticky bit
In order to set sticky bit `+t` is applied on a directory with `chmod` as follow :
``` bash
chmod a+t directory-name
```

- Applying permissions and owner recursively to files 
The `-R`  option specifies to apply chnage to a permission or ownership recursively
``` bash
chmod 777 . -R
chown user:group . -R
```

- Running an executable as different user(setuid)
    - First , change the ownerwhip to the user that needs to execute it 
    - log in as the user.
    - Then run the following command:
    ``` bash
    chmod +s executable_file
    ```

## making files immutable 
Files on extended type filesystems ,which are comman in Linux (for example ,ext2 ,ext3 ,ext4 ,and so on) can be mode immutable using a certain type of file attribute

> we can easily find out the filesystem type of any mounted partition by looking at the `/etc/mtab` file

The `chattr` command can be used to make files immutable . 
- A file can be made immutable using the following command. `chattr +i file`
- In order to make it writable ,remove the immutable attribute as follow `chattr -i file`


## Generating blank files in bulk
touch is a command that can create blank files or modify the timestamp of files if they already exist.

- A blank file with the name filename will be created using command ` touch filename`
- If we want to specify that only certain stamps are to be modified ,we use the following option :
    - `touch -a` -> modify only access time
    - `touch -m` -> modify only the modification time

- instead of using current time for the timestamps ,we can specify the time and date with which to stamp the file follow :
``` bash
touch -d "Fri jun 25 20:50:14 IST 1999" filename 
```

## Finding symbolic links and their targets 
symbolic links are just pointers to other files .When symbolic links are removed ,they will not cause any harm to the original file.

- We can create a symbolic links as follows `ln -s target symbolic_link_name`

- To verify that the link was created `ls -l web`

- In order to print symbolic links in current direcotory `ls -l | grep "^l"`

- Use find to print all symbolic links from the current directory and subdirectories `find . -type l -print`

- To read the target path for a given symbolic link use the readlink command `readlink sybmbolic_link_name`

## Enumerating file type statistics 
> write a script that can enumerate through all the files inside a directory ,its descendants ,print a report that provides details on type of files.

The file command can be used to find out the type of file by looking at the content of the file .
- To print the type of file use the following command : `file filename`
- print the file type only by excluding the filename as follows : `file -b filename`

``` bash 
#!/bin/bash
# Filename: filestat.sh
if [ $# -ne 1 ];
then
    echo "Usage is $0 basepath";
    exit
fi
path=$1

declare -A statarray;

while read line;
do
    ftype=`file -b "$line" | cut -d, -f1`
    let statarray["$ftype"]++;
done < <(find $path -type f -print)

echo ============ File types and counts =============

for ftype in "${!statarray[@]}";
do
    echo $ftype : ${statarray["$ftype"]}
done
```
The usage is as follows : `./filestat.sh /home/movahed/temp`

- We use `cut -d, f1` ,which  specifies to use ,as the delimiter and print only the first field .
- `done < <(find $path -type f -print);` is an important bit of code . the logic is as follows:
``` bash
while read line;
    do something
done < filename
```
instead of the filename we used the output of find note that the first `<` is for input redirect and the second `<` is for converting the subprocess output to a filename

?> In bash 3.x and higher we have a new operator `<<<` that lets us use a string output as an input file `` done <<< "`find $path -type f -print`" ``

- ${!statarray[@]} is used to return the list of array indexes.

## Using loopback files 
We can mount those files as filesystems at a mount point .This essentially lets you create logical "disks" inside a file on your physical disk !

``` bash 
dd if=/dev/zero of=loopbackfile.img bs=1G count=1
## now format the 1GB file to ext4 using the mkfs command 
mkfs.ext4 loopback.img
## now you can mount the loopback file as follows 
mkdir /mnit/loopback
mount -o loop loopbackfile.img /mnt/loopback
```

- The `-o` loop additional option is used to mount loopback filesystems. internally it attaches to a device called `/dev/loop1` pr loop2.

- We can attach to a loopback device manually as follows : `losetuo /dev/loop1 loopbackfile.img`

- To unmount use ,the following sysntax : `unmount mount-point`
    - We can use the device file path as an argument to the unmount commands as : ` ummount /dev/loop1`

?> Note that the mount and unmount commands should be axecuted as a root user since it is privileged command .

### Creating partitions inside loopback images 
We have to manually set up the device and mount the partitions in it 
``` bash 
losetup /dev/loop1 loopback.img
fdisk /dev/loop1
losetup -o 32256 /dev/loop2 loopback.img
```
- `-o` is the offset flag ,32256 bytes are for a DOS partition scheme
- `/dev/loop2` will represent the first partition

?> However ,there is a quicker way to mount all the partitions inside such an image using `kpatx`

``` bash 
kpartx -v -a diskimage.img
```

This creates mappings from the partitions in the disk image to devices in `/dev/mapper` which you can then mount .For example ,To mount the first partition use the following command :
``` bash
mount /dev/mapper/loop0p1 /mnt/disk1
```
when you're done with the devices (and unmounting any mounted partitions using unmount) ,remove the mapping by `kpartx -d diskimage.img`

We can even use a nonempty directory as the mount path .Then ,the mount path will contain data from the device rather than the original contents untill the device in unmounted.

> __iso__ is a read-only filesystem.


### Flush changing immediately with sync
while making chnages on a mounted device ,they are not immediately written to the physical devices .They are only written when the buffer is full .But ,We can force writing of changes immediately by using th sync command as follows : `sync`



## Creating ISO files and hybrid ISO
- In order to create an ISO image from __/dev/cdrom__ 
``` bash 
cat /dev/cdrom > image.iso
#or
dd if=/dev/cdrom of=image.iso
```

- `mkisofs` is a command used to create an ISO system .

- The output file of `mkisofs` can be written to CD-ROM or DVD-ROM using utilities such as `cdrevord` .
``` bash 
mkisofs -v "Label" -o image.iso source_dir/
```
the `-o` option in the `mkisofs` command specifies the ISO file path.

the `source_dir/` command is the path of the directory that should be used as source content for the ISO and the `-v` option specifies the label that should be used for ISO file .

### Hyrid ISO that boots off a flash drive or hard disk 
- We can convert standard ISO files into hybrid ISos with the `isohybrid` command

- To write the ISO to a usb storage device 
``` bash 
dd if=image.iso of=/dev/sdb1
#or
cat image.iso >> /dev/sdb1
```

### Burning an ISO from the command line 
`cdrecord` can be used to burn the image to the CD-ROM as follows :
``` bash 
cdrecord -v dev=/dev/cdrom/ image.iso
```
- We can specify the burning speed with the -speed option as follows 
``` bash 
cdrecord -v dev=/dev/cdrom image.iso -speed 8
```
Here ,8 is the speed specified as 8x.

- Multisession burning can be performed using the -multi option as follows :
``` bash 
cdrecord -v dev=/dev/cdrom image.iso -multi
```

- Playing with the CD-ROM tray 
 - This command is used to eject the tray `eject`
 - This command is used to close the tray `eject -t`

## Finding the difference between files ,patching
We can add the changes specified in the patch file to the original source code by using the `patch` command . We can also revert the changes by patching again.

- The `diff` command utility is used to generate difference files 
    - Nonunified diff output `diff version1.txt version2.txt`
    - The unified diff outpput `diff -u version1.txt version2.txt`

    > In unified diff ,the lines starting with `+` are the newly-added lines and the lines starting with `-` are the removed lines .

- A patch file can be generated by redirecting the diff output to a file `diff -u version1.txt version2.txt > version.patch`

- To apply the patch ,use the following command 
``` bash 
patch -p1 version1.txt < version.patch
# We now have version1.txt with the same contents as that of version2.txt
```

- To revert the chnages back ,use the following command :
``` bash 
patch -p1 version1.txt < version.patch # :)
```
?> As shown ,patching an laready patched file reverts back the chnages .

- To avoid prompting the user with y/n ,we can use the `-R` option along with the `patch`  command .

- The `diff` command can also act recursively against directories `diff -Naur directory1 directory2`

    - `-N` is for treating absent files as empty
    - `-a` is to consider all files as text files 
    - `-u` is to produce unified output
    - `-r` is to recursively traverse through the files in the directories .

## Using head and tail for printing the last of first lines

- print the first 10 lines as follows : `head file`
- Read the data from stdin as follows : `cat text | head`
- Specify the number of first lines to be printed as follows :`head -n 4 files`
- print all the lines excluding the last M lines as follows : ` head -n -m follows` 
?> Note that it is negative M.

- print the last 10 lines of a file as follows :` tail file`
- in order to read from stdin ,you cab use the following command line `cat text | tail`
- print the last five lines as follows : `tail -n 5 file`
- In order to print all lines excluding the first  M lines : `tail -n (M+1)`

For Example :
``` bash 
seq 100 | tail -n +6 #This will print from 6 to 100
```

### One of the important usage of tail is to read a constantly growing file
tail has a special option `-f` or `--follow` ,which enables tail to follow the appended lines and keep being updated as data is added `tail -f growing_file`

For example:
``` bash 
dmesg | tail -f
```
We frequently run `dmesg` to look at kernel ring buffer messages either to debug the USB devices or to look at sdx .

- The `-f` tail can also add a sleep interval `-s` , so that we can set the interval during which the file updates are monitored.

- `tail` has the interesting property that allows it to terminate after a given process ID dies.
``` bash 
PID=$(pidof foo)
tail -f file --pid $PID
```

## Listing only directories 
 - Using `ls` with `-l` to print direcotries `ls -dl */`
 - Using `ls -F` with grep : `ls -F | grep "/$`
 - Using `ls -l` with grep : `ls -l | grep "^d"`
 - Using find to print directories : `find . -type d -maxdepth 1 -print`


## Fast command-line navigation using pushd and popd
`pushd` and `popd` are used to switch between multiple directories. `pushd` and `popd` opera on a stack .We know that a stack is a last in first out (LIFO) data structure

We Omit the use of the `cd` command while using `pushd` and `popd`

- To push and change a directory to a path `pushd /var/w`
- now ,again push the next directory path as following : `pushd /usr/src`
- To view the stack contents ,use the following command : `dirs`
- Now when you want to switch to any path in the list ,number each path from 0 to n ,then use the path number for which we need to switch ,for example :`pushd +3`
?> `pushd` will always add paths to the stack
- To remove a last pushed path and change directory to the next directory ,use the following command : `popd`
- To remove specific path from the list ,use : `popd +num`

?> `num` is counted as 0 to n from left to right

?> Change to most frequently used directory as follows : `cd -`

## Counting the number of lines ,Words ,and char in a file 
 - Count the number of lines in the following manner : `wc -L file`
 - To use stdin as input ,use the following command : `cat file | wc -L`
- Count the number of words as follows : `wc -w file`

- In order to count the number of char : `wc -c file`

- To print the number of lines ,words ,and characters ,execute `wc` without any options .

- Print the length of the longest line in a file : `wc file -L`

## Printing the directory tree 
the `tree` command is the hero  that helps us to print graphical trees of files and directories 

- To highlight only files matched by the pattern 
``` bash 
tree path -P PATTERN #pattern should be wildcard
```
- To hightlight only files excluding the match pattern 
``` bash 
tree path -I PATTERN
```
- To print the size along with files and directories
``` bash 
tree -h
```
- To create an HTML file wilh the `tree` output :
``` bash 
tree PATH -H http://localhost -o out.html
```
Replace http://localhost with the URL where you are planning to host the file
