## Archiving with tar
It allows you to store multiple files and directories as a single file while retaining all the file attributes ,such as owner ,permissions ,and so on.

- To archive files with tar ,use the folowing syntax : ` tar -cf output.tar [SOURCE]`

- To list files in an archive ,use the `-t` option `tar -tf archive.tar`

- In order to print more details while archiving or listing ,use the `-v` or `-vv`  flag

- The filname must appear immediately after the `-f` and it should be the last option in the argument group 

- In order to append a file into an already existing archive use 
``` bash 
tar -rvf original.tar new-file
```

- the following command extracts the contents of the archive to the current directory `tar -xf archive.tar`

- We can also specify the directory where the files need to be extracted by using the `-C` flag :
``` bash 
tar -xf archive.tar -C /path/to/extraction_direcotry
```
- We can also extract only a few files by specifying them as command arguments `tar -xvf file.tar file1 file4`

- stdin and stdout as the output file so that another command appearing through a pipe can read it as stdin
``` bash 
tar -cvf - files/ | ssh user@example.com "tar xv -C Documents/"
```

- containing two archives 
We can easily merge multiple tar files with the `-A` option 
``` bash 
tar -Af file.tar file2.tar
```

- Updating files in an archive with a time stamp check  
We can use the update option `-u` to specify only append files that are newer than the file inside archive with the same name.
``` bash 
tar -uf archive.tar filea
```

- Comparing files in the archive and file system 
the `-d` flag can be used to print the differences 
``` bash 
tar -df archive.tar
```
To know wheter files in the archive and the files with the same filename in the filesystem are the same or contain any defferences

- Deleting files from the archive
We can remove files from a given archive using the `-delete` option
``` bash 
tar -f archive.tar --delete file1 file2
```

- Comparison with the tar archive 
    The tar command only archive files ,it does not compress them ,Tarballs are often compressed into one of the following formats 
    - file.tar.gz
    - file.tar.bz2
    - file.tar.lzma

    Different tar flags are used to specify different compression formats :
    - `-j` for bunzip2
    - `-z` for gzip
    - `--lzma` for lzma

    In order for tar to support compression automatically by looking at the expression ,use `-a` or `--auto-compress` with tar
    ``` bash 
    tar -acvf archive.tar filea fileb filec
    ```

- Excluding a set of files from archiving 
use `--excluse [PATTERN]` for excluding files watched by wildcard patterns . For example :
``` bash 
tar -cf arch.tar * --exclude "*.txt"
```
?> Note that the pattern should be enclosed within quotes to prevent the shell from expanding it

- It is also possible to exclude a list of files provided in a list file with the `-X` flag as follows :
``` bash 
tar -cf arch.tar * -X list
```
- Excluding version control directories 
In order to exclude version control related files and directories while archiving use the `--exclude-vcs` option along with tar . For example 
``` bash 
tar --exclude-vcs -czvvf source_corde.tar.gz file1 
```

- Printing total bytes
To print the total butes copied after archiving use the `--total` option .


## Archiving with cpio
- We can archive the files as follows :
    ``` bash
    echo file1 file2 file3 | cpio -ov > archive.cpio
    ```
    - `-o` specifies the output
    - `-v` is used for printing a list of files archived 
- In order to list files in a cpio archive use the following command
    ``` bash 
    cpio -iy < archive.cpio
    ```
    - `-i` is for specifying the input
    - `-t` is for listing
- In order to extract files from the cpio archive use :
    ``` bash 
    cpio -id < archive.cpio
    ```
    - `-d` stands for extracting and cpio oversrites files without prompting.

## Compressing data with gzip
gzip can be applied only on a single file or data stream .Hence ,we must first create a tar archive and compress it with gzip.
- In order to compress a file with gzip `gzip filename`
- extract a gzip compressed file as follows `gunzip filename.gz`
- In order to list out the properties of a compressed file use 
``` bash 
gzip -l test.text.gz
```
- The gzip command can read a file from stdin and also write a      compressed file into stdout .Read data from stdin and output the compressed data to stdout as follows :
    ``` bash 
    cat file | gzip -c > file.gz
    ```
    - The `-c` option is used to specify output to stdout

- We can specify the compression level for gzip using `--fast` or `--best` option to provide low and high compression ratios ,respectively.

- gzip with tarball
    - `tar -czvvf archive.tar.gz [FILES]`
    ?> or use -a option to automatically be detected from the extension.
    - alternatively ,we can first create a tarball and then compress the tarball through gzip command .

    ?>  if many files are to be archived in a tarball and need to be compressed We use the second method with few changes .
    
- We can create a tar file by adding files one by one using a loop with an append option `-r`.

### zcat - reading gzipped files without extracting
`zcat` is a command that can be used to dump an extracted file from a __.gz__ file to stdout without manually extracting it .

### Compression ratio
- We can specify the compression ratio ,Which is available in range 1 to 9 ,Where
    - `1` is the lowest ,but fastest
    - `9` is the best ,but slowest

    You can specify any ratio in that range as follows :
    ``` bash 
    gzip -5 testing
    ```


## bzip2
`bzip2` offers more effective compression than `gzip` ,while taking more time than `gzip`.
- To compress a file using `bzip2` : `bzip2 filename`
- Extract a bzipped file as follows : `bunzip2 filename.bz2`
- Use `-j` by the tar command to denotes that the archive is bzip2 format .For example :
``` bash 
tar -xjvf archive.tar.bz2
```


## Using lazma
`lzma` is a compression tool which has ever better compression than gzip and bzip2
- To compress a file using laza : `lzma filename`
- Extract a lzma'd file as follows : `unlzma filename.lzma`

- A tarbal can be compressed by using the `--lzma` option passed to the tar command while archiving and executing .
``` bash 
tar -cvvf --lzma archive.tar.lzma [FILES]
```

## Archiving and compressing with zip
- In order to archive with zip
``` bash 
zip archive_name.zip [SOURCE FILES / DIRS ]
```
- Archive directories and files recursively as follows
``` bash 
zip -r archive.zip folder1 folder2
```
- In order to extract files and fo;ders in a zip file `unzip file.zip`
- In order to update files in the archive with newer files in the filesystem ,yse the `-u` flag
``` bash 
zip file.zip -u newfile
```
- Delete a file from a zipped archive ,by using `-d` as follows
``` bash 
zip -d src.zip file.txt
```

- In order to list the files in a archive use
``` bash 
unzip -l  archive.zip
```  
?> zip unlike lzma ,gzip ,or bzip2 won't remove the source file after archiving

## Faster archiving with pbzip2

`pbzip2` can use multiple cores ,hence decreasing overall time taken to compress your files 
- compress a single file like this `pbzip2 myfile.tar`
- To compress and archive multiple files or directories ,we use `pbzip2` in combination with tar as follows 
``` bash 
tar -cf myfile.tr.bz2 --use-compress-prog=pbzip2 dir_to_compress/
```
- Extracting a pbzip2'd file
``` bash 
pbzip2 -dc myfile.tar.bz2 | tar x
# or
pbzip2 -d myfile.tar.bz2
```
- Manually specifying the number of CPUs use the `-p` option to pbzip2 to specify the number of cpu cores manually .For example:
``` bash 
pbzip2 -p4 myfile.tar
```

- Just like other compression tools we saw up to now ,we can use the options from 1 to 9 to specify the fastest and best compression ratios respectively

## Creating filesystems with compression
`squashfs` can be useful when it is required to keep files heavily compressed and to access a few of them without extracting all the files .
- In order to create a squashfs file by adding source directories and file ,use
``` bash 
mksquashfs SOURCE compressedfs.squashfs
```
?> Source can be wildcards ,or files ,or folders paths.

- To mount the squashfs file to a mount point ,use loopback mounting as follows :
``` bash 
# mkdir /mnt/squash 
mount -o loop compressedfs.squashfs /mnt/squash
```
- Excluding files while creating a squashfs file 
To exclude a list of files specified as command-line arguments use the `-e` option.

For example :
``` bash 
sudo mksquashfs /etc test.squashfs -e /etc/passwd /etc/shadow
```

- It is also possible to specify a list of exclude files given in file with `-ef` as follows :
``` bash 
sudo mksquashfs /etc test.squashfs -ef excludelist
```

- If we want to support wildcards in excludes list ,use `-wildcard` as argument.

## Backup snapshot with rsync
- To copy a source directory to a destination use 
    ``` bash 
    rsync -av source_path destination_path
    ```
    - `-a` stands for archiving
    - `-v` (verbose) prints the details or progress on stdout

The above command will recursively copy all the files from the source path to the destination path .

- In order to backup data to a remote server or host
```bash
rsync -av source_dir user@host:PATH
```

- Restore the data from the remote host to localhost as follows
``` bash 
rsync -av username@host:PATH destination
```

- We can use the rsync option `-z` to specify to compress data while transferring through a network .For exmaple :
    ``` bash 
    rsync -avz source destination
    ```
- `/` not presented ,so rsync will copy that end directory itself to the destination
``` bash 
rsync -av /home/test /home/backups
```
- use `/` at the end of the source ,rsync will copy contens of that end directory specified in the source_path to the destination .For example :
``` bash 
rsync -av /home/test/ /home/backups
```

- Excluding files while archiving with rsync 
    It is possible to tell rsync to exclude certain files from the current operatin
    - `--exclude PATTERN`
    - or we can specify a list of files to be excluded by providing a list file. `--exclude-from Filepath`

- Deleting non-existing files while updating rsync backup
In order to remove the files from the destination that do not exist at the source ,use the rsync `--delete` option :
``` bash 
rsync -avz SOURCE Destination --delete
```

## Version control-based backup with git
- `git init --bare`
- Add user details to Git in the source host machine
``` bash 
git config --global user.name "Sarah Lakshman"
git config --global user.email "blah-blah@gmail.com"
```
- In the source direcotry in  the host machine whose files are to be back up:
``` bash 
git init
git commit --allow-empty -am "Init"
```

- In the source directory .execute the following command to add the remote git directory and rsync backup :
``` bash 
git remote add origin user@remotehost:/home/backups/backup.git
git push origin master
```

- Add or remove files for git tracking 
``` bash 
git add *
```

- We can conditionally add certain files only to the backup list as follows :
``` bash 
git add *.txt
git add *.py
```
- We can remove the files and folders not required to be tracked by using `git rm file`

- we can mark checkpoints for the backup with a message using the following command :
``` bash 
git commit -m "Commit Message"
```
- To view all backup versions 
``` bash
git log
```

- To revert back to any prev state or version ,use the commit ID with git checkout 
``` bash 
git checkout <COMMIT_ID>
```

- To make this revert permanent
``` bash 
git commit -am "Restore @$(data) commit ID:32Feba4e6c"
```
- We can create the contents from the backup at the remote location as follows :
``` bash 
git clone user@remotehost:/home/backup/backup.git
```

## Creating entire disk image using fsarchiver
- Creating a backup of a filesystem/partition use `savefs` option of `fsarchiver` like this :
``` bash 
fsarchiver savefs backup.fsa /dev/sda1
```

- We can pass multiple filesystems as source

- Restore a partition from a backup archive use the `restfs` option of `fsarchiver` like this 
    ``` bash 
    fsarchiver restfs backup.fsa id=0,dest=/dev/sda1
    ```
    - `id=0` denotes that we want to pick the first partition from the archive to the partition specified as `dest=/dev/sda1`

- To restore multiple partitions from a backup archive /use the `restfs` option as follow :
``` bash 
fsarchiver restfs backup.fsa id=0/dest=/dev/sda1 id=1,dest=/dev/sdb1
```
    ?> Here ,we use two sets of the id,dest param to tell fsarchiver to restore the first two partitions from the backup to the physical partiotions.

> - `/dev/sda1` 
>    - `/dev/` stands for device filesystem
>    - `sd` stands for sata disk 
>    - `a` stands for disk number
>    - `1` stands for partition number




    