---
layout: post
title: "TLS Protocol (1)"
featured-img: TLS_Protocol
category: Background
---

와이어샤크로 이것 저것 패킷을 보면서 분석하던 중 TLS로 암호화 된 패킷을 복호화 할 수 없을까? 
라는 의문으로 시작하여 이번 기회에 깔끔하게 TLS를 정리해보자 라는 생각이 들었습니다.

1. TLS 통신을 이해하기 위한 기본 지식(암호화 방식, 인증)
2. TLS 동작원리
3. openssl

순서로 정리를 해보도록 하겠습니다.



# TLS
---
TLS는 Client와 Server 사이에 Secure Channel을 생성하여 전송되는 자료를 **암호화 하고, 중간에 변조되지 않고, 서로를 인증**할 수 있는 역할을 합니다. 

따라서 TLS의 동작원리를 이해하기 위해서는 **암호화 방식**, **인증**을 알고 있어야 합니다.


# TLS의 암호화 방식
---
TLS는 패킷을 암호화 하기 위해서 **대칭키 암호화, 공개키 암호화** 방법을 사용합니다.<br>
결론부터 말하면 대칭키 암호화 방식은 패킷을 주고 받을때, 공개키 암호화 방식은 키를 교환할 때 사용합니다.

## 대칭키 암호화 방법
---
대칭키 암호화 방법은 **동일한 KEY로 암호화, 복호화를 진행**하는 방법입니다.<br>
따라서 client, server 모두 동일한 KEY를 가지고 있어야 합니다.<br>
하지만 네트워크 상에서 **KEY를 전달하는 과정에 공격자가 KEY를 탈취할 경우 공격자가 암호화를 해제**할 수 있는 문제가 발생합니다.


일반적인 대칭키 암호화 방법은 다음과 같은 방식으로 진행됩니다.

<img src="{{site.url}}/assets/img/posts/Background/2019-03-15-symmetric-key.png">


1) Client에 있는 KEY를 Server와 공유<br>
2) Client는 KEY를 이용하여 데이터 암호화<br>
3) Client는 암호화된 데이터를 Server로 전송<br>
4) Server는 KEY를 이용하여 데이터 복호화<br>
5) Server는 KEY를 이용하여 데이터 암호화<br>
6) Server는 암호화된 데이터를 Client로 전송<br>
7) Client는 KEY를 이용하여 데이터 복호화<br>


대칭키 암호화 방법은 **암호화, 복호화를 빠르게 진행**할 수 있다는 장점이 있지만 <br>
1번과정 처럼 **KEY를 공유하는 방법에 문제**가 존재합니다.



## 공개키 암호화 방법
---

공개키 방식은 한쌍의 키(A_KEY, B_KEYB)를 생성하여 **A_KEY로 암호화를 하면 B_KEY로 복호화** 할 수 있고, **B_KEY로 암호화하면 A_KEY로 복호화**할 수 있는 특징을 가지고 있습니다.
각각의 키는 **공개키, 개인키**로 지정하여 암호화, 복호화를 진행합니다.


일반적인 공개키 암호화 방법은 다음과 같은 방식으로 진행됩니다.

<img src="{{site.url}}/assets/img/posts/Background/2019-03-15-public key.png">

1) Client에 있는 Public Key를 Server에 공유<br>
2) Server에 있는 Public Key를 Client에 공유<br>
3) Client는 Server에 Data를 전송하기 위하여 Server Public Key로 암호화<br>
4) Client는 암호화 된 데이터를 Server에 전송<br>
5) Server는 Server의 Private Key를 이용하여 데이터 복호화<br>
6) Server는 Client에 Data를 전송하기 위하여 Client Public Key로 암호화<br>
7) Server는 암호화 된 데이터를 Client에 전송<br>
8) Client는 Client의 Private Key를 이용하여 데이터 복호화<br>


공개키 암호화 방법은 **Private Key**만 보관을 잘 하면 대칭키 암호화 방식에서 문제가 되었던 **KEY 교환 문제를 해결할 수 있지만 암호화, 복호화의 속도가 오래걸린다**는 단점이 있습니다.


- TLS 통신에서는 각각 암호화 방법의 장단점을 이용합니다. **패킷과 같이 큰 데이터 전송은 대칭키 암호화 방식**을 이용,
대칭키 암호화 방식에서 사용되는 **KEY는 공개키 암호화 방식**을 통해서 Client, Server가 공유하게 됩니다.


# TLS의 인증 방식
--- 

인증은 자신이 받은 정보가 올바른 사람이 전송한 정보가 맞는지 확인하는 것입니다.

TLS 통신에서는 인증을 확인하기 위해서 CA(Root Certificate)에서 발급하는 인증서를 사용합니다.

## CA, 인증서
---

**인증서**는 클라이언트가 접속한 서버가 클라이언트가 의도한 서버가 맞는지를 보장하는 역할(인증)을 합니다.<br>
이러한 인증서를 제공하는 **신뢰할 수 있는 민간 기업들이 있는데 이런 기업들을 CA(Root Certificate)**라고 합니다.<br>
TLS를 통해서 암호화된 통신을 제공하려는 서비스는 **서비스의 정보, 서비스의 Public Key**와 같은 정보를 CA에 전달하고<br>
CA는 이를 적절하게 검증하고, 검증이 완료되었다면 **CA는 자신의 Private Key로 해당 내용을 암호화**하여 인증서를 만들어 서버에 전달합니다.

따라서 **인증서**에는 다음과 같은 정보를 포함하게 됩니다.

- 서비스의 정보(인증서를 발급한 CA, 서비스의 도메인 등등)
- 서비스의 공개키(서버 측 공개키, 공개키의 암호화 방법)

**CA들의 Public Key** 리스트는 브라우저(Client)에 탑재 되어 있습니다.

Client와 Server가 TLS 통신 시 Server는 Client에게 **Server 인증서를 전달**합니다.
Server의 인증서는 **CA의 Private Key로 암호화** 되어 있으므로 Client의 브라우저에 있는 **CA의 Public Key로 복호화**를 성공한다면
해당 인증서는 CA에 의해 검증되어 있음을 확인할 수 있고 **인증**이 완료됩니다.


정리하자면 **TLS의 인증 방법**은 다음과 같은 방식으로 진행됩니다.

<img src="{{site.url}}/assets/img/posts/Background/2019-03-15-certification.png">

1) Server는 서버의 정보, 서버의 Public Key를 CA에 전송<br>
2) CA는 Server를 검증하고 서버의 정보, 서버의 Public Key를 CA의 Private Key로 암호화 하여 Server에게 인증서(Certificate)를 제공<br>
3) CA는 CA Public Key를 브라우저에게 제공<br>
4) Client는 Server에게 통신 요청<br>
5) Server는 인증서를 Client에게 전송<br>
6) Client의 브라우저는 인증서를 발급한 CA가 브라우저에 내장된 CA의 리스트에 포함되어 있는지 확인, 포함되어 있으면 CA의 공개키를 이용해서 인증서를 복호화<br>



### 끄읕
---
TLS 통신을 이해하기 위한 기본적인 배경 지식을 정리해봤습니다.<br>
다음 포스팅에서는 TLS의 동작원리에 대해서 정리하겠습니다.

## 참고
---

[HTTPS와 SSL 인증서](https://opentutorials.org/course/228/4894)<br>
[도난당한 패스워드:한국 인터넷에서 살아 남는 법](https://minix.tistory.com/520)
