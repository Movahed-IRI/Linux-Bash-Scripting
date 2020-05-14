## Setting up the network 
- list the current network interface configuration `ifconfig`
The left-most column in the i`ifconfig` output lists the names of network interfaces.

- In order to manually set the ip address for a network interface ,use `ifconfig wlan0 192.168.0.80`

- Set the subnet mask along with the IP address as follows
``` bash 
ifconfig wlan0 192.168.0.80 netmask 255.255.255.0
```

- Automatically configure network interface `dhclient eth0`

> Exercise ! : Print the list of network interface

### Dispaying ip address
We can restrict ifconfig to a specific interface using `ifconfig iface_name`

- ?> From the output of the previously mentioned command out interest lie in the list below
    - `HWaddr` -> is the hardware address (MAC)
    - `inet addr` -> is the ip address
    - `Bcast` -> is the broadcast address
    - `Mask` -> is the subnet mask

- In order to extract the IP address from the ifconfig output use 
``` bash
ifconfig wlan0 | egrep -o "inet [^ ]*" | grep -o "[0-9.]*"
```

### Spoofing the hardware address(MAC address)
We can spoof the hardware address at the software level as follows 
``` bash 
ifconfig eth0 hw ether 00:1c:bf:87:25:d5
```

## Nameserver and DNS(Domain name service)
- Name servers asssigned to the current system can be viewed by reading `/etc/resolv.cong`

- We can add name servers manually as follows : `echo nameserver IP_ADDRESS >> /etc/resolv.conf`

## DNS lookup
When `host` is executed it will list out all of the IP address attached to the domain name . `nslookup` is another command that is similar to `host`

``` bash 
host google.com
nslookup google.com
```

?> Without using the DNS server ,it is possible to add a symbolic name to the ip address resolution just by adding entries into the file `/etc/hosts`

## Showing routing table information
- The routing table can be displayed as follows `route` or `route -n`
    - using `-n` specifies to display the numeric address

- A default gateway is set as follows :
``` bash 
route add default gw IP_ADDRESS Interface_name
```

## Let us ping !
In order to check the community of two hosts on a network ,the `ping` command uses __Internet Control Message Protocol__(ICMP) echo packets .
- Check wheter a host is reachable as follows `ping ADDRESS`

?> Network administrator generally configure devices such as router not to respond to ping .This is done to lower security risks ,as ping can used by attackers (Using brute-force) to find out IP ADDRESS ot machine .

- Limiting The number of packets to be sent 
We can limit the count of echo packets to ,be sent by using the `-c` flag as follows
``` bash 
ping 192.168.0.1 -c 2
```

- Return status of the ping command
The ping command return exit status 0 when it succeeds and return non-zero when it fails.

### Round Trop Time
The ping command can be used to find out the Round Trip Time(RTT) between two hosts on a network .RTT is the time required for the packet to reach the destination host and comeback to source host.
    - `RTT mdev` parameter in the ping output stands for mean deviation


## traceroute
There an interesting command `traceroute` that displays the address of all intermediate gateway through which a packet travelled to reach a particular destination . 

`traceroute` information helps us to understand how many hops each packets packets should take in order to reach the distination . An example of `traceroute` is as follows:
``` bash 
traceroute google.com
```
?> Modern Linux distribution also ship with a command `mtr` ,which is similar to `traceroute` but shows real-time data which keeps refreshing.

## Listing all the machine alive on a network
We can write our own script using the `ping` command to query a list of IP ADDRESS and check whether they are alive or not.
- Using fping
    ``` bash 
    fping -a 192.160.1/24 -g 2> /dev/null
    # or
    fping -a 192.168.0.1 192.168.0.255 -g 
    ```
    - `-a` stands for alive machine 
    - `-g` specifies to generate a range of ip address .If `-g` not specified so we can use stdin

- Parallel pings 
    - To make the `ping` commands run in parallel .We enclose the loop body in `()&`
    - `()` encloses a block of commands to run as a subshell and `&` sends it to the background
    - place `wait` at the end of the script ,so that it waits for the time untill all the child() subshell process complete

## Running commands on a remote host with SSH
SSH stands for secure shell as it transfer the network data transfer over an encrypted tunnel
- To connect to remote host with the SSH server running `ssh username@remote-host`

- In order to connect to an SSH server runnig at port 422 ,use `ssh user@localhost -p 422`

- To run command at the remote host and displat its output on the local shell `ssh user@host 'COMMANDS'`
?> We can also pass a more complex subshell in the command sequence by using the `()` subshell operator.

- SSH with compression
use `-c` option with the ssh command to enable compression

- Readirecting data into stdin of remote host shell commands `ssh user@host 'echo' < file`
> echo on the remote host prints the data recieved through stdin which in turn is passed to stdin from localhost

- Reading graphical commands on a remote machine 
    - This will launch the graphical output on the remote machine 
    ``` bash 
    ssh user@host "export DISPLAY=:0; command1;"
    ```
    - This will run the commands on the remote machine ,but it will bring the graphical out to your machine
    ``` bash 
    ssh -X user@host "command1; command2"
    ```

## Transferring files through the network
This receipe discuss how to transfer files using commonly used protocols FTP , SFTP , RSYNC ,and SCP.

- Using `lftp` command -> Files can be transfered via FTP
- Using `sftp` command -> Files can be transfered via SSH connection
Rsync and scp are over SSH to.

### FTP 
- To connect to an ftp server and transfer files in between use `lftp username@ftphost`
    after authentication you can type commands in this prompt . For example
    - To change to a direcotory ,use `cd direcotory`
    - To change to a directory of a local machine ,use `lcd direcotry`
    - To create the directory use `mkdir`
    - To list files in the current directory on the remote machine use `ls`
    - To download a file ,use `get <filename>`
    - To upload a file from the current dirctory ,use `put filename`
    - An ftp session can be terminated by using the `quit` command

    ?> Autocompletion is supported by the lftp prompt

#### Automated FTP transfer 
`ftp` is another command used for FTP-based file transfer . `lftp` is more flexible for usage.

> Exercise ! What if we want to automate file transfer instead of interactive mode ?

``` bash 
#!/bin/bash
#Filename: ftp.sh
#Automated FTP transfer
HOST='domain.com'
USER='foo'
PASSWD='password'
ftp -i -n $HOST <<EOF
user ${USER} ${PASSWD}
binary
cd /home/slynux
puttestfile.jpg
getserverfile.jpg
quit
EOF
```
- The `-i` option of ftp turns off the interactive session with user
- `binary` sets the file mode to binary
- The `-n` option tells ftp to not attemp automatically logging in and use the username and password we supply it

### SFTP 
- In order to run sftp ,use `sftp user@domainname`
- The following commands are used to perform the file transfer
``` bash 
cd /home/mov
put testfile.jpg
get serverfile.jpg
```
- Session can be terminated by typing the `quit` command.
- Sometimes ,the SSH server will not be runnig at the default port 22 .we can specify the port along with sftp as `-oport=PORTNO`:
``` bash 
sftp -oport=422 user@movah.com
```

### SCP (Secure copy program)
- We can easily transfer files to a remote machine as follow : `scp filename user@remotehost:/home/path`
- Source or destination can be in the format `username@host:/path`
- If SSH is running at a different port than 22 ,use `-oport` with same syntax ,as sftp
- We can recursively copy a directory between two machnes on a network by using `-r` param.
- scp can also copy files by preserving permissions and modes by using the `-p` param.

## Connecting to a wireless network
For a wireless network connection ,it will require additional utilities such as `iwconfig` and `iwlist` to config more param
- We can scan and list the available wireless network using the utility `iwlist` : `iwlist scan`
- Write a script for connecting to a wireless LAN with WEP 
``` bash 
#!/bin/bash
#Filename: wlan_connect.sh
#Description: Connect to Wireless LAN
#Modify the parameters below according to your settings
######### PARAMETERS ###########

IFACE=wlan0
IP_ADDR=192.168.1.5
SUBNET_MASK=255.255.255.0
GW=192.168.1.1
HW_ADDR='00:1c:bf:87:25:d2'
#Comment above line if you don't want to spoof mac address

ESSID="homenet"
WEP_KEY=8b140b20e7
FREQ=2.462G
#################################

KEY_PART=""
if [[ -n $WEP_KEY ]];
then
    KEY_PART="key $WEP_KEY"
fi

/sbin/ifconfig $IFACE down # Turn the interface down before setting new config

if [ $UID -ne 0 ];
then
    echo "Run as root"
    exit 1;
fi
if [[ -n $HW_ADDR ]];
then
    /sbin/ifconfig $IFACE hw ether $HW_ADDR
    echo Spoofed MAC ADDRESS to $HW_ADDR
fi

/sbin/iwconfig $IFACE essid $ESSID $KEY_PART freq $FREQ

/sbin/ifconfig $IFACE $IP_ADDR netmask $SUBNET_MASK

route add default gw $GW $IFACE

echo Successfully configured $IFACE
```
?> WEP is used in this example for simplicity ,but its worthy to note that currently it is considered insecure .Use a cariant of Wi-Fi protected Access2(WPA2) to be secure

## Password-less auto-login with ssh
SSH uses an encryption technique called asymmetric keys consisting of two keys for automatic authentication :
- A public key
- A private key 

There are two steps toward setup of automatic authenctication 
- Creating the SSH key on the machine ,which requires a login to a remote machine `ssh-keygen -t rsa`
?> Now `~/.ssh/id_rsa.pub` and `~/.ssh/id_rsa` have been generated .id_rsa.pub is the generated public key and id_rsa is the private key

- Transferring the public key generated to the remote host and appending it to authorized_key as follows :
``` bash 
ssh user@remote_host "cat >> ~/.ssh/authorized_keys" < ~/.ssh/id_rsa.pub
```
Most Linux distros ship with a tool called `ssh-copy-id` which will automatically add your private key to the authorized_keys file on the remote server .Use it like this : 
``` bash 
ssh-copy-id USER@REMOTE_HOST
```

## Port forwarding using SSH
- Use this command to forward a port 8000 on your local machine to port 80 of www.kernel.org as follows : 
``` bash 
ssh -L 8000:www.kernel.org:80 user@localhost
```
now we can access linux kernel website through port 80 of local machine

- You can even use this command to forward a port 8000 on a remote machine to port 80 of www.kernel.org
``` bash 
ssl -L 8000:www.kernel.org:80 user@REMOTE_MACHINE
```
 
### non-interactive port forward
```bash 
ssh -fL 8000:www.kernel.org:80 user@localhost -N
```
- The `-f` instructs ssh to fork to background just before executing the command
- `-L` for the login name for the remote host machine
- `-N` tells ssh that there is no command to run

## Reverse Port Forwarding
reverse port forwarding is very similar to port forwarding
``` bash 
ssh -R 8000:localhost:80 user@REMOTE_MACHINE
```
This will forward port 8000 on the remote machine to port 80 on the local machine

## Mounting a remote device at a local mount point 
`sshfs` is an extension to the __FUSE__ filesystem package that allows supported OSs to mount a wide variety of data as if it were a local filesystem.
- In order to mount a filesystem location at a remote host to a local mount point ,use :
``` bash 
sshfs -o allow_other user@remotehost:/home/path /mnt/mountpoint
```
- In order to unmount after completing the work ,use :
``` bash 
unmount /mnt/mountpoint
```

## Network traffic and port analysis
- In order to list all opened ports on the system along with the details on each service attached to it ,through 
``` bash 
lsof -i
```
?> The last column of output consists of lines similar to `laptop.local:41197->192.168.0.2:3128` .
In this output , `laptop.local:41497` corresponds to the remote host .
In order to list out the opened ports from the current machine ,use :
``` bash 
lsof -i | grep ":[0-9]\+->" -o | grep "[0-9]\+" -o | sort | uniq
```

- Use `netstat -tnp` to list opened port and services.

## Creating arbitary sockets
- Setup the listening socket using the following `nc -l 1234`
- Connect to the socket using special ports as follows `nc HOST PORT`
- Exploit netcat and shell redirection to easily copy files :
``` bash 
# On the receiver machine ,run the following command
nc -l 1234 > destination_filename
# on the sender machine ,run the following command :
nc HOST 1234 < source_filename
```

## Sharing an internet connection 
This receipe use `iptables` for setting up Network Address Transaltion(NAT) which lets a network device share a connection with other devices .

> In the receipe ,We are assuming that the primary wired network connection ,eth0 is connected to internet

> - Using your distro's network management tool ,create a new adhoc wireless connection with the following settings
    - IP address : 10.99.66.55
    - Subnet Mask : 255.255.0.0(16)

- use the following shell script to share the internet connection :
``` bash 
#!/bin/bash
#filename: netsharing.sh
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -A FORWARD -i $1 -o $2 -s 10.99.0.0/16 -m conntrack
--ctstate NEW -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j
ACCEPT
iptables -A POSTROUTING -t nat -j MASQUERADE
```

- Run the script as follows : `./netsharing.sh eth0 wlan0

- Connect yout device to the wireless network you just created with the following settings 
> IP address : 10.99.66.56
> Subnet mask : 255.255.0.0

?> To make this more convenient ,you might want to install a DHCP and DNS server on your machine .A handy tool for this is `dnsmasq` which you can use for performing both DHCP and DNS operations.

## Basic firewall using iptables 

- Block traffic to a specific IP address 
    ``` bash 
    iptables -A OUTPUT -d 8.8.8.8 -j DROP
    ```
    - The first argument in iptables is `-A` which instructs iptables to append a new rule to the chain specified as the next param.
    - A chain is simply a collection of rules ,and in this recipe we have used the __OUTPUT__ chain which runs on all outgoing traffic 
    - The `-d` param specifies the destination to match with the packet being sent .
    - We use the param `-j` to instruct iptables to __DROP__ the packet.
- Block traffic to a specific port 
    ``` bash 
    iptables -A OUTPUT -p tcp -dport 21 -j DROP
    ```
    - We use `-p` param to specify that this rule should match only __TCP__ on the port specified with `-dport`

- While playing with iptables  commands ,you might want to clear the changes made to the iptables chain .To do this ,just use `iptables --flush`