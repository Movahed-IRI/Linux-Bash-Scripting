## Downloading from a web page
- A webpage or remote file can be downloaded by using `wget` as follows
``` bash 
wget URL
```
- We can specify the output filename with the `-O` option.

- You can also specify a different logfile path rather than printing logs to stdout , by using `-o` option

- we can use the number of tries ,use the `-t` flag as follows `wget -t 5 URL`
?> To ask wget to keep trying infinitely `wget -t O URL`
- We can restrict the speed limits in wget by using the `--limit-rate` argument as follows 
``` bash 
wget --limit-rate 20k URL
```
You can also use `m` for megabyte

- We can also specify the maximum quota for the speed limit 
use `--quota` or  `-Q` as follows
``` bash 
wget --quota 100m URL1 URL2
```
- We can resume the download from where we left off by using the `-c` option as follows :
``` bash 
wget -c URL
```

- Copying a complete website (mirroring)
    In order to download the pages ,use `--mirror` options :
    ``` bash 
    wget --mirror --convert-links exampledomain.com
    # or
    wget -r -N -l -k DEPTH URL
    ```
    - `-l` specifies the depth of webpages .It is used along with `-r`(recusrive).
    - The `-N` argument is used to enable time stamping .
    - The `-k` or `--convert-links` option instructs wget to convert the links to other pages in a downloaded page ,to the local copy of those pages
    - Sometimes pages require authentication .By using the `--user` and `--password` .It is also possible to ask for a password without specifying the password inline .For this ,use `--ask-password` instead of the `--password` argument


## Downloading a webpage as plain text
`lynx` is an interesting command-line web browser ,which can get the web page as plain text .
- Let's download the webpage view ,in ASCII char representation ,in a text file by using the `-dump` flag with the lynx command:
``` bash 
lynx URL -dump > webpage_as_text.txt
```
- This command will also list all the hyperlinks seprately under a heading refrences ,as the footer of the text output.

## A primier on cURL
cURL is very useful when we play around with automating a web page usage sequnce ,and to retrieve data .

- To dump the downloaded file onto the terminal : `curl URL`
- To prevent the curl command from displaying the progress information , mention `--silent` .
- To write the downloaded data into a file with the filename parsed from the url rather than writing into the standard out use the `-O` option . `curl URL --silent -O`

- To show the # progress bar while downloading use `--progress` instead of `--silent`

- If the filenames are not present in the URL ,it will produce an error ,Hence ,we can manually specify the filename as follows ,by using the `-o` option `curl URL --silent -o new-filename`

- `curl` has advanced resume download features to continue at a given offset `curl URL/File -C offset`
?> curl doesn't require us to know the exact byte oddset. `curl -C - URL`

- you can use `--referer` with the curl command to specify the referer string as follows : `curl --referer Referer_URL target_URL`

- By using curl we can specify ,as well as store ,the cookie that are encountered during HTTP operation .To specify cookies ,use the `--cookie "COOKIES"` option.
?> Cookies should be provided as `name=value` .Multiple cookie should be delimited by a semicolon(;).

- To specify a file to which the cookies encountered are to be stored ,use the `--cookie-jar` option . `curl URL --cookie-jar COOKIE_FILE`

- using Curl user agent string can be set using `-user-agent` or `-A` as follows :
``` bash 
curl URL --user-agent "Mozila/5.0"
```

- Use `-H` "Header" to pass multiple additional headers. For example : 
``` bash 
curl -H "Host:www.google.com" -H "Accept-language: en" Url
```

> http://www.useragentstring.com/pages/useragentstring.php

- We can limit the download rate to a specifix limit in curl by using the `--limit-rate` option.
    - `k` -> kilobyte
    - `m` -> megabyte

- The maximum download file size for cURL can be specified by using the `-max-filesize` option as follows `curl URL --max-filesize bytes`
?> it will return a non-zero exit code if the filesize exceeds. It will return zero if it secceeds.

- The username and password can be specified by using `-u username:password` if you prefer to be prompted for the password ,you can do that by using only `-u username`

- printing response headers excluding the data
    - We can check the  __content-length__ parameter in the HTTP header to find out the length of a file before downloading . 
    - The __Last_Modified__ parameter enables us to know the last modification time for the remote file . 
    ?> use the `-I` or `-head` option with curl to dump only the HTTP headers.

## Accessing Gmail e-mails from the command line
``` bash 
#!/bin/bash
#Desc: Fetch gmail tool

username='PUT_USERNAME_HERE'
password='PUT_PASSWORD_HERE'

SHOW_COUNT=5 # No of recent unread mails to be shown
echo

curl -u $username:$password --silent "https://mail.google.com/mail/
feed/atom" | \
tr -d '\n' | sed 's:</entry>:\n:g' |\
    sed -n 's/.*<title>\(.*\)<\/title.*<author><name>\([^<]*\)<\/
name><email>
\([^<]*\).*/From: \2 [\3] \nSubject: \1\n/p' | \
head -n $(( $SHOW_COUNT * 3 ))
```
?> you will have to generate a new key to bypass two factor authentication

## Image crawler and downloader
``` bash 
#!/bin/bash
#Desc: Images downloader
#Filename: img_downloader.sh

if [ $# -ne 3 ];
then
    echo "Usage: $0 URL -d DIRECTORY"
    exit -1 
fi

for i in {1..4}
do
    case $1 in
    -d) shift; directory=$1; shift ;;
    *) url=${url:-$1}; shift;;
    esac
done

mkdir -p $directory;
baseurl=$(echo $url | egrep -o "https?://[a-z.]+")

echo Downloading $url
curl -s $url | egrep -o "<img src=[^>]*>" |
sed 's/<img src=\"\([^"]*\).*/\1/g' > /tmp/$$.list

sed -i "s|^/|$baseurl/|" /tmp/$$.list

cd $directory;

while read filename;
do
    echo Downloading $filename
    curl -s -O "$filename" --silent
done < /tmp/$$.list
```

## Finding broken links in a website 
``` bash 
#!/bin/bash
#Filename: find_broken.sh
#Desc: Find broken links in a website

if [ $# -ne 1 ];
then
    echo -e "$Usage: $0 URL\n"
    exit 1;
fi

echo Broken links:

mkdir /tmp/$$.lynx
cd /tmp/$$.lynx

lynx -traversal $1 > /dev/null
count=0;

sort -u reject.dat > links.txt

while read link;
do
    output=`curl -I $link -s | grep "HTTP/.*OK"`;
    if [[ -z $output ]];
    then
        echo $link;
        let count++
    fi
done < links.txt

[ $count -eq 0 ] && echo No broken links found.
```

## Tracking chnages to a website 
``` bash 
#!/bin/bash
#Filename: change_track.sh
#Desc: Script to track changes to webpage

if [ $# -ne 1 ];
then
    echo -e "$Usage: $0 URL\n"
    exit 1;
fi

first_time=0
# Not first time

if [ ! -e "last.html" ];
then
    first_time=1
    # Set it is first time run
fi

curl --silent $1 -o recent.html

if [ $first_time -ne 1 ];
then
    changes=$(diff -u last.html recent.html)
    if [ -n "$changes" ];
    then
        echo -e "Changes:\n"
        echo "$changes"
    else
        echo -e "\nWebsite has no changes"
    fi
else
    echo "[First run] Archiving.."
fi

cp recent.html last.html
```

## Posting to a webpage and reading the response 
- By curl 
``` bash 
curl URL -d "postvar=postdata1&postvar2=postdata2" 
# -d used for posting
```

- By wget `--post-data "string"`
``` bash 
wget url --post-data "host=test_host&user=movahed" -O output.html
```

?> the string to the post argument (to `-d` or `--post-data`) should always be given in quotes .If quotes are not used ,`&` is intrepted by the shell to indicate that this should be a background process.