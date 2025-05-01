# pWnOS-1.0-Walkthrough

This writeup covers the exploitation of the pWnOS CTF machine, highlighting key steps like network scanning, path traversal, and privilege escalation using public exploits.

## Target Discovery

We start by identifying the IP address of the target machine using `netdiscover`.
```bash
netdiscover
```
![image](https://github.com/user-attachments/assets/a4a0753b-3748-4dd0-821d-0b92f7a7ac1a)

Once we identify the IP, we perform a comprehensive scan using `nmap` to find open ports and running services:
```bash
nmap -sS -sV -A <target-ip>
```
![image](https://github.com/user-attachments/assets/f20a38a4-dda1-4163-b2b6-6e1d96e0f6ca)

## Web Enumeration

Upon accessing the target via the HTTP port in a browser, we are greeted by a basic website.
![image](https://github.com/user-attachments/assets/9444efd6-8612-4469-94d4-6e4945007ba7)

Trying to manipulate the URL leads to errors. This hints at the possibility of a **Path Traversal** vulnerability. We test this by appending common file paths:

![image](https://github.com/user-attachments/assets/47b22c10-0fc2-4e65-8f0c-016ee0bac707)

```bash
connect=/etc/passwd
```
![image](https://github.com/user-attachments/assets/4e759469-3036-4898-80c0-8057ce94a40f)

Success. This grants access to the contents of the `/etc/passwd` file. We extract four usernames:

- obama  
- osama  
- yomama  
- vmware  

At this point, our enumeration on this port is complete.

## Exploring Port 10000 (MiniServ)

Further `nmap` scanning reveals **port 10000**, commonly used by **Webmin** (MiniServ v0.01 in this case).

After some online research, we find a working [**exploit**](https://github.com/CyberKnight00/Exploit/blob/master/Webmin%20%3C%201.290%20Usermin%20%3C%201.220%20-%20Arbitrary%20File%20Disclosure/webmin.py) targeting this outdated MiniServ version. The exploit allows us to retrieve the `/etc/shadow` file.

![image](https://github.com/user-attachments/assets/8290e050-7a37-40f7-a515-c2d6aad75f94)
![image](https://github.com/user-attachments/assets/e8c7f759-a97a-41b4-9a6d-47bad163ee3d)

A reference website is found that explains how to **decrypt the shadow hashes** with using [**John the Ripper**](https://erev0s.com/blog/cracking-etcshadow-john/).

![image](https://github.com/user-attachments/assets/26f601da-e1dc-450d-96b4-20f56ec07fb4)

Here is the found password for one of our found usernames.

![image](https://github.com/user-attachments/assets/d7c3c17d-4470-47da-aa27-9d4c4a47ec41)


## Gaining Shell Access

We check the Linux distribution running on the target. It turns out to be an outdated version, which opens the door for **privilege escalation** exploits.

![image](https://github.com/user-attachments/assets/9a6fd9d7-abaf-480b-a96e-9db69480e2b7)

![image](https://github.com/user-attachments/assets/9c1b184a-adda-4962-810e-78ef9d5fe4cd)

![image](https://github.com/user-attachments/assets/c4624d7f-29ff-4ba6-93db-b0cc54f8c128)

After more research, we locate a suitable exploit at [**Exploit-DB**](https://www.exploit-db.com/exploits/5092).

![image](https://github.com/user-attachments/assets/22799c16-ef72-46b5-9c50-5eb9b3907183)

We set up a local server using Python's built-in HTTP module to upload the exploit to the target machine:

```bash
python3 -m http.server 8000
```

Then, on the target machine, we use `wget` or `curl` to download and execute the exploit.

![image](https://github.com/user-attachments/assets/ff1311d6-e643-467b-a082-36eda3fe6d90)

## Summary

In this CTF challenge, we demonstrated the following skills:

- Network discovery using `netdiscover`
- Service enumeration with `nmap`
- Exploiting **Path Traversal** to read sensitive system files
- Extracting and attempting to crack hashed passwords
- Identifying and using public exploits for known vulnerable services (MiniServ/Webmin)
- Uploading and executing exploits using `python -m http.server`

This box emphasized the importance of manual enumeration and understanding common web vulnerabilities. It also highlights how outdated services can still be found in real-world systems, making patch management critical for system administrators.
