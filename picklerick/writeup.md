# Picklerick CTF Tryhackme Walkthrough


## 🧩 Challenge Description
A Rick and Morty CTF. Help turn Rick back into a human!****


## ▶️YouTube🔴 
If you want a video guided walkthrough of this CTF please visit @capricorncybersecuritywalkthroughs on youtube for a full walkthrough of this


## 🛠️ Approach
*** In this challenge the IP address I am using is 10.146.141.128 - please change it to suit the machine that you are using ***

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
We know that the target is running an http web server so lets see if we can discover any directories using gobuster


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


Let's have a quick look at what is in the source code of the main web page and for comments in particular. I'm going to use curl here to do this but you can visit the web page in person if you choose, just for me curl makes it easier


```
curl http://10.146.141.128
```


We can see the result is that we have a little note which contains a username we can use once we try and login. Maybe we will find a password to be for now lets keep enumerating the website for anything more of interest to us


```
Note to self, remember username!

Username: R1ckRul3s
```

I'm also gonna check robots.txt for anything as classicly web paths for the search engine to see are there. I'm going to use curl for this as it makes it easier to examine the results but feel free to browse it in your browser


```
curl http://10.146.141.128/robots.txt
```


We have obtained another item - this one looks like a password. This means we now have a full credential set that can be used at the path 'login.php' which we discovered using gobuster in the earlier stage of our enumeration

```
Wubbalubbadubdub
```


I now move my attention over to 'login.php' since we now have a full credential set 


<img width="1916" height="867" alt="image" src="https://github.com/user-attachments/assets/8578198b-c58d-44e0-8a60-9cc68118b4a0" />

# Web exploitation
Using the credentials we obtained I login to the website and I find that we have a command execution panel. A very easy example of command execution, but I suppose this is just a beginner level CTF so it makes sense. A great CTF to get a first touch of command execution. Here I am going to execute the ls command and see what files we have in the current working directory on the system


<img width="1911" height="900" alt="image" src="https://github.com/user-attachments/assets/30f24425-1d3c-48d3-90a8-895ef6be337b" />


We can see that we have our first ingredient. But we can't execute the cat command! We can get round this by executing the 'less' command which more or less does the same job slightly different but isn't filtered by the WAF.


<img width="1906" height="891" alt="image" src="https://github.com/user-attachments/assets/4690c2a4-353c-404c-8aa7-138b84ff0899" />


Now that we have the first ingredient lets keep going and see what we can do next. After exploring the system a little further, I find that we have a user called rick which has its home directory in /home/rick. Lets see what is in Rick's home directory using another ls command.


<img width="1868" height="850" alt="image" src="https://github.com/user-attachments/assets/a37f358e-e455-4348-8f60-25b4d03ebfd9" />


We can see that in this directory we have the second ingredients! I'm going to use the less command once again to extract the second ingredient


<img width="1860" height="846" alt="image" src="https://github.com/user-attachments/assets/4f41da1b-0d16-40b2-8cbe-87bf154f0be7" />


# Privilege escalation
Now that we have obtained the first and second ingredient we must find a way to escalate our privileges to root since the last ingredient will be in the root directory on the system meaning we need root privileges to read what it is. I'm going to try the traditional 'sudo -l' quickly against the panel to see if we can run on commands with sudo privileges


<img width="1866" height="880" alt="image" src="https://github.com/user-attachments/assets/2dee42ad-4bc8-4bb6-96e9-a750cbfcd2ba" />


It appears that we can run ANY command with the sudo command. This means we essentially have root privileges and can pretty much do anything. Well in that case, I'm gonna list what is in the root directory using 'sudo ls /root' to list the root directory and see what is in it which I would not normally be able to do if I didn't have root privileges


<img width="1895" height="783" alt="image" src="https://github.com/user-attachments/assets/6c4e9344-fbb6-4998-89d6-1a629dafb867" />


As you can make out from the image above, there is the third ingredient in the root directory called '3rd.txt'. I'm going to once again bypass the WAF and use the 'sudo less /root/3rd.txt' command to read what is in the root directory


<img width="1862" height="868" alt="image" src="https://github.com/user-attachments/assets/86d5e7e3-8f4c-44f5-8bc0-80ccfafd5850" />


And that is that! Thanks for following this walkthrough through today. I hoped it helped you solve this CTF and I hope you learnt something today!
