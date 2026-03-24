I’ve read a good few writeups to help me with CTF’s so thought it’s about time that I gave something back to the community. In this writeup, and walkthrough I will dig into how to hack templates CTF using kali linux. This is purely my methodology — there are many ways of tackling this challenge. I will be using the following IP:

http://10.10.21.165





Thanks for reading capricorn’s Substack! Subscribe for free to receive new posts and support my work.

CTF Details

Name: Templates

Type: Red

Length: 30 minutes +



Description:

Pug is my favourite templating engine! I made this super slick application so you can play around with Pug and see how it works.



Enumeration:

Without any further ado, I’m going to dive straight into this fabulous CTF. Let’s start the way we should always start — a simple NMAP scan to identify what ports are open

nmap --top-ports 1000 10.10.21.165





An NMAP scan against the target machine

We can see in the above that ports 22 and ports 5000 are open. Let’s investigate port 5000 since port 22 is simply an SSH server

nmap -A -sS -T5 --version-intensity 5 --reason 10.10.21.165 -oN advancedScan.txt -p 5000





A quick version scan using NMAP

After an in depth NMAP scan we discover that an HTTP web server seems to be running on port 5000

Now that we know we have HTTP running on the target machine, this is good news. It means we have a potential vulnerable point. Let’s take a look at this web page and see what we can infer from some basic enumeration





Me opening the website on my Kali Linux machine


What we seem to have is a website using the pug templating engine



Pug is an HTML pre-processor and template engine that simplifies writing HTML by using a clean, indentation-based syntax. It converts Pug code into native, human-readable HTML, allowing developers to define dynamic content using JavaScript variables, conditions, and loops, as well as reusable components via mixins and template inheritance.

This means that this website could very well be vulnerable to a type of attack known as SSTI (Server Side Template Injection)



Server-Side Template Injection (SSTI) is a web security vulnerability that allows an attacker to inject malicious code into a template, which is then executed server-side. This can lead to Remote Code Execution (RCE)



Exploitation:

Great. Now we are into the exploitation phase of this CTF. My favourite part of any CTF. Now that we have the enumeration done I am going to user a wordlist that I have downloaded called ‘ssti-payloads.txt’ to fuzz the target machine. This is basically a wordlist of potential ssti payloads that can lead to exploitation, a bit like a sql payload list. It can help us identify if the server is vulnerable.

When we press the convert button it takes us a page labelled render and renders the content that we entered in HTML.





My firefox once I press convert

Now lets fuzz against the website ‘http://10.10.21.165:5000’ with the list and see what we get.

ffuf -w ~/Downloads/ssti-payloads.txt -u http://10.10.21.165:5000/FUZZ -fs 0

I got the output below by running the above command in bash — try it in your machine and see what you get





The above output means that these payloads successfully worked as an injection. So on the screen, it should output ‘9’ if this is not just a false positive. Let’s try it out in our firefox browser





Me putting it into the template engine

I click convert and lets see what we get





We have successfully carried out Server Side Template Injection. This means the server is vulnerable and can be attacked to give us remote code execution. Now we know this is pug and it is vulnerable we can do a little bit of research on a best friend google to try and find out how to run commands.

After a bit of research on google, I found out how to execute commands with pug. It seems easy enough, so lets try to inject this into the template engine

#{global.process.mainModule.constructor._load(’child_process’).execSync(’ls’)}

I ran the code into the template engine as we did before and it succesfully printed out the working directory contents as ‘ls’ should do.

Now I can simply use this technique to extract flag.txt which will give us what we want for the CTF. I shall adapt the code to the code I just put below to suit our needs.

#{global.process.mainModule.constructor._load(’child_process’).execSync(’cat flag.txt’)}

Executing on the pug template engine we are given our flag





Final flag


Well there we have it — we’ve successfully extracted the flag. What a CTF! I personally really enjoyed this CTF — having done it a couple of hours before the write up I decided it would be great to create a write up for this brilliant machine that didn’t really have any write ups!

Produced by @CapricornCyberSecurityWalkthroughs find more useful hacking content on our YouTube channel



Thanks for reading capricorn’s Substack! Subscribe for free to receive new posts and support my work.
