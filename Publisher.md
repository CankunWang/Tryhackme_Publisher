---
Title: "Publisher — TryHackMe Writeup"
Author: Cankun Wang
date: 2025-11-04
tags: [tryhackme, writeup, rce]

---

#Publisher - Tryhackme Writeup

This writeup will provide details about the process of Publisher CTF machine.

This is my first writeup, so if there are any problems please email me: kb010098@bu.edu

#user.txt

We know through a series of enumeration techniques, including directory fuzzing and version identification, a vulnerability can be discovered and allowed for Remote code execution(RCE). These information is provided. 

We start with using nmap to scan the opening ports.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-16-20-14-image.png)

Nmap done, but we didn't find enough useful information.

Let's try to scan all the ports for more details as well as the services. 

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-16-22-33-image.png)

We still don't find much more informations. Let's take a look at the target website.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-16-23-49-image.png)

We find something interesting here---SPIP. Some outdated version of SPIP may exist vulnerabilities for RCE, so let's try to find the version of SPIP.

We fist view the page source, but there are no more information provided here.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-16-28-26-image.png)

Let's try the directory fuzzing for more possible directories. 

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-16-31-05-image.png)

We use the gobuster and a medium wordlists to find the possible hidden directories.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-16-32-37-image.png)

Results is nice, we find /spip and /images. Let's take a look at these two. 

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-16-40-14-image.png)

Not much more information. Let's view the page source.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-16-41-01-image.png)

Nothing interesting here. Let's take a look at /images

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-16-41-50-image.png)

It's looks like this is the place of storing images, but this isn't used for uploading the images. The page source is also not provided useful information. Let's go deeper into the directories. 

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-16-45-13-image.png)

Since we want to know more details about SPIP, we first do the directory fuzzing on /spip

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-16-46-12-image.png)

There are a lot more useful directories here. Let's first take a look at /local.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-16-47-08-image.png)

A lot more interesting file here. Let's take a look at config.txt.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-16-48-02-image.png)

An interesting point here. We now know the version of SPIP this website is using---4.2.0

This is a quite useful information. There is a known vulnerability of SPIP that could allow RCE---CVE-2023-27372.(We could find this information either by searching on the Google hacking database or searching in Metasploit framework)

Let's search in the MSF.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-16-57-08-image.png)

We find it. Let's use the first one.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-16-58-11-image.png)

Show options will show all the required infos. Let's set the LHOST, RHOSTS and the payload. We need a payload here.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-17-02-46-image.png)

There are many payloads here, so we just use 3 here, a more generic payload.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-18-47-13-image.png)

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-18-48-11-image.png)

We successfully open a session after exploiting the target.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-18-52-11-image.png)

Running whoami we know we are currently in www-data; next when we back to parent directories, we find a user directory think.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-18-53-06-image.png)

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-18-53-32-image.png)

We find the user.txt in think directory!

#establish SSH

When we successfully access to the target, we can check the ssh directory.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-18-56-13-image.png)

Obviously, there is a id_rsa key here. We can use it to establish ssh connection as user think.

Since we don't have the access to use wget or something else to get the file, we can just copy it.

Then use ssh to establish connection.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-18-59-52-image.png)

#Root.txt

##Escalation privilege

After SSH login as think, we find that we need to find a way to escalation privilege. 

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-22-39-02-image.png)

We don't know the password of think, so we need to find another way. Let's first check the file that is running as root. 

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-22-42-15-image.png)

There is a lot. 

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-22-58-11-image.png)

Run_container has root access. 

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-23-00-00-image.png)

This is our target. run_container has root access and it will finally run the file in /opt.

Proceed to /opt.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-23-04-23-image.png)

We don't have write access in /opt

The hint tells us to look at Apparmor. Let's take a look.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-23-05-29-image.png)

Apparmor is running. And we are currently running in ash shell.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-23-06-34-image.png)

Then let's look at the content of apparmor file.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-23-09-07-image.png)

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-23-09-18-image.png)

It denies our access for several directories.

However, we find that flags=(complain), which means it doesn't actually ban our access.

Now we know what to do next.

First, let's find a directory which we have write access.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-23-12-58-image.png)

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-23-13-12-image.png)

We find it! We have write access at /dev/shm.

Let's continue.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-23-14-15-image.png)

Now we have a bash that has write access, and it is not limited by ash.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-23-24-00-image.png)

/dev/shm/bash will make sure we are writing the command in bash, not ash. 

Because the apparmor can only "complain", once we restart the run_container.sh, this command will directly write into apparmor file. Thus, we successfully bypassed the apparmor.

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-23-24-10-image.png)

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-23-25-26-image.png)

![](C:\Users\Administrator\AppData\Roaming\marktext\images\2025-11-04-23-25-38-image.png)

Now, we finally find the root.txt.

Thanks for reading!
