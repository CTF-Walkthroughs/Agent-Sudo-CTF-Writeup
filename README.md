![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/87d1ad0c-8e43-401c-a36b-ce7365a976c0)

# Agent-Sudo-CTF-Writeup
This is the writeup for the Agent Sudo CTF on TryHackMe!
https://tryhackme.com/room/agentsudoctf

tags: enumerate, hash cracking, exploit, brute-force


## **ENUMERATION**

First let's kick things off with some classic nmap scans to get a lay of the land.

export IP=10.10.246.36
export myIP=10.13.24.71

`nmap -F $IP`

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/55cd8569-57ae-4ae5-8e5c-d67451f26f67)

Next, we'll start another scan in the background to check the higher ports.

`sudo nmap -p- $IP`

### WALKING THE WEBSITE

We find the following message.

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/eba6986b-8560-42f1-8a74-b726792e9d6f)

From this message we can deduce that we need to modify the user-agent header on the http request to get to the next hint. The hint suggests using the  header editor extension on firefox called "user-agent switcher and manager. The text mentions that there are 25 agents. Maybe we can try each letter of the alphabet as the user-agent to see who we can find. After trying the first few letters we strike gold with agent C and are redirected to: http://10.10.246.36/agent_C_attention.php

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/f7f58772-7666-4edb-8657-d8a07b2639c8)

It looks like Agent C is Chris. Let's try to bruteforce the FTP as user Chris with hydra.

`hydra -l chris -P /usr/share/wordlists/SecLists/Passwords/500-worst-passwords.txt ftp://10.10.246.36
`
It worked! 

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/c7035f3a-0371-4561-b7d7-6154c44a47ed)

Now we can grab the three files from the ftp server and continue our enumeration.

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/33f3d16d-10f9-40c1-84ee-d13efb86db7e)

## **GAINING INITIAL FOOTHOLD VIA HIDDEN DATA**

This message greets us in the txt file

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/00ff7daf-ec6c-47d5-be68-d7912eaf5a00)

`steghide extract -sf cute-alien.jpeg`

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/bad3a9a1-38d6-47e6-9236-947da98af8ab)

It looks like we don't have the password yet. Let's move on to the other jpeg file.

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/f23dee20-e788-47be-82d2-eecd791a51c8)

Inside the new folder we find a zip file. Let's use zip2john to extract the hash

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/53191077-ab2f-4835-a833-bcd08e78311e)

And now use'll use john to crack the hash.

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/28cb7edf-97fb-40ad-83d6-3ab2b61bb2cc)

Next, let's unzip the file using this password with 7zip.

The text gives us a code to use. We can convert this base64 string to find the next password.

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/a4d86607-2b0f-4199-829c-473f7b9ec44e)

This reveals the password for the hidden data in cutie.jpeg.

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/a3ebd2ae-e296-4e72-852a-de524b7956d6)

Let's fire up steghide again to find the SSH password for James.

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/b0cf417d-fd24-4725-8aca-6c295f1534bd)

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/0cbef846-1f86-4df1-a6ff-49432cc4ed9a)

## **ENUMERATING SSH**

Now we can log in as James via ssh with the new password.

Let's grab the user.txt flag.

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/b1aea0f9-2361-4db8-a3cb-7cec40203434)


## **LOCAL ENUMERATION AND EXPLOITATION**

Checking `sudo -l` shows a curious permission.

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/9817d8b4-ff6a-4b30-9cf0-5a3e0f0304dc)

A quick google search for exploits leads me to `CVE-2019-14287`.

We can bypass the sudo authentication with the command:
`sudo -u#-1 /bin/bash`

And now we are root and can grab the root flag and finish the last question.


## **PRIVESC**

![image](https://github.com/Benjamin-James-Reitz/Agent-Sudo-CTF-Writeup/assets/97861439/718e5301-c827-4168-bf8d-d63d7609027b)

Let's grab the file Alien_autospy.jpg by spinning up a simple http server and grabbing the file to our attacker machine via the browser. We can reverse image search this picture on google and find that it relates to: Roswell Alien Autopsy

The final question is reveled in the root.txt, the name of Agent R.

Thanks for reading!
