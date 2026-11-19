# Brooklyn-Nine-Nine-Penetration-Test
A TryHackMe Brooklyn Nine-Nine penetration testing project featuring a detailed penetration test report and practical walkthrough covering reconnaissance, service enumeration, anonymous FTP, steganography, credential discovery, SSH access, and privilege escalation using Kali Linux.
# Brooklyn Nine-Nine – TryHackMe Penetration Testing Walkthrough

A practical penetration-testing walkthrough of the **Brooklyn Nine-Nine** room on TryHackMe. This project documents reconnaissance, service enumeration, anonymous FTP access, steganography, credential discovery, SSH access, and privilege escalation.

## 1. Lab Overview

| Item | Details |
|---|---|
| Platform | TryHackMe |
| Room | Brooklyn Nine-Nine |
| Difficulty | Easy |
| Target | Linux machine |
| Attacking machine | Kali Linux |
| Assessment date | September 2025 |

**Objective:** Identify exposed services, investigate potential weaknesses, gain an initial foothold, and demonstrate the impact of privilege misconfiguration.

> The target IP is assigned by TryHackMe and may change when the machine is restarted. Replace `<TARGET_IP>` with the IP displayed in your active room.

## 2. Reconnaissance – Nmap

I started by scanning the target to identify open ports, running services, and their versions.

```bash
nmap -sV -sC <TARGET_IP>
```

The scan identified three open TCP ports:

| Port | Service | Version |
|---|---|---|
| 21 | FTP | vsftpd 3.0.3 |
| 22 | SSH | OpenSSH 7.6p1 |
| 80 | HTTP | Apache httpd 2.4.29 |

The Nmap output also indicated that anonymous FTP login was permitted. This became the first area for further investigation.

## 3. Enumeration – Anonymous FTP

I connected to the FTP service:

```bash
ftp <TARGET_IP>
```

At the login prompt, I entered:

```text
Username: anonymous
```

After successfully logging in, I listed the available files and downloaded the note.

```bash
ls
get note_to_jake.txt
```

After exiting the FTP session, I read the downloaded file:

```bash
cat note_to_jake.txt
```

The note contained a message about a weak password and references to users Amy, Jake, and Holt.

**Finding:** The FTP server allowed unauthenticated users to access and download files. This exposed information that could assist further enumeration.

## 4. Web Enumeration – Discovering the Image

I visited the target's web server:

```text
http://<TARGET_IP>/
```

After inspecting the page source, I found a reference to an image named `brooklyn99.jpg`.

I downloaded the image:

```bash
wget http://<TARGET_IP>/brooklyn99.jpg
```

I inspected the downloaded file:

```bash
file brooklyn99.jpg
exiftool brooklyn99.jpg
```

The image and the page-source hint suggested investigating steganography.

## 5. Steganography – Extracting Hidden Information

I used Stegcracker with the RockYou wordlist to test the image's hidden-data passphrase.

```bash
stegcracker brooklyn99.jpg /usr/share/wordlists/rockyou.txt
```

The tool successfully recovered the passphrase.

Next, I used Steghide to extract the embedded file:

```bash
steghide extract -sf brooklyn99.jpg
```

I then inspected the extracted file:

```bash
cat note.txt
```

The file contained a password associated with the user Holt.

**Finding:** Sensitive information was concealed inside a publicly accessible image. The weak passphrase made the embedded information recoverable using a common password wordlist.

## 6. Initial Access – SSH

Using the recovered credentials, I connected to the target through SSH:

```bash
ssh holt@<TARGET_IP>
```

After authentication, I enumerated the user's home directory:

```bash
ls -al
```

I located and read the user flag:

```bash
cat user.txt
```

This confirmed successful access to the Holt user account.

## 7. Privilege Escalation – Sudo Misconfiguration

After obtaining SSH access, I checked the user's permitted sudo commands:

```bash
sudo -l
```

The output showed that Holt could execute Nano as root without entering a password:

```text
(ALL) NOPASSWD: /bin/nano
```

I used this permission to read the root-owned flag:

```bash
sudo nano /root/root.txt
```

The root flag was successfully accessed.

**Finding:** Allowing an unprivileged user to run an interactive editor as root creates a privilege-boundary weakness. An editor with elevated privileges may allow access to files that the user would otherwise be unable to read or modify.

## 8. Key Findings

| Finding | Security impact |
|---|---|
| Anonymous FTP access | Unauthenticated access to exposed files |
| Hidden information in a public image | Sensitive information disclosure |
| Weak passphrase and exposed credentials | Unauthorized account access |
| Excessive sudo permission for Nano | Access to root-owned resources |

## 9. Tools Used

- **Nmap:** Port scanning and service enumeration.
- **FTP client:** Anonymous FTP access and file retrieval.
- **wget:** Downloading the image from the web server.
- **ExifTool:** Image metadata inspection.
- **Stegcracker:** Testing the image's steganography passphrase.
- **Steghide:** Extracting embedded data.
- **SSH:** Remote access to the target.
- **Nano and sudo:** Testing the impact of excessive privileges.

## 10. Lessons Learned

This lab helped me practise a complete attack path, from initial reconnaissance to accessing a privileged file.

Key takeaways:

- Always investigate exposed services and anonymous access.
- Inspect web page source code and publicly accessible assets.
- Understand how steganography can conceal sensitive information.
- Never store credentials in accessible files or media.
- Review sudo permissions and apply the principle of least privilege.

## Disclaimer

This walkthrough documents activity performed in the authorised TryHackMe training environment. It is intended for educational purposes only. Do not use these techniques against systems without explicit permission.

Sensitive credentials, passwords, and challenge flags have intentionally been omitted from this public write-up.
