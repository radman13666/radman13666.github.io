---
layout: post
title:  "[THM Experience] - Wreath Network - New Things Learnt"
date:   2021-07-23 03:51:29 +0300
tags: tryhackme easy experience learning
categories: tryhackme
---
![THM Network - Wreath Network](/assets/thm/wreath/thm-wreath.png)

**Room URL**: <https://www.tryhackme.com/room/wreath>

**Difficulty**: Easy

## INTRODUCTION

For around two months now, I've been digging deep into the rabit hole that is Red Team Operations, at least the cyber-part of it. To put it simply, it's basically using one's hacking skills to mimic an adversary with respect to a target environment. This target environment can range from a multi-national multi-billion dollar empire to a small home network. The purpose of this mimicing is to provide the defending forces in target environment to better their defenses and test their incident detection and response.

Being a blue teamer (defending force) in my work place right now, thinking like a red teamer (an attacking force) is VERY benefitial because you learn how the attackers think and therefore, how to defend against them. So I dug deeper into the topic of Red Team Ops, I wanted to have a practical way to experience doing some of that stuff, but I didn't know anywhere to even start. Then recently, in June 2021, as I was exploring things to do on TryHackMe, I discovered the Wreath Network.

The Wreath Network is a "room" on the TryHackMe platform that, unlike a regular room there, comprises multiple machines that are availed to the platform's users to learn a lot of cool stuff (that I'm going to talk about in a moment). It is rated "easy" because it is meant for teaching particular skills and thus, was designed to be a walkthrough. From the introduction into the room:

***Wreath is designed as a learning resource for beginners with a primary focus on:***

- ***Pivoting***
- ***Working with the Empire C2 (Command and Control) framework***
- ***Simple Anti-Virus evasion techniques***

***This is designed as almost a sandbox environment to follow along with the teaching content; the focus will be on the above teaching points, rather than on initial access and privilege escalation exploits (contrary to other boxes on the platform where the focus is on the challenge).***

Imagine my excitement when I read these words. In as much as there is more to Red Team Ops than just the above mentioned topics, they are skills that are essential to executing a successful Red Team operation. I was ecstatic! Thinking that it couldn't possibly get any better, it did:

***Note: You are also encouraged to treat this Network like a penetration test -- i.e. take notes and screenshots of every step and write a full report at the end (especially if you're not already familiar with writing such reports). Keeping track of any files (e.g. tools or payloads) and users you create would also be a good idea. Reports will not be marked, but the act of writing them is good practice for any professional work -- or certifications -- you may do in the future.***

This is every beginning cybersecurity professional's dream; to have a way to practice and learn skills for penetration testing and/or red team operations and also practicing and learning how to put their actions into written words that should be understood by both techies and non-techies alike. I dived in IMMEDIATELY!!!

I went through the material and finished the entire experience in one week. It was everything I wanted, despite that fact it was teaching entry-level concepts. But that's the thing; when one gets the basics and fundamentals down, there is no other way but up for them. Having gone through that experience, I decided to share here the new things I learnt during that journey.

## NEW THINGS LEARNT

### PIVOTING

The only kind of pivoting I knew prior to undetaking this network was **[SSH port forwarding](https://phoenixnap.com/kb/ssh-port-forwarding)**. There is A LOT more out there.

So from what I learnt while working in the Wreath network, pivoting is a technique to leverage a network host / endpoint in penetrating one's presence in a network. Basically, using the host (a.k.a the pivot) as a stepping stone to reach other network hosts, which may possibly be unreachable via the public Internet. To a layman, this is similar to using a local VPN connection from a home computer to reach the internal corporate network at work. This should be very farmiliar given the prevailing global situation. There are two main methods to perform pivoting; tunneling and port forwarding.

**Tunneling / Proxying** is a connection type which allows a client machine to transmit it's network traffic through another machine using... (you guess it) a tunnel. This tunnel allows the client to make use of the pivot's IP routes to transfer the traffic to the internal target destination. This can be used to evade IDS, especially if using SSH tunnels, where the traffic is encrypted. A few example of tools that can help one achieve this are sshuttle and proxychains.

**Port forwarding**, unlike tunneling, does not allow passing all traffic from the client machine to the target machine, but rather passes traffic on specified ports. An example of this would be receiving HTTP traffic from port 443 of an internal corporate server through port 4443 on your local machine, by using SSH port forwarding like I mentioned earlier. This method tends to be faster and more reliable, but limits traffic flow to other ports if that's the requirement, such was during a port scan. Example tools that can help one achieve this are socat, openssh clients and chisel.

I always had a sense of what pivoting was, but my problem was how to execute it. There are multiple tools to perform pivoting that I discovered while in the network, and now I know how.

### COMMAND AND CONTROL (C2)

When I started working in cybersecurity in 2019, the first thing I heard (and eventually got as my first point of research) was command and control servers; how they worked and how to take them down (or at least reduce their influence on the network).

From my studies in this network and **[other AWESOME material](https://youtube.com/playlist?list=PL9HO6M_MU2nfQ4kHSCzAQMqxQxH47d1no)** I've gone through, I got to understand what they are and how they are setup, and what to look for to confirm their indicators.

Basically, from my undestanding, Command and Control is about consolidating red team tasks on target networks using an integrated set of tools, hosted on a server (or set of servers), enabling the team to gain and retain access to the network, perform lateral movement and escalate privilege as required to fulfil the desired objectives. The servers on which these activities are done are known as Command and Control servers (a.k.a CnC servers or C2 servers).

There exist multiple platforms and frameworks that have been created to work as functional C2 servers such as **[Cobalt Strike](https://www.cobaltstrike.com/)**, **[Powershell Empire](https://github.com/BC-SECURITY/Empire)** and **[Covenant](https://github.com/cobbr/Covenant)**. More can be found and compared using the **[C2 matrix](https://howto.thec2matrix.com/)**. In the Wreath network, I was introduced to Powershell Empire as a C2 server framework and I love it because it's free and opensource software (FOSS) and it has functionality to support controlling both UNIX-like (i.e. MacOS & Linux) and Windows hosts with Python and Powershell payloads respectively.

### ANTI VIRUS (AV) EVASION

This was the icing on the cake for me in this network. It was basic, but the foundational information from the experience is golden! AV Evasion is exactly that; the different techniques applied to reduce the likelihood of malicious software being flagged as so by anti-virus solutions, like Windows Defender.

The activity itself is complex and requires a lot more experience, knowledge and practice to get it right. It's both an art and a science. From what I learnt, AV evasion can be done at 2 main levels; **on disk**, where malicious programs are steathily saved to disk and executed, and **in memory**, where code is run in working computer memory a.k.a RAM (random access memory).

For AV evasion to succeed, time must be invested on gathering information about the target's environment, replicating it in a local lab environment and testing different payload variations to find which ones are successful and why. The variations are generated through **obfuscation** at multiple levels.

To really be successful at evasion, understanding how the anti-virus solution detects malware is CRITICAL! Basically, there are two main ways of detection; **static** detection, where the malware is examined for known signatures / indicators without running it. It's basically looking at what the malware "looks like". The other method is **dynamic** detection, where the malware is executed in a sandboxed environment to examine it's actions. It's basically finding out how the malware "behaves". There could be new methods already out there and it's always a neck-on-neck race between the defending forces and the attacking forces.

### OTHER TRICKS AND NOTES I LEARNT

So this section basically shows some tricks I picked up that were really cool and I saw would make my hacking life (and general life as a Linux user) easier.

#### Sharing folders using RDP clients

With **[Evil-WinRM](https://github.com/Hackplayers/evil-winrm)**, one can use Windows static binaries and Powershell scripts without having to actually download them on a remote Windows system. This can be done with the `-s`
 and `-e` options on the terminal.

Example usage to load Powershell Empire network scripts is here:
```
$ evil-winrm -i 10.10.10.10 -P 5985 -u Administrator -p password -s /usr/share/powershell-empire/data/module_source/situational_awareness/network/
```

Example usage to load static Windows binaries with `xfreerdp` client, while enabling clipboard and window resizing:

```
$ xfreerdp +clipboard /u:Administrator /p:password /cert:ignore /v:10.10.10.10 /port:3389 /dynamic-resolution /drive:/usr/share/windows-resources,share
```

#### Getting ExploitDB exploit code (the cool way)

Initially when I would want to use an exploit from **[ExploitDB](https://www.exploit-db.com/)**, I would copy it from the folder directly.

```
$ cp /usr/share/exploitdb/exploits/path/to/exploit.py .
```

But the better way is this (taking note the the EDBID is the ExploitDB ID of that particular exploit, usually displayed when querying using `searchsploit`):

```
$ searchsploit -m EDBID # copies over the code as EDBID.extension
$ dos2unix EDBID.extension # coverts line endings from CRLF to LF. VERY USEFUL!
```

I can't picture my life without this anymore.

#### SSH key files end in a single LF line ending

During the engagement in the Wreath network, I had to obtain an SSH private key. Little did I know that I hadn't copied it completely. I missed out the last empty line (a.k.a the line with a single LF line ending).

This resulted in me getting frustrated as to why I wasn't able to log into one of the hosts on the network via SSH. It went on for close to 2 days until I discovered it was all my fault here: <https://newbedev.com/ssh-suddenly-returning-invalid-format>. I laughed at myself so hard! Haha.

#### Inactive Windows accounts have "empty" hashes

During the last parts of the Wreath network engagement, I learned that when password hashes of Windows user accounts are dumped, not all hashes are as sensitive as I had initially thought.

Account hashes that have `31d6cfe0d16ae931b73c59d7e0c089c0` as an NTLM hash are **inactive** accounts i.e. their passwords are **EMPTY**. Like a literal ''!

Completely blew my mind! You can verify it yourself by submitting this hash to <https://crackstation.net/>.

## THE REPORT

So like I initially said in the INTRODUCTION, the Wreath network creator recommended that users treat it like a real network penetration test and we were encouraged to make a report on it.

So, using the **[CherryTree](https://www.giuspen.com/cherrytreemanual/)** note-taking application, that comes pre-installed in Kali Linux, I took notes on what I did in the network, and also used it to make the report. This was the first time to use CherryTree to make any kind of report, and it will also the last!!! It was painful. It's better used for just note-taking.

Anyway as a wrap up to this post, I tried really hard to make the report a good report to the best of my ability given my circumstances. You can find the report **[here](/assets/thm/wreath/thm-wreath-report--radwolfsdragon.pdf)**. Let me know what you think of it. Also, go try out the network yourself. You'll not regret it.

Until next time, stay cool and stay you.
