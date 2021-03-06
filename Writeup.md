# THM-Simple-CTF-Writeup
https://tryhackme.com/room/easyctf

First step is to enumerate all the open ports that exists in the target,
to get this I use nmap, the command is:

```
nmap -oA nmap_result -A 10.10.234.
```
Command breakdown:

- -oA : to output the results to a file called “nmap_result”

- -A : do an extensive scan on the target.

![NMAP Resulst](images/1-1.png "NMAP")

From the output of nmap now we know that there are 3 open ports,
that is: port 21 that is running FTP, Port 80 running a webserver, and
port 2222 running an ssh, therefore. Based on this we can answer
the following questions

1. Q: How many services are running under port 1000?
A: 2
2. Q: What is running on the higher port?
A: ssh

# Accessing the Webserver

Currently we don't know the credentials to access the SSH or FTP, so
first we need access the webserver to find anything interesting there.

![Webserver](images/2-1.png)

Open up the webserver, it only shows the default landing page for
Apache2. This is where dirsearch comes into play

# Finding directory

To find other pages or directories that may exist within the server
we can use any directory brute-force tools such as dirbuster,
gobuster, dirb, etc. But In my case I want to use dirsearch due to its
simplicity and fast. The options that I use for dirsearch are:

```
dirsearch -u 10.10.44.13 -e php,html,js -x 403
```
Command Breakdown:

- u : is the target URL.
- e : the extensions to search for the page.
- x : is to exclude certain status code, in this case is 403.

This is the result of its execution.

![](images/3-1.png)

We found 3 pages for this server. The first page called index.html is
the landing page for this server that is the apache2 page that we
visited earlier. Now, let's take a look at the robots.txt, what does it
contain?

![](images/3-2.png)

And, opening up the /openemr-5_0_1_3 fil, reveal this.

![](images/3-3.png)

Not Found. Then, lets access the final page that we found using
dirsearch. /simple

![](images/3-4.png)

It opens up the default page for CMS Made Simple, we can navigate
to other different CMSMS related page by clicking all the different 
links in this page. But, since we have questions that needs to be
answered, is there anything particularly interesting in order to solve
this room?. There is one information that I found.

By scrolling down this default page we can find the version of CMS.

![](images/3-5.png)

Then by googling the CVE or vulnerabilities for this version, we get
CVE-2019-9053, which is the answer to the questions.

3. Q: What's the CVE you're using against the application?

   A: CVE-2019-9053
   
4. Q: To what kind of vulnerability is the application vulnerable?

   A: sqli

# Executing the CVE

Now that we know what CVE to use, we can then download this CVE
from the internet, personally I got the file from exploit-db:
https://www.exploit-db.com/exploits/46653

![](images/4-1.png)

Download it by clicking the download symbol besides the word
“Exploit”, save it to your prefered folder and we're ready to go. To
execute, simply by typing this command in the console:

```
python <CVE_FILENAME>.py
```
Depends on what the current version of python installed on our
machine, the execution always leads to failure, at least in my case,
proven by this error:

![](images/4-2.png)

this is because the version of python coded in the CVE is less than 3
making the format unmatch. What we can do for this is to convert
the code into python3, we do so by using python module called
“2to3”. We can install it using pip:

```
pip install 2to
```
After it is installed, then we run:

```
python -m lib2to3 -w <CVE_FILENAME>.py
```
Command Breakdown:
- -m : tell python to use module, in this case its 2to3
- -w : is the flag from the module, it tells it to write the modification
into the file.


Now we can run the CVE. To run:

```
python 46635.py -u http :// 10.10.172.150 / simple / --crack -
w / opt / Wordlist / rockyou.txt
```
Command Breakdown:
- -u : Target URL
- -w : Wordlist to use

This command will then output the username and password of the
server, we can use this to login to admin or to connect with ssh later
on. However, though I ran to some problem that makes the script
returning which can be seen in the picture below.

![](images/4-3.png)

This output is unfinished since I only know the username and the email but not the password, I
don't know if this happens to you or the others. But fortunately,
since we know the username we can try to crack the password using
Hydra to connect to the SSH.

# Connect to SSH

We know the username, we can try to use Hydra to crack the
password for the ssh and connect to it. To use, type:

```
hydra -l mitch -P / opt / Wordlist / rockyou.txt ssh://
10.10.164.201 : 2222
```
Command breakdown:

- -l : is the login name or username for the service
- -P: is the wordlist for the password
- The command then followed by [Service]://[IP_Adress]:[Port] which
in this case is **ssh://** 10.10.164.201 **:** 2222 

Once that done, execute it and then we get

![](images/5-1.png)

Awesome, it work, now we know the password to ssh this machine,
to connect, simply:

```
ssh mitch@10.10.164.201 -p 2222
```
then we input the password and we finally connect. By doing this
process we know the answer to question 5 and 6

5. Q: What's the password?
    A: secret
6. Q: Where can you login with the details obtained?
    A: SSH

# Getting the flag

Inside the machine, we can list all files of the directory we're
currently in by typing:

```
ls -la
```
Command breakdown:

- -l : use a long format
- -a: to list all files including the hidden one

![](images/6-1.png)

the user.txt file looks interesting, open it up reveals the user flag

![](images/6-2.png)

Okay, next we can try to use sudo to list to check what commands
that are able to run as root without asking for the password, to do
this we type:

```
sudo -l
```
Command breakdown:

- -l : is to list all possible commands that can be run as root

Then we found this

![](images/6-3.png)

Looks like vim is able to be use with sudo without it asking for
password, we can use this to get the final flag in the root directory
by simply:

```
sudo vim / root / root.txt
```
it will then open up the file in vim as root

![](images/6-4.png)

And that's the final flag. Before we end this, lets take a look at the
home directory of this to see who else use this machine, a simple ls
to /home will do the job.

![](images/6-5.png)

Now we finished. In this process we answer question 6 to 9

6. Q: What's the user flag?

   A: G00d j0b, keep up!
7. Q: Is there any other user in the home directory? What's its name?

   A: sunbath
   
8. Q: What can you leverage to spawn a privileged shell?

   A: vim
   
9. Q: What's the root flag?

   A: W3ll d0n3. You made it!



