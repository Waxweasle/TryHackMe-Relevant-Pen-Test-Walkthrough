# TryHackMe-Relevant-Pen-Test-Walkthrough

## Enumeration
Starting out with a basic scan, a few open ports were returned, including an SMB server and webpage.
![scan](https://user-images.githubusercontent.com/103790652/218334014-fc8b38da-da51-4f1b-ae53-6c6822595e6c.png)

Investigating SMB first showed an anonymous/ guest login was possible on the n4twrksv share which contained the passwords.txt file.
![initialsmb](https://user-images.githubusercontent.com/103790652/218334215-90af2043-2071-4ebe-ac92-e6614ca9fca1.png)

![pass tct](https://user-images.githubusercontent.com/103790652/218334993-e6170224-4d3e-42fd-ab46-74f612b7b65e.png)


The file contained a base64 string that when decoded, gave a username and password.

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

I initially crafted a php shell to upload but realised that an aspx shell would be more appropriate given the webserver was IIS based.
After uploading the aspx shell and setting up a listener, the response was caught and I had access and could grab the user.txt flag.
![shell+catch](https://user-images.githubusercontent.com/103790652/218334928-582ada99-1415-4afd-8b9c-f8a448a7f5f5.png)



