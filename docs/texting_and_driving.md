## Using regular expressions 
Regular expressions are a form of tiny ,highly-specialized programming languages to match text.

Lets see a few example of text matching 
- To match all words in  agiven text ,we can write the regex as follows: ` ( ?[a-zA-Z]+ ?)
- `?` is the notation for zero of one occurrence of the previous expression ,which  in this case is the space char
- The `[a-zA-Z]+` notation represents one or more alphabet char(a-z and A-Z)

To match an ip address ,we can write the regex as follows :
```bash
[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}
```
- `\.` matches the dot char (.) .
- `{1-3}` matches one to three digits

- `^` -> This specifies the start of the line market
- `$` -> This specifies the end of the line marker
- `.` -> This matches any one character
- `[]` -> This matches any one of the char enclosed in `[chars]`
- `[^]` -> This matches any one of the char except those that are enclosed in `[^chars]`
- `[-]` -> This matches any char within the range specified in `[]`
- `?` -> This means that the preceding item must match one or zero times 
- `+` -> This means that the preceding item must match one or more times 
- `*` -> this means that the preceding item must match zero or more times
- `()` -> This treats the term enclosed as one entity
- `{n}` -> This means that the preceding item must match n times
- `{n,}` ->This specifies the minimum number of times the preceding item should match 
- `{n,m}` -> This specifies the min and max number of times the preceding item should match 
- `|` -> This specifies the alternation one of the items on rither side of `|` should match
- `\` -> This is esc char for escaping any of the special char mentioned previously

?> visualizing regualr expressions : `https://www:regexpr.com`


## Searching and mining a text inside of a file with grep
- To search for lines of text that contain the given pattern :
``` bash 
grep pattern filename
```
- We can also read from stdin

- Perform a search in multiple files by using a single grep invocation as follows : 
``` bash 
grep "match_text" file1 file2 file3
```

- We can highlight the word in the line by using --color option as follow
``` bash 
grep word filename --color=auto
```
- To use the full set of regular expressions as input arguments the `-E` option should be added ,Which means an extended regular expression

- In order to output only the matching portion of a text in a file ,use the `-o` option .
``` bash 
echo this is a line. | egrep -o "[a-z]+\."
line. #output
```
- In order to print all of the lines ,except the line containing match_pattern ,use:
``` bash 
grep -v match_pattern file
```
?> `-v` suppose as invert

- count the number of lines in which a matching string or regex match appear in a file or text ,as follows :
``` bash 
grep -c "text" filename
```
!> count on the number of matching line ,not the number of times a match is made

- To count the number of matching item in a file ,use the following trick 
``` bash 
echo -e"1 2 3 4\nhello\n5 6" | egrep -o "[0-9]" | wc -l
```
- Print the line number of the match string as follow `grep linux -n sample1.txt`

- Print the char or byte offset at which a pattern matches ,as follows :
``` bash 
echo gnu is not unix | grep -b -o "not" 
# output
7:not  #(a counter form zero)
```

- To search over multiple files ,and list which files contain the pattern ,we use the following :
``` bash 
grep -l linux sample1.txt sample2.txt
```
?> The inverse of the `-l` argument is `-L`. The `-L` argument returns a list of non-matching files.

- Recursively search many files 
`grep "text" . -R -n`
> The option `-R` and `-r` mean the same thing when used with grep 

- The `-i` argument helps match patterns to be evaluated ,without considering uppercase or lowercase .For example : `echo hello world | grep -i "HELLO"`

- grep by matching multiple pattern 
we can use an argument `-e` to specify multiple patterns for matching ,as follows 
``` bash 
grep -e "pattern1" -e "pattern"
```
- We can use a pattern file for reading patterns.
write pattern to match line-by-line ,and execute grep with a `-f` argument as follows 
``` bash 
grep -f pattern_filesource
```

- Including and excluding files in a grep search we can specify includes files or exclude files by using wild card pattern
``` bash 
grep "main()" . -r --include *.{c,cpp}
# Exclude all Readme files in the search ,as follows:
grep "main()" . -r --exclude "README"
```

- To exclude directories ,use the `--exclude-dir` option.

- To read a list of files to exclude from a file ,use `--exclude-from File`

### Using grep with xargs with zero-byte suffix:
``` bash 
grep "test" file* -lZ | xargs -0 rm
```
grep outputs filenames with a zero-byte terminator (\0) ,because of the `-Z` option .
> Usually ,`-Z` is used along with `-l`

- Silent output to grep 
We can use the quiet option (`-q`) , where the grep command does not write any output to the standard output .It runs command and return an exit status based on success or failure .

- In order to print three lines after a match ,use the `-A` option `seq 10 | grep 5 -A 3`

- In order to print three lines before the match ,use the `-B` option `seq 10 | grep 5 -B 3`

- print three lines after and before the match ,and use the `-c` option as follows : `seq 10 | grep 5 -c 3`

?> if there are multiple matches ,then each section is delimited by a line "--"

## Cutting a file column-wise with cut
? __Tab__ is the default delimiter for fields or columns .
- To extract particular fields or columns ,use the following syntax `cut -f Field_List filename`
Field_list is a list of columns that are to be displayed . the list consists of column number delimited by commas . 
- `cut` can also read input from stdin 
- To avoid printing lines that do not have delimiter characters ,attach the `-s` option along with cut.
- We can also complement the extracted fields by using `--complement` options .For example to print all the columns except the third column ,then use the following command `cut -f3 --complement file_name.txt`
- To specify the delimiter char for the field ,use the `-d` option as follows :
``` bash 
cut -f -d ";" delimited_data.txt
```
- Specifying the range of characters or bytes as fields suppose that we don't rely on delimiters ,but we need to extract firlds in such a way that we need to define a range of chars .
    - `N-` from the Nth byte ,characters ,or filed ,to the end of the line
    - `N-M` from the Nth to the Mth(included) byte ,char or field
    - `-M` from the first to the Mth(included) byte ,char ,or field 

    We use preceding notation to specify fields as a range of bytes or char with the following options 
    - `-b` for byte
    - `-c` for char
    - `-f` for defining fields

    ?>  We can specify the output delimiter while using with `-c` , `-f` ,as follows : `--output-delimiter "delimiter string"`

## Using sed to perform text replacement
- It can be matched using regular expressions .
``` bash 
sed 's/pattern/replace_string/' file
```

?> command to replace text in vi editor is very similar to the one discussed here

- By default ,sed only prints the submited text .To save the changes along with the subtitutions to the same file ,use `-i` option .for example `sed -i 's/text/replace/' file`

- These usage of sed command will replace the first occurrence of the pattern in each line . if we want to replace every occurrence ,we need to add the `g` param at the end as follows `sed 's/pattern/replace_string/g' file`

- We sometimes need to replace only the `Nth` occurrence onwards .For this ,we can use the `/Ng` form of the option

- We have used `/` in sed as a delimiter char. We can use any delimiter char as follows : `sed 's:text:replace:g'`

?> when the delimiter char appears inside the pattern ,We have to escape it using the `\` prefix 

- Removing blank line
Blanks can be matched with regular expressions `^$` 
``` bash
sed '/^$/d' file
```
`/pattern/d` will remove lines matching the pattern

- Create newfile contains stdout 
    You can use the following form of sed
    ``` bash 
    sed -i .bak 's/\b[0-9]\{3\}\b/NUMBER/g' sed-data.txt
    ```
    - `\b[0-9]\{3}\b` is the regular expression to match three digit number
    - `\` in `\{3\}` is used to give a special meaning for { and } .
    - `\b` is the word boundary marker

- Matched string notation (&)
in sed we can use & as the matched string for the subtituation pattern . For example :
``` bash 
echo This is as example | sed 's/\w\+/[&]/g'
```
Here regex `\w\+` matches every word

- Substring match notation (\1)
    We can also match the substring of the given pattern through `\1` as follows
    ``` bash 
    echo this is digit 7 in a number | sed 's/digit \([0-9]\)/\1/'
    ```
    The preceding command replace __digit 7__ with __7__.

    - `\(pattern\)` is used to match the substring .
    - `\1` is the corresponding notation for the first substring match .
    ?> for the second ,it is `\2` for example :
    ``` bash 
    echo seven EIGHT | sed 's/\([a-z]\+\) \([A-Z]\+\)/\2 \1/'
    #outout
    EIGHT seven
    ```

    - `([a-z]\+\)` matches the first word ,and `\([A-Z]\+\)` matches the second word . `\1` and `\2` are used for refering them ,this type of refrencing is called __back refrencing__.


- Combination of multiple expressions 
The combination of multiple sed using a pipe
``` bash 
sed 'expresseion' | sed 'expression'
#or
sed 'expression;expression'
#or 
sed -e 'expression' -e 'expression'
```

- Usually ,it is seen that the sed expression is quoted by using single quotes .
?> Using double quotes are useful when we want to use a variable string in a sed expression


## Using awk for advance text processing
The structure of an awk script is as follows :
``` bash 
awk ' BEGIN{ print "start" } pattern { command }
END { print "end" }' file
```
- The `awk` command can read from stdin also 
- An awk script usually consist of three parts
    - BEGIN
    - END
    - a common statements block with the pattern match option
    
    ?> The three of them are optional and any of them can be absent 

- How it works ? 
    - Execute the statement in the `BEGIN { commands }` block .
    - Read on line from the file or stdin and execute `pattern { commands }` .Repeat this step until the end of the file is reached.
    - When the end of the input stream is reached ,execute the `END { commands }` block.

    ?> Usually we place initial variable assignements such as `var=0;` and like statements ,print the file header in the `BEGIN` block .In the `END {}` block ,we place statement such as printing results


- Some special variables that can be used with `awk` are as follows
    - `NR` -> Itstands for the current record number ,Which corresponds to the current line number when it uses lines as records
    - `NF` -> It stands for the number of fields ,and corresponds to the number of field in the current record under execution (fileds are delimited by space)
    - `$0` -> It is a variable that contains the text content of the current line under execution
    - `$1` -> It is a variable that holds the text of the first field
    - `$2` -> It is the variable that holds the text of the second field

- We can print the last field of a line as print `$NF` ,last but the second as `$(NF-1)` ,and so on. `awk` also provides the `printf()` function with the same syntax as in C
    - print the second and third field of ecery line as follows `awk '{ print $3,$2 }' file`
    - In order to count the number of lines in a file `awk'END { print NR }' file`

- Passing an external variable to awk 
By using the `-v` argument ,we can pass external values other than stdin as follows :
``` bash 
VAR=1000
echo | awk -v VARIABLE=$VAR '{ print VARIABLE }'
```

- There is a flexible alternate method to pass many variable values from outside awk.
``` bash 
echo | awk '{ print v1,v2 }' v1=$var1 v2=$var2
```

- When an input is given through a file rather than standard input ,use the following command 
``` bash 
awk '{ print v1,v2 }' v1=$var1 v2=$var2 filename
```
- Reading a line explicity using getline 
    if you want to read a specific line ,you can use the `getline` function .Sometimes ,you may need to read the first line from the `BEGIN` block
    - The syntax is `getline var` .The variable var will contain the content for the line .
    - If `getline` is called without an argument ,we can access the content of the line by using $0,$1, and $2.

- Filtering line processed by awk with filter patterns
    - `awk 'NR < 5'` -> first four lines
    - `awk 'NR==1 , NR==4'` -> first four lines
    - `awk '/linux/'` -> lines containing the pattern linux (we can specify regex)
    - `awk '!/linux/'` -> lines not contaning the pattern linux 

- Setting delimiter for fields 
By default delimiter four field is a space we can explicity specify a delimiter by using `-F "delimiter"`

``` bash 
awk -F: '{ print $NF }' /etc/passwd
#or
awk 'BEGIN { FS=":" } { print $NF }' /etc/passwd
```
?> We can set output fileds seprator by setting `OFS="delimiter"` in the `BEGIN` blok.

- Reading the command output from awk 
the syntax for reading out the commands in a variable is as follows `"command" | getline out;`

For example : 
``` bash 
echo | awk '{"grep root /etc/passwd" | getline cmdout ; print cmdout }'
```


- A for loop is available in awk . It has the following format `for(i=0;i<10;i++) { print $i; }` or `for(i in array) { print array[i]; }`
?> awk supports associative arrays ,which can use the text as the index 

- Starting manipulation function in awk 
    - `lenght(string)` -> This return the string length.
    - `index(string , search , string)` -> This return the position at which search_string is found in the string
    - `split(string , array , delimiter)` -> This stores the list of strings generated bu using the delimiter in the array
    - `substr(string , start-position , endposition)` -> This return the substring created from the string by using the start and end character offsets
    - `sub(regex , replacement_str , string)` -> This replace the first occuring regular expression match from the __string__ with __replacement_str__
    - `gsub(regex , replacement_str , string)` -> This is similar to `sub()` ,but it replace every regular expression match.
    - `match(regex , string)` -> This return the result of wheter regular expression match is found in string or not .It returns a non-zero output if a match is found ,otherwise it return zero. Two special variables are assocciated with `match()`. They are `RSTART` and `RLENGTH`
        - `RSTART` -> contains the position at which the regular expression match start
        - `RLENGTH` -> contains the length of the string matched by regular expression

## Finding the frequency of words used in a given file 
``` bash 
#!/bin/bash
#Name: word_freq.sh
#Desc: Find out frequency of words in a file
if [ $# -ne 1 ];
then
    echo "Usage: $0 filename";
    exit -1
fi

filename=$1

egrep -o "\b[[:alpha:]]+\b" $filename | \

awk '{ count[$0]++ }
END{ printf("%-14s%s\n","Word","Count") ;
for(ind in count)
{ printf("%-14s%d\n",ind,count[ind]); }
}'
```

- `egrep -o "\b[[:alpha:]]+\b"` $file is used to output only words . the `-o` option will print the matching char sequence delimited by a newline char .Hence ,we receive words in each line
- Since awk ,by default ,executes the statements in the `{}` block for each row ,we don't need a specific loop for doing that . Hence ,the count is incremeneted as `count[$0]++` by using the associative array
- Finally ,in the `END{}` block ,we print the words and their count by iteration through the words.



## Compresssing or decompressing javascript
Following are the tasks that we need to perform for compressing js :
- Replace newline and tab char 
- Squeeze spaces
- Replace comment that look like /* content */

```bash
# Remove the \n and \t char
tr -d '\n\t'
# Remove extra spaces
tr -s ' ' 
# or
sed 's/[ ]\+/ /g'
# remove comments
sed 's:/\*.*\*/::g' #       .* is used  to match all text in between /* and */
# remove all the spaces preceding and suffixing the {,},(,),;,: and , .
sed 's/ \?\([{}();,:]\) \?/\1/g'
```

To decompress ,we can use the following tasks 
- Replace `;` with `;\n`
- Replace `{` with `{\n` ,and `}` with `\n}`

## Merging multiple files as columns
`paste` is the command that can be used for column-wise concatenation
- the default delimiter is tab . We can also explicitly specify the delimiter by using `-d`.

## printing text between line numbers or pattern
- `awk 'NR==M , NR==N' filename`
- To print the lines of a text in a section with start_pattern and end_pattern ,use the following syntax
``` bash 
awk '/start_pattern/, /end_pattern/' filename
```

## printing lines in reverse order
There is a direct command ,`tac`, to do same as follows :
``` bash 
tac file1.txt file2.txt
```
- In `tac` , `\n` is the line separator .But we can also specify our own seprator by using the `-s` "separator" option.

?> `\` in the shell script is used to conveniently break a single line command sequence into multiple lines

## parsing e-mail addresses and URLs from text.

- the `egrep` regex pattern for an email `[A-Za-z0-9._]+@[A-Za-z0-9.]+\.[a-zA-Z]{2,4}`

- the `egrep` regex pattern for an HTTP url `http://[a-zA-Z0-9\-\.]+\.[a-zA-Z]{2,4}`

## Removing a sentence in a file containing a word 
``` bash 
sed 's/ [^.]*mobile phones[^.]*\.//g' sentence.txt
```
> This receipe assumes that no sentence spans more than one line.

- every match sentence is replaced by `//` (nothing)

## Replacing a pattern with text in all the files in a direcotry 
Let's say we want to replace the text __Copyright__ with the word __Copyleft__ in all .cpp files:
``` bash 
find . -name *.cpp -print0 | xargs -I{} -0
sed -i 's/Copyright/Copyleft/g' {}
# or
find . -name *.cpp -exec sed -i 's/Copyright/Copyleft/g' \{\} \;
# or 
find . -name *.cpp -exec sed -i 's/Copyright/Copyleft/g' \{\} \+
```
?> While they perform the same function ,then second will call sed once for every file that is found ,while in third form ,find will combine multiple filenames and pass them together to sed .

## Text slicing and parameter operation 
- replacing some text from a variable 
``` bash 
echo ${var/line/REAPLACED}
```
`line` is replaced with `REPLACED`

- We can produce a substring by specifying the start position and string length 
``` bash
${variable-name:start_position:length}
```
- To print from the fifth char onwards `echo ${string:4}`
- The index is specified by counting the start letter as 0
- We can also specify counting from the last letter as `-1` it is used inside a paranthesis. `(-1)` is the index for the last letter