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
To start off enumerating the website we use ferobuster to find hidden directories.
```
./feroxbuster -w raft-large-directories.txt --url https://10.10.11.122 -k --silent
https://10.10.11.122/
https://10.10.11.122/login
https://10.10.11.122/assets => /assets/
https://10.10.11.122/Login
https://10.10.11.122/assets/js => /assets/js/
https://10.10.11.122/assets/css => /assets/css/
https://10.10.11.122/assets/images => /assets/images/
https://10.10.11.122/privacy
https://10.10.11.122/signup
https://10.10.11.122/terms
https://10.10.11.122/Assets => /Assets/
https://10.10.11.122/Assets/css => /Assets/css/
https://10.10.11.122/Assets/images => /Assets/images/
https://10.10.11.122/Assets/js => /Assets/js/
https://10.10.11.122/Privacy
https://10.10.11.122/Terms
https://10.10.11.122/Signup
```
Going through all the directories I don't find anything interesting, so next we enumerate vhosts and we find this.
```
ffuf -w subdomains-top1million-110000.txt -u https://nunchucks.htb -H "Host: FUZZ.nunchucks.htb" -fs 30589

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : https://nunchucks.htb
 :: Wordlist         : FUZZ: subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.nunchucks.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response size: 30589
________________________________________________

store                   [Status: 200, Size: 4029, Words: 1053, Lines: 102]
```
We find a sub domain at store.nunchucks.htb
