To start our enumerating process we start with a quick scan with rustscan 2.0.0
```
rustscan -a 10.10.11.122
```
Once the scan finishes we get back 3 ports, port 22, 80, and 443.
```
Open 10.10.11.122:22
Open 10.10.11.122:80
Open 10.10.11.122:443
```
Just looking at these port I could assume by default port 22 is an ssh server, port 80 and 443 are web servers. I then run an nmap scan to confirm my theory and these are the results.
```
sudo nmap -sV -p 80,443,22 10.10.11.122
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-22 15:49 GMT
Nmap scan report for 10.10.11.122
Host is up (0.0046s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http     nginx 1.18.0 (Ubuntu)
443/tcp open  ssl/http nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:li
```
