![Screenshot 2023-05-26 132650](https://github.com/CTF-Walkthroughs/Agent-Sudo-CTF-Writeup/assets/97861439/e59fc503-17e8-4677-9a25-adb260e95593)

# Agent-Sudo-CTF-Writeup
This is the writeup for the Agent Sudo CTF on TryHackMe!
https://tryhackme.com/room/agentsudoctf

tags: enumerate, hash cracking, exploit, brute-force


## **ENUMERATION**

First let's kick things off with some classic nmap scans to get a lay of the land.

export IP=10.10.246.36

export myIP=10.13.24.71

`nmap -F $IP`



Next, we'll start another scan in the background to check the higher ports.

`sudo nmap -p- $IP`

### WALKING THE WEBSITE

We find the following message.

![Screenshot 2023-05-26 100135](https://github.com/CTF-Walkthroughs/Agent-Sudo-CTF-Writeup/assets/97861439/41cd3a4b-cbdc-48bb-8a81-b677a4f4a1d3)

From this message we can deduce that we need to modify the user-agent header on the http request to get to the next hint. The hint suggests using the  header editor extension on firefox called "user-agent switcher and manager. The text mentions that there are 25 agents. Maybe we can try each letter of the alphabet as the user-agent to see who we can find. After trying the first few letters we strike gold with agent C and are redirected to: http://10.10.246.36/agent_C_attention.php

![Screenshot 2023-05-26 100829](https://github.com/CTF-Walkthroughs/Agent-Sudo-CTF-Writeup/assets/97861439/b178f0f0-d845-40da-a8af-02eeeb7610f5)

It looks like Agent C is Chris. Let's try to bruteforce the FTP as user Chris with hydra.

`hydra -l chris -P /usr/share/wordlists/SecLists/Passwords/500-worst-passwords.txt ftp://10.10.246.36
`
It worked! 

![Screenshot 2023-05-26 101048](https://github.com/CTF-Walkthroughs/Agent-Sudo-CTF-Writeup/assets/97861439/5d263857-27fa-4c67-8119-1aa8c04776bb)

Now we can grab the three files from the ftp server and continue our enumeration.

![Screenshot 2023-05-26 101133](https://github.com/CTF-Walkthroughs/Agent-Sudo-CTF-Writeup/assets/97861439/2ff76a19-7316-4776-8cd6-a68a9f8363a3)

## **GAINING INITIAL FOOTHOLD VIA HIDDEN DATA**

This message greets us in the txt file

![Screenshot 2023-05-26 101401](https://github.com/CTF-Walkthroughs/Agent-Sudo-CTF-Writeup/assets/97861439/a7c8c774-267b-49cf-b0bc-2982157fc4c4)

`steghide extract -sf cute-alien.jpeg`

![Screenshot 2023-05-26 101708](https://github.com/CTF-Walkthroughs/Agent-Sudo-CTF-Writeup/assets/97861439/d43f5d47-37ae-4171-973f-653d1eed60b6)

It looks like we don't have the password yet. Let's move on to the other jpeg file.

![Screenshot 2023-05-26 102659](https://github.com/CTF-Walkthroughs/Agent-Sudo-CTF-Writeup/assets/97861439/1588ac8d-bc0a-4066-b0e8-f3a335d431e5)

Inside the new folder we find a zip file. Let's use zip2john to extract the hash


And now use'll use john to crack the hash.

Next, let's unzip the file using this password with 7zip.

![Screenshot 2023-05-26 102753](https://github.com/CTF-Walkthroughs/Agent-Sudo-CTF-Writeup/assets/97861439/c37a9336-2eb4-4fb8-b85a-3802aafebe7b)

The text gives us a code to use. We can convert this base64 string to find the next password.

![Screenshot 2023-05-26 102859](https://github.com/CTF-Walkthroughs/Agent-Sudo-CTF-Writeup/assets/97861439/1cf3e228-ce73-4fa1-82b6-5df6d0209aa2)

This reveals the password for the hidden data in cutie.jpeg.

![Screenshot 2023-05-26 103017](https://github.com/CTF-Walkthroughs/Agent-Sudo-CTF-Writeup/assets/97861439/7c2750ef-4798-4ad3-851a-432670b1e4a2)

Let's fire up steghide again to find the SSH password for James.

![Screenshot 2023-05-26 103126](https://github.com/CTF-Walkthroughs/Agent-Sudo-CTF-Writeup/assets/97861439/274c4d51-8e9b-4607-abc3-63c62e0b278a)

![Screenshot 2023-05-26 103227](https://github.com/CTF-Walkthroughs/Agent-Sudo-CTF-Writeup/assets/97861439/b7af4957-0015-4c31-855f-2d6253d0861c)


## **ENUMERATING SSH**

Now we can log in as James via ssh with the new password.

Let's grab the user.txt flag.

## **LOCAL ENUMERATION AND EXPLOITATION**

Checking `sudo -l` shows a curious permission.

![Screenshot 2023-05-26 103454](https://github.com/CTF-Walkthroughs/Agent-Sudo-CTF-Writeup/assets/97861439/9987c161-7308-4db1-9760-d01a270a1c35)

A quick google search for exploits leads me to `CVE-2019-14287`.

We can bypass the sudo authentication with the command:
`sudo -u#-1 /bin/bash`

And now we are root and can grab the root flag and finish the last question.


## **PRIVESC**

![Screenshot 2023-05-26 103701](https://github.com/CTF-Walkthroughs/Agent-Sudo-CTF-Writeup/assets/97861439/ef879e8e-8734-4449-925f-2b11f4621fb6)

Let's grab the file Alien_autospy.jpg by spinning up a simple http server and grabbing the file to our attacker machine via the browser. We can reverse image search this picture on google and find that it relates to: Roswell Alien Autopsy

The final question is revealed in the root.txt, the name of Agent R.

Thanks for reading!
