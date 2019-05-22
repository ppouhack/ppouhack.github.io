---
layout: post
title: "hack the box-popcorn"
featured-img: hack_the_box
category: Etc
---

**popcorn box**는 다음과 같은 순서로 root를 획득할 수 있습니다.

1. 웹 컨텐츠 스캐닝
2. 파일 업로드 우회
3. shell connection
4. 서버 정보 수집
4. MOTD File Tampering Privilege Escalation OR dirty COW Exploit

popcorn 문제에서는 파일을 업로드할 수 있는 디렉토리가 2개 존재합니다.<br>
`torrents` 디렉토리의 경우에는 파일 업로드가 불가능하지만 `upload` 디렉토리는 파일 업로드가 가능합니다.<br>
두개의 디렉토리를 비교하면서 어떠한 조건에서 파일 업로드가 가능한지 대해서 정리하겠습니다.


# 파일 업로드 우회
---
## 1) Directory
---
대부분의 경우 파일 업로드를 통해 SHELL을 올리고 실행하기 위해서는 업로드 되는 **디렉토리의 실행 권한이 존재**해야합니다.

```console
www-data@popcorn:/var/www/torrent$ ls -al
total 196
drwxr-xr-x 15 www-data www-data  4096 Mar 17  2017 .
drwxr-xr-x  4 www-data www-data  4096 May 18 12:34 ..
drwxr-xr-x  2 www-data www-data  4096 Jan 31  2010 PNG
drwxr-xr-x  4 www-data www-data  4096 Jun  3  2007 admin
-rw-r--r--  1 www-data www-data  1704 Jun  1  2007 browse.php
-rw-r--r--  1 www-data www-data  3042 Jun  3  2007 comment.php
-rw-r--r--  1 www-data www-data  6683 Apr 11  2017 config.php
...
drwxrwxrwx  2 www-data www-data  4096 May 18 12:20 torrents		;check
-rw-r--r--  1 www-data www-data  7221 Jun  3  2007 torrents.php
...
drwxrwxrwx  2 www-data www-data  4096 May 18 12:32 upload 		;check
-rw-r--r--  1 www-data www-data 15087 Mar 17  2017 upload.php
...
```

`torrents`(토렌트 파일 디렉토리), `upload`(토렌트 이미지 디렉토리) 모두 실행 권한이 존재합니다.

## 2) DirectoryIndex
---

**apache는 DirectoryIndex 지시어**를 사용하여 디렉토리에 접근하였을 때 기본적으로 읽을 파일을 지정할 수 있습니다. 
[https://httpd.apache.org/docs/2.4/ko/mod/mod_dir.html](https://httpd.apache.org/docs/2.4/ko/mod/mod_dir.html)

```console
root@popcorn:/# find / -name "*.conf" | xargs grep DirectoryIndex 2>/dev/null
/etc/apache2/mods-available/dir.conf:          
DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
/etc/apache2/mods-enabled/dir.conf:          
DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm

root@popcorn:/etc/apache2/mods-enabled# cat dir.conf
<IfModule mod_dir.c>
          DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm ; check
</IfModule>
```

### 2-1) torrents directory
---

`torrents`의 경우 **index.php에 PHP tag(\<? ?>)**이 존재합니다.

```console
www-data@popcorn:/var/www/torrent/torrents$ ls -al
total 300
drwxrwxrwx  2 www-data www-data   4096 May 18 12:48 .
drwxr-xr-x 15 www-data www-data   4096 Mar 17  2017 ..
-rw-r--r--  1 www-data www-data 235216 Mar 17  2017 723bc28f9b6f924cca68ccdff96b6190566ca6b4.btf
-rw-r--r--  1 www-data www-data  56111 May 18 12:20 c466035da5de7b04df065831e87ac368456e7fbe.btf
-rw-r--r--  1 www-data www-data      6 May 18 12:48 index.php 		; check

www-data@popcorn:/var/www/torrent/torrents$ cat index.php
<? 		
?> 		
```

따라서 `torrents`에 접근하였을 경우에 index.php 파일을 읽게 되고 빈 페이지를 보여주게 됩니다.<br>
**이런 경우에는 업로드 되는 파일명을 확인할 수 없으므로** 상대적으로 공격이 어려워집니다.

### 2-2) upload directory
---
하지만 `upload`에는 DirectoryIndex 지시어에 설정된 파일이 존재하지 않으므로 디렉토리 목록을 확인할 수 있습니다.

```console
www-data@popcorn:/var/www/torrent/upload$ ls -al
total 120
drwxrwxrwx  2 www-data www-data  4096 May 18 12:32 .
drwxr-xr-x 15 www-data www-data  4096 Mar 17  2017 ..
-rw-r--r--  1 www-data www-data   687 May 18 12:31 .rev.py
-rw-r--r--  1 www-data www-data 59294 Mar 17  2017 723bc28f9b6f924cca68ccdff96b6190566ca6b4.png
-rw-r--r--  1 www-data www-data    39 May 18 12:25 c466035da5de7b04df065831e87ac368456e7fbe.php
-rw-r--r--  1 www-data www-data  4663 May 18 12:23 c466035da5de7b04df065831e87ac368456e7fbe.png
-rw-r--r--  1 www-data www-data 33029 Jun  2  2007 noss.png
```

## 3) sourcecode
---

### 3-1) torrents sourcecode
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

### 3-2) upload sourcecode
---

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

### 4) RESULT
---
POPCORN 문제의 파일업로드를 통해 SHELL을 올리는 과정을 정리해보면 다음과 같습니다.

| torrents       | upload       |
|:------------:|:-------------:|
|디렉터리 실행 권한 있음|디렉터리 실행 권한 있음|
|디렉터리에 DirectoryIndex 파일 있음|디렉터리에 DirectoryIndex 파일 없음|
|상대적으로 업로드 파일에 대한 적절한 검증|업로드 파일에 대한 부적절한 검증|

## ETC Command
---

xclip을 사용하면 리눅스 터미널창에서 파일의 내용 또는 문자열 등을 파이프(\|)를 이용한 I/O 리다이렉션(redirection)을 통해 클립보드에 저장할 수 있습니다.

```console
root@kali:~/Desktop/popcorn/python-pty-shells# cat /usr/share/exploitdb/exploits/linux/local/14339.sh | xclip
```
---
시스템의 모든 정보를 출력합니다.

```bash
www-data@popcorn:/home/george/.cache$ uname -a
Linux popcorn 2.6.31-14-generic-pae #48-Ubuntu SMP Fri Oct 16 15:22:42 UTC 2009 i686 GNU/Linux
```
---
설치된 패키지 리스트 출력

```console
www-data@popcorn:/home/george/.cache$ dpkg -l | grep -i pam
ii  libpam-modules                      1.1.0-2ubuntu1                    Pluggable Authentication Modules for PAM
ii  libpam-runtime                      1.1.0-2ubuntu1                    Runtime support for the PAM library
ii  libpam0g                            1.1.0-2ubuntu1                    Pluggable Authentication Modules library
ii  python-pam                          0.4.2-12ubuntu3                   A Python interface to the PAM library
```

