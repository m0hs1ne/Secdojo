## Second Machine

We started like always scanning the IP using `nmap`, we see that port `445` is open so we can look for shares in `SMB`.
using `smbclient` to list all the available shares:
smbclient -L [ip]
we got this:

```Sharename Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        Backup          Disk
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
```

as you see we can access `Backup`.
we managed to access Backup using:
smbclient \\\\[IP]\\Backup
and we found out three files:
![](https://i.ibb.co/FHR0CxL/Screen-Shot-2022-12-29-at-9-59-47-AM.png)
we got them using `get`.
and those are obviously a `Windows registry file`, so i searched about a way to read them or access the machine using them.
The three files are likely to be backup copies of three of the hives in the Registry:

- "sam.save" is likely to be a backup copy of the "SAM" hive, which stands for Security Account Manager. The SAM hive stores information about user accounts and security policies.
- "security.save" is likely to be a backup copy of the "SECURITY" hive, which stores information about security policies and access control lists (ACLs).
- "system.save" is likely to be a backup copy of the "SYSTEM" hive, which stores information about the system and installed hardware, such as device drivers and services.

using `secretsdump.py` module in impacket collections that performs various techniques to dump hashes from the remote machine without executing any agent there.
so we run it with `psexec.py` to get logged in as Administrator.
we dumped the local SAM hashes and LSA secrets after running:
secretsdump.py -system system.save -security security.save -sam sam.save LOCAL
and we got access after running `psexec.py` with Administrator hashes:
psexec.py Administrator@[IP] -hashes <NTLM hash>
so we got the flag in the same directory as the first machine.
