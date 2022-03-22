Start enumerating with rustscan 2.0.0
```
rustscan -a 10.10.11.122
```
Open Ports
```
Open 10.10.11.122:22
Open 10.10.11.122:80
Open 10.10.11.122:443
```
NMAP Scan for services and versions
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
Web server redirects to port 443 when trying to acess port 80
So just start enumerating with ferobuster to find hidden directories on port 443
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
Nothing interesting so I fuzz vhosts
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
Found a sub domain at store.nunchucks.htb

Web server has ssti vulnverability in "email" parameter
```
Requests

{"email":"{{7*7}}@htb.org"}

Response

{"response":"You will receive updates on the following email address: 49@htb.org."}
```
Request Header
```
POST /api/submit HTTP/1.1
Host: store.nunchucks.htb
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://store.nunchucks.htb/
Content-Type: application/json
Origin: https://store.nunchucks.htb
Content-Length: 27
DNT: 1
Connection: keep-alive
Cookie: _csrf=Tq_qR2QxRfBNVi5JncWUOntN
Sec-GPC: 1
```


