---
layout: post
title: "PHP Configuration (1)"
featured-img: PHP
category: Dev
---

PHP 관련 오픈소스나 코드를 분석하는데 전체적인 개념보다는 그때 그때 찾아서 분석했기 때문에 개념이 없었습니다(?)<br>
이왕 이렇게 된거 PHP 관련된 책(성공적인 웹 프로그래밍 PHP와 MySQL,5판) 뚝딱 정리해보려고 합니다. <br>
개발공부를 하기 위해서는 환경구성을 예쁘게 해야하므로 다음 목록에 있는 것을 설치하는 것을 간단하게  정리하겠습니다.

1. XAMPP
2. phpstorm
3. xdebug

설치하는 것을 자세하게 보고 싶으면 아래 URL에 있는 가이드가 아주 잘 설명되어 있습니다.
[https://www.jetbrains.com/help/phpstorm/configuring-php-development-environment.html](https://www.jetbrains.com/help/phpstorm/configuring-php-development-environment.html)

# XAMPP
---

저는 새로운것, 신상을 좋아하기 떄문에 최신 버전의 XAMPP를 설치하였습니다.
[https://www.jetbrains.com/help/phpstorm/configuring-php-development-environment.html](https://www.apachefriends.org/index.html)


설치하는 과정은 별다른게 없기 때문에 default로 설정하여 설치하였습니다.

<img src="{{site.url}}/assets/img/posts/Dev/2019-04-18-XAMPP_Control_Panel.png">

# phpstorm 
---

php를 개발하는데 있어서 반드시 IDE를 사용해야 하는건 아니고 sublimenote, notepad++등을 이용해서 개발을 많이 한다고 들었습니다.<br>
하지만 저는 개발을 하는게 아니라 코드 보고 분석, 디버깅하는 것이 목표이므로 xdebug를 간단하게 설치할 수 있으며 UI가 예쁜 phpstorm을 설치하였습니다. 마침 대학생 인증을 하였기 때문에 무료로..

다음의 URI에서 phpstorm을 설치합니다.<br>
[https://www.jetbrains.com/phpstorm/](https://www.jetbrains.com/phpstorm/)


마찬가지로 뭐 별다른게 없기 때문에 default로 설정하여 설치하였습니다.


interpreter는 다음과 같이 설정하였습니다.

```no-highlight
file -> settings -> languages & framework -> php -> CLI interpreter -> ... -> +(CLI interpreter) 
-> Local Path to Interpreter -> General -> PHP executbale -> C:\xampp\php\php.exe (php 설치 경로)
```

Deployment는 다음과 같이 설정하였습니다.
```no-highlight
file -> settings -> Build, Execution, Deployment -> Deployment -> + -> Local or mounted folder
-> servername -> Type(Local or mounted folder) -> Folder(C:\xampp\htdocs)  
```


설치하는 것을 자세하게 보고 싶으면 아래 URL에 있는 가이드가 아주 잘 설명되어 있습니다.
[https://www.jetbrains.com/help/phpstorm/configuring-local-interpreter.html](https://www.jetbrains.com/help/phpstorm/configuring-local-interpreter.html)
[https://www.jetbrains.com/help/phpstorm/installing-an-amp-package.html#integrating-xampp](
https://www.jetbrains.com/help/phpstorm/installing-an-amp-package.html#integrating-xampp)

# xdebug
---
xdebug는 php, Compiler, Architecture	, Architecture에 맞춰서 설치해야합니다.<br>
해당 내용은 phpinfo()를 통해서 쉽게 확인할 수 있습니다.

| version | value |
|:------------:|:-------------|
|System|Windows NT DESKTOP-KNQ7C94 10.0 build 17134 (Windows 10) AMD64|
|Compiler|MSVC15 (Visual C++ 2017)|
|Architecture|x64|
|Thread Safety|enabled|
|PHP Version|7.3.4|

따라서 저는 `PHP 7.3 VC15 TS (64 bit)`를 설치하였습니다.

다음의 URI에서 xdebug를 설치합니다.<br>
[https://xdebug.org/download.php](https://xdebug.org/download.php)


다운받은 xdebug를 C:\xampp\php\ext 로 이동하고 C:\xampp\php\php.ini 파일을 다음과 같이 설정하였습니다.

```no-highlight
[XDebug]
zend_extension="C:\xampp\php\ext\php_xdebug-2.7.0-7.3-vc15-x86_64.dll"
xdebug.remote_enable=true
xdebug.remote_host=localhost 
xdebug.remote_port=9000 ; phpstorm에서 디버깅을 할 때 기본적으로 사용하는 포트
xdebug.remote_handler=dbgp 
xdebug.profiler_enable=1 
xdebug.profiler_output_dir="C:\xampp\tmp"
```
apache를 재시작하고 phpinfo()로 확인하면 xdebug 항목이 생성된 것을 확인할 수 있습니다.
<img src="{{site.url}}/assets/img/posts/Dev/2019-04-18-phpinfo.png">

phpstorm에서도 정상적으로 설치되었는지 확인할 수 있습니다.
```no-highlight
file -> settings -> languages & framework -> php -> CLI interpreter -> ... 
```
<img src="{{site.url}}/assets/img/posts/Dev/2019-04-18-xdebug_check.png">


설치하는 것을 자세하게 보고 싶으면 아래 URL에 있는 가이드가 아주 잘 설명되어 있습니다.
[https://www.jetbrains.com/help/phpstorm/configuring-xdebug.html](https://www.jetbrains.com/help/phpstorm/configuring-xdebug.html)

# xdebug helper
---
xdebug로 디버깅을 편하게 하도록 도움을 주는 크롬 확장프로그램입니다.<br>
사용법도 아주 간단한데 xdebug helper를 설치하고 chrome에서 아래와 같이 활성화합니다.

<img src="{{site.url}}/assets/img/posts/Dev/2019-04-18-chrome_xdebughelper.png">

phpstorm에서 다음 버튼을 활성화합니다.

<img src="{{site.url}}/assets/img/posts/Dev/2019-04-18-phpstorm_xdebughelper.png">

디버깅하고 싶은 부분에 BreakPoint를 지정하고 해당 코드가 호출이 될 경우 다음과 같이 디버깅을 할 수 있습니다.

<img src="{{site.url}}/assets/img/posts/Dev/2019-04-18-phpstorm_debugging.png">


설치하는 것을 자세하게 보고 싶으면 아래 URL에 아주 잘 설명되어 있습니다.<br>
[https://ericdraken.com/php-debugging-with-phpstorm-and-xdebug/](https://ericdraken.com/php-debugging-with-phpstorm-and-xdebug/)


### 끄읕
---
PHP 개발 환경을 구성해봤습니다.<br>
다음 포스팅에서는 본격적으로 PHP 예제 코드를 정리하겠습니다.

## 참고
---

[https://ericdraken.com/php-debugging-with-phpstorm-and-xdebug/](https://ericdraken.com/php-debugging-with-phpstorm-and-xdebug/)<br>
[https://www.jetbrains.com/help/phpstorm/configuring-php-development-environment.html](https://www.jetbrains.com/help/phpstorm/configuring-php-development-environment.html)