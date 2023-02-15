# TryHackMe-Relevant-Pen-Test-Walkthrough-2-ways

## Enumeration - PrintSpoofer path
Starting out with a basic scan, a few open ports were returned, including an SMB server and webpage.
![scan](https://user-images.githubusercontent.com/103790652/218334014-fc8b38da-da51-4f1b-ae53-6c6822595e6c.png)

Investigating SMB first showed an anonymous/ guest login was possible on the n4twrksv share which contained the passwords.txt file.

![initialsmb](https://user-images.githubusercontent.com/103790652/218334215-90af2043-2071-4ebe-ac92-e6614ca9fca1.png)

![pass tct](https://user-images.githubusercontent.com/103790652/218334993-e6170224-4d3e-42fd-ab46-74f612b7b65e.png)


The file contained a base64 string that when decoded, gave a couple of usernames and passwords.

I spent some time trying to find a login page/ portal , connect to RDP, trying to sign into other SMB shares - trying to find somewhere to use the credentials but came up short so decided to backtrack and see if I'd missed something. 

After performing a more indepth scan, a few more ports in the 49000 range were returned as open and upon further specification one was returned as another webpage.
![ports](https://user-images.githubusercontent.com/103790652/218334518-bb862c09-5bb8-472e-b4c2-5a883f5db1b3.png)

Both ports used to serve the web pages were returning a generic IIS page and not much of interest but I decided to follow through with thoroughly scanning the new port with gobuster.
After quite a bit of waiting, a page was returned that was the same as the SMB share name from earlier

![gobuster](https://user-images.githubusercontent.com/103790652/218334623-b5df0328-3c2f-4a6e-8f7b-02d646728c23.png)
![result](https://user-images.githubusercontent.com/103790652/218334631-d7ad8b9e-4bd4-4998-8a6d-a64b7895aa8a.png)

Finally. After navigating to the webapge, the passwords.txt file could also be seen.
![smbfind](https://user-images.githubusercontent.com/103790652/218334723-06e53024-705d-443c-953f-5b6de0a67d3b.png)

I decided to try and upload a test file to the share given that I was able to log in -- success
![test proof](https://user-images.githubusercontent.com/103790652/218334757-55bb822c-ece9-4077-9e05-1f216be6b0a0.png)

## Exploitation

I initially crafted a php shell to upload but realised that an aspx shell would be more appropriate given the webserver was IIS based.
After uploading the aspx shell and setting up a listener, the response was caught and I had access and could grab the user.txt flag.
![shell+catch](https://user-images.githubusercontent.com/103790652/218334928-582ada99-1415-4afd-8b9c-f8a448a7f5f5.png)

## Escalation
Once in the system, I checked the users privileges via whoami /priv which retrned the following permissions
![priv](https://user-images.githubusercontent.com/103790652/219200208-d8ee4bd5-e22e-4d95-8ec6-ed860d8db4a9.png)

Initially, I checked escalation methods via ChangeNotify but found nothing promising so moved on to Impersonate. 
I found an possible mthod via https://www.plesk.com/kb/support/microsoft-windows-seimpersonateprivilege-local-privilege-escalation/ via PrintsPoofer (named pipe impersonation). "As a prerequisite, the only required privilege is SeImpersonatePrivilege" - perfect!

After downloading the .exe from https://github.com/dievus/printspoofer I uploaded to the smb share on the vulnerable server for easy access.
All that was required was to then run the program from the machine and I was granted system.
![print_sploit](https://user-images.githubusercontent.com/103790652/219204120-312677a1-209f-461e-ae8e-00903b7f9b72.png)

## Enumeration Eternal Blue path
A vulnrability script returns that the server is vulnerable to CVE-2017-0143 / the infamous Eternal Blue exploit.
![nmap vuln](https://user-images.githubusercontent.com/103790652/219210381-bb2ed937-8545-4499-b890-bc9a41272fde.png)

I decided to deploy this exploit manually, without Metasploit.
Checking for the exploit within the database:
![searchsploit](https://user-images.githubusercontent.com/103790652/219210744-9d5b1fec-9cc2-4744-9a51-1634f4ab74e0.png)

After editing the program to include a username and password (one of the ones found contained in the passwords.txt file on the SMB server) and running the exploit, there were multiple errors that prevented successful exectution. I believe that the script was written for python2, rather than the present version of python3 which handles (concatenates) variables differently (this might be wrong but is my current understanding).
I downloaded the requirements to run the program with python2 and it worked.
![exploit_test](https://user-images.githubusercontent.com/103790652/219213182-4a1a9a23-5480-4c18-8ff9-4244d3b73b65.png)

You can see that the base exploit simply drops a .txt file as a test so I located the responsible function (smb_pwn) and edited the code to drop an msfvenom payload (the same as in the above path) instead:
![code](https://user-images.githubusercontent.com/103790652/219213646-42208592-3d46-472e-b509-f8cb788e0306.png)

Set up a listener and successfully ran the exploit with the payload included:
![exploit_run](https://user-images.githubusercontent.com/103790652/219213757-6cf90973-beb7-486b-a605-852b6d52c87c.png)

I was instantly granted nt authority\system shell!
![system](https://user-images.githubusercontent.com/103790652/219214233-94397815-e0c1-4e59-8718-dc8f6be6c768.png)









