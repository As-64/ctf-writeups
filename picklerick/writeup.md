# Picklerick CTF Tryhackme Walkthrough


## 🧩 Challenge Description
A Rick and Morty CTF. Help turn Rick back into a human!****


## ▶️YouTube🔴 
If you want a video guided walkthrough of this CTF please visit @capricorncybersecuritywalkthroughs on youtube for a full walkthrough of this


## 🛠️ Approach
*** In this challenge the IP address I am using is 10.146.141.128 - please change it to suit the machine that you are using

# Basic service exploration against the target
```
nmap 10.146.141.128 --top-ports 10000 -T5 -oN top10000.txt

Host is up (0.17s latency).
Not shown: 8375 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

+ it is revealed to us ports 22 and 80 are open on the target machine - we can infer that http and ssh are running

# Basic website enumeration
+ We know that the target is running an http web server so lets see if we can discover any directories using gobuster

```
gobuster dir -u http://10.146.141.128 -w /usr/share/wordlists/dirb/big.txt -x php,txt -t 85

/.htpasswd.php        (Status: 403) [Size: 279]
/.htpasswd.txt        (Status: 403) [Size: 279]
/.htaccess            (Status: 403) [Size: 279]
/.htpasswd            (Status: 403) [Size: 279]
/.htaccess.php        (Status: 403) [Size: 279]
/.htaccess.txt        (Status: 403) [Size: 279]
/assets               (Status: 301) [Size: 317] [--> http://10.146.141.128/assets/]
/denied.php           (Status: 302) [Size: 0] [--> /login.php]
/login.php            (Status: 200) [Size: 882]
/portal.php           (Status: 302) [Size: 0] [--> /login.php]
/robots.txt           (Status: 200) [Size: 17]
/robots.txt           (Status: 200) [Size: 17]
/server-status        (Status: 403) [Size: 279]
```

+ Let's have a quick look at what is in the source code of the main web page and for comments in particular

`curl http://10.146.141.128`


Note to self, remember username!

Username: R1ckRul3s

