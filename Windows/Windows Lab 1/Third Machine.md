## Third Machine

Honestly, i didn't get anything after running `nmap` so i used my first hint, then i found out that The domain controller is vulnerable to ZeroLogon (CVE-2020-1472).
after reading this Article about [ZeroLogon](https://dirkjanm.io/a-different-way-of-abusing-zerologon/).
so i understand that The vulnerability is caused by a flaw in the Netlogon Remote Protocol (MS-NRPC), which is used by domain controllers to authenticate users and other systems. The flaw allows an attacker to send a specially crafted authentication request to a domain controller, which can be accepted as valid without requiring a password. This allows the attacker to gain unauthorized access to the domain controller and potentially compromise the entire domain.
so the first thing i searched in `Metasploit` about any modules that use zerologon
and i found this one:

![](https://i.ibb.co/Zmzcc15/Screen-Shot-2022-12-29-at-10-41-16-AM.png)

basically, this module can exploit the machine with ZeroLogon using the target host and `NETBIOS` name.
`NETBIOS (Network Basic Input/Output System) is a network communication protocol that was developed in the 1980s to support networked computers running the Windows operating system. It provides services such as name resolution, datagram transmission, and session establishment.`
we got our NETBIOS name from nmap using `-A` option:

![](https://i.ibb.co/bgYLD7F/Screen-Shot-2022-12-29-at-11-42-52-AM.png)

we set the value `NBNAME` to `SRV-DC1` and `RHOSTS` to `IP` .
we run the module and the password successfully changed to empty password

![](https://i.ibb.co/k6bVQSn/Screen-Shot-2022-12-29-at-11-50-01-AM.png)

now we can use `secretsdump.py` and extract all the hashes in `SAM` database:
`secretsdump.py "SRV-DC1$@[IP]" -no-pass`
![](https://i.ibb.co/7WKFjqP/Screen-Shot-2022-12-29-at-12-00-46-PM.png)

after we got the Administrator hash we can easily log in with `psexec.py`:
`psexec.py Administrator@[IP] -hashes <hash>`
and we can get the flag in the same directory as the other machines.
