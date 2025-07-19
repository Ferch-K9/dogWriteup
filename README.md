# Write Up for The HTB Dog box 

## Enumeration

Launching into the Box my IP was `10.10.11.58`

Using Nmap I will Scan the Ip for open Ports `nmap -sV -T4 <ip>`

<img width="776" height="204" alt="NmapDogPortScan" src="https://github.com/user-attachments/assets/221694be-8075-46e3-a92e-24ef1ec1235b" />

We can see there is a http service running so I run `nmap -sC <ip>` to use scripts to get more information.

<img width="790" height="389" alt="NmapDogWebScan" src="https://github.com/user-attachments/assets/70c2cd36-51f8-46aa-bfb7-ba217ccd5f3c" />

## Expoiting web misconfiguration

First thing that comes up is `Backdrop CMS` which will be important later.

 We also see that the websites `/.git` is exposed to the internet which is a configuration error. 

 Using a program called git-dumper we can grab the contents. `git-dumper http://10.10.11.58/.git dog.git`

 \***Note**\* this wasent installed for me so I used the command `pipx install git-dumper` To download the program.
 
<img width="654" height="239" alt="dogGitFile" src="https://github.com/user-attachments/assets/f5e14571-8c63-42f5-9007-8bc4b54e8530" />

Using Nano I can look into the `settings.php` to try and find the password the database \(Backdrop CMS\) uses to connect to the website


