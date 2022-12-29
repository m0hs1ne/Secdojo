## Fourth Machine

`nmap` result:

![](https://i.ibb.co/x6Z9Xh0/Screen-Shot-2022-12-29-at-12-41-02-PM.png)

in port 80 we see there is `HttpFileSever` with version `2.3`.
so i searched if there is any exploit in this version, and i found that the findMacroMarker function in parserLib.pas in Rejetto HTTP File Server (aka HFS or HTTP Fileserver) 2.3x before 2.3c allows remote attackers to execute arbitrary programs via a %00 sequence in a search action.
so i used `Metasploit` to search for any module using this vulnerability and i found:

![](https://i.ibb.co/VJS3Rtj/Screen-Shot-2022-12-29-at-1-14-30-PM.png)

then i just set `RHOSTS` with the machine IP, then i got the access:

![](https://i.ibb.co/R7pZYRg/Screen-Shot-2022-12-29-at-1-17-20-PM.png)

in the same directory as others, i got the last flag.
