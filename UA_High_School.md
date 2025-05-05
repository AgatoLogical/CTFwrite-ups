# U.A. High School
Welcome to the web application of U.A., the Superhero Academy.
>### Description
>Join us in the mission to protect the digital world of superheroes! U.A., the most renowned Superhero Academy, is looking for a superhero to test the security of our new site.
>
>Our site is a reflection of our school values, designed by our engineers with incredible Quirks. We have gone to great lengths to create a secure platform that reflects the exceptional education of the U.A.

## Reconnaissance
### Ports Scan
The ports scan reveals two open ports: 22/SSH and 80/HTTP.
>
![image](https://github.com/user-attachments/assets/8c57432c-f276-4f81-b7d7-1ec433e40403)

### Port 80
After accessing the website we find a queastionaire that might be vulnerable to some injection attacks. 
>
![image](https://github.com/user-attachments/assets/37b3dd89-d4a1-461c-b1f7-d57d00a9bef5)
>
However, trying multiple different payloads leads to no results, so we look for a different vulnerability. 
>Tested Payloads
- `test123` – Basic input
- `'` – Single quote (check for SQL injection clues)
- `<script>alert(1)</script>` – XSS test
- `../../../../etc/passwd` – LFI test (Local File Inclusion)
- `%0a`, `%0d`, `%27`, `%22` – URL-encoded characters

### Page Source Code Inspection
In the head segment, we notice that the css file is located in the assets directory, hence we check whether we can access it ourselves. 
>
![image](https://github.com/user-attachments/assets/ec9b8389-4e48-41b7-914a-06ab76b8c945)
>
... and we get an empty page. 
>
![image](https://github.com/user-attachments/assets/cce1d98d-33c0-4174-ab44-1c22f845085a)
However, it does not forbid access, just doesn't display anything. Checking the network monitor, we notice that the request header sets the PHPSESSID cookie.
>
![image](https://github.com/user-attachments/assets/263dc627-0ae8-4635-87b7-fec281231c31)
>
Checking for a default file index.php, we see that the server returns 200 in response, hence confirming there exists a valid php file. Now, we can use ffuf to see if we find any valid GET parameters for that file.
>
![image](https://github.com/user-attachments/assets/b76624c6-774c-4cf1-8826-95e267dbd08f)
>
After noticing that cmd is one of the valid parameters, we can try to use it to create a reverse shell. First, we confirm the validity of cmd.
>
![image](https://github.com/user-attachments/assets/e1314f1d-49b5-4eee-b5a8-b1c86d86b632)
>
The response is encoded in Base64 format, after decoding, we get a valid result: www-data. Since the server indeed returns a valid response, we can now proceed to creating a reverse shell.  

## Reverse Shell
Before we send the malicious payload to the target machine, we start a netcat listener on our device. 
>
![image](https://github.com/user-attachments/assets/87239210-09e2-49ea-a2f6-081e2ac10af1)
>
Once that is done, we gain the reverse shell access by passing the cmd injection as a parameter:
>
![image](https://github.com/user-attachments/assets/d2d8cb92-103d-4dfe-b4fa-8d03ef7343a1)
>
We check that our listener detected an incoming connection and we successfully gained a reverse shell. 
>
![image](https://github.com/user-attachments/assets/a62df013-9850-42e8-968c-d244f185808a)
>
### Image Data Extraction
Exploring the directories, we find two images. We have to export them to our machine. We can do that by starting another NetCat listener on our machine, that will capture whatever is sents to jpg file we specify. And then, we connect to our machine from the exploited machine (through the reverse shell we established earlier) and send the files.
>
![image](https://github.com/user-attachments/assets/8565bfa9-bb15-49eb-bc47-ef42570ca74d)
>
![image](https://github.com/user-attachments/assets/3d859105-ed27-407e-983e-21d78cad0dea)
>
The image oneforall.jpg does not want to open, which raises suspicion. We need to investigate it further. We hex dump its first few bytes to check if they match a jpg file. 
>
![image](https://github.com/user-attachments/assets/0a897850-9cff-4039-a97f-cd926dac66b0)
>
It reveals that the magic bytes match a png file instead. So with the hexeditor we change the png magic bytes (89 50 4E 47 0D 0A 1A 0A) to jpg magic bytes (FF D8 FF 00 00 00 00 00).
>![image](https://github.com/user-attachments/assets/6952fb42-d05f-4680-9793-125063d5088e)
>
![image](https://github.com/user-attachments/assets/31f2bb5b-7cb5-4aa0-afd3-850ece4c8915)
>
Now, when we try opening the file we succeed. The picture itself does not reveal any sensitive data, but maybe we can extract any hidden data from it with steghide.
>
![image](https://github.com/user-attachments/assets/b48dfb22-dce3-4517-8402-967d356676f1)
>
We are prompted to enter a passphrase, however so far we haven't encountared any, so we go back to looking through the files.
>
![image](https://github.com/user-attachments/assets/18692e4a-1801-4242-b240-0acdfbede08e)
>
Let's now try to use this passphrase. Unfortunately, steghide could not extract any data with that passphrase. We can try decoding the passphrase.
>
![image](https://github.com/user-attachments/assets/1950cc6b-f416-4d8b-8c80-542538b70ff9)
>
It seems to have been a good move. Now let's use the decoded phrase as the passphrase.
>
![image](https://github.com/user-attachments/assets/3a9c93b4-1241-4657-b73b-1c9934bb97ae)
>
Success! Let's view the content of creds.txt.
>
![image](https://github.com/user-attachments/assets/12b12190-2f7b-41d6-99ab-2db2ed54b20f)
>
It seems like we have obtained Deku's account credentials. Using ssh we can try gainng access to Deku's account.
### Accessing Deku's Account
>
![image](https://github.com/user-attachments/assets/d1889076-0504-42c5-b965-def2e308e729)
>
And here we are! Right away we find the user flag in user.txt!

## Privilage Escalation
