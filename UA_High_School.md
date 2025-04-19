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

