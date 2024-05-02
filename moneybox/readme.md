# Moneybox Write-up

## Introduction
Welcome to my first official write up for a boot2root machine on OffSec's Playground, also found on vulnhub, made by [Kirthik_T](https://www.vulnhub.com/author/kirthik_t,782/)

## Objective
On the VulnHub version there are a total of 3 flags, however I played the OffSec PG version which has 2 flags, one user flag and a root flag.

## Tools used
- nmap
- feroxbuster
- steghide
- hydra

## Port Scanning
As always we start by doing an nmap scan, which came back with 3 open ports, FTP, SSH and HTTP.

![nmap scan](https://github.com/MP3vius/OffSec-Vulnhub-Write-ups/blob/main/moneybox/pictures/nmap.png)

## Web Enumeration
I kind of assumed that there would be a webpage running. The homepage was nothing special, so while the nmap was still scanning, I found a directory with feroxbuster that I checked out.

![feroxbuster scan](https://github.com/MP3vius/OffSec-Vulnhub-Write-ups/blob/main/moneybox/pictures/feroxbuster.png)

Here we have a message from someone called T0m-H4ck3r who gives us a hint, which we can see by pressing Ctrl+U

![secret page](https://github.com/MP3vius/OffSec-Vulnhub-Write-ups/blob/main/moneybox/pictures/secret.png)

As we can see, there should be a hidden directory at /S3cr3t-T3xt and when we visit that page and press Ctrl+U again, we get some sort of code/key that reads "3xtr4ctd4t4" or: "extractdata". At this point I did not know how it could be of use yet, so I put it in my notes.

![key page](https://github.com/MP3vius/OffSec-Vulnhub-Write-ups/blob/main/moneybox/pictures/secretkey.png)

## FTP Enumeration
At this point the nmap scan was done and as we saw from the screenshot earlier, FTP allowed anonymous login and had a file called trytofind.jpg which sounded interesting. I logged on and fetched the picture and figured there had to be something inside it. Upon using steghide it asked for a passphrase, which of course was the secret key we found earlier on the webpage ("3xtr4ctd4t4"). It then extracted data to data.txt, which contained evidence for weak password usage by a user, presumably called 'renu'. 

![data.txt](https://github.com/MP3vius/OffSec-Vulnhub-Write-ups/blob/main/moneybox/pictures/data.png)

## SSH Brute Force
Whenever you have an indication there is a user that has a weak password, you should check if you can brute force their password. In this case I used hydra and rockyou.txt to try and crack the SSH password of user 'renu'. And it worked!

![brute force](https://github.com/MP3vius/OffSec-Vulnhub-Write-ups/blob/main/moneybox/pictures/bruteforce.png)

## Privilege Escalation
When we succesfully logged on with renu's cracked password, we cat out the first flag.

![user flag](https://github.com/MP3vius/OffSec-Vulnhub-Write-ups/blob/main/moneybox/pictures/flag1.png)

After that I tried sudo -l but renu was not allowed to use sudo on this machine. Next I tried looking at renu's .bash_history and noticed he generated SSH keys for a user called lily, which I found in renu's .ssh folder. (After I pwned this machine I learned of another way that might have been easier after all, I will also share this method at the end of this write-up.)

![ssh key](https://github.com/MP3vius/OffSec-Vulnhub-Write-ups/blob/main/moneybox/pictures/sshkey.png)

After saving the key to id_rsa and giving it the right permissions, we use it to log in to ssh as lily and check if lily is able to run any commands as root with sudo -l

![sudo -l](https://github.com/MP3vius/OffSec-Vulnhub-Write-ups/blob/main/moneybox/pictures/sudo.png)

We can see that lily is able to run perl as root and this is an easy win for us. We now simply run this specific command to spawn a root shell and cat out the root flag:

![perl win](https://github.com/MP3vius/OffSec-Vulnhub-Write-ups/blob/main/moneybox/pictures/root.png)

## Alternative Method
Initially I checked the bash_history of renu and found he generated SSH keys for lily. I then had to locate the key, cat it out, copy and save it to a file on my own system, give proper permissions and then use it to log in. However, a faster way would've been if I went to lily's home folder and then to her .ssh folder. There was a file called "authorized-key" that mentioned renu. This meant that renu could connect to ssh as lily without a password.

## Overview
Overall I thought this was a perfect box for beginners like me, it felt really satisfying to move laterally as well, as opposed to just gaining a foothold and elevate to root from there. I also knew about steghide beforehand but never used it myself, so I had to look up how it worked exactlt, which was pretty straightforward. Thank you for reading this write-up, see you in the next one!

