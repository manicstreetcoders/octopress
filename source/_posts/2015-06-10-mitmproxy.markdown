---
layout: post
title: "mitmproxy"
date: 2015-06-10 14:47:29 +0900
comments: true
categories: 
---

### Explicit HTTP

Explicity proxy 로 설정하는 것이 제일 안정적.

Client 는 proxy 에게 다음과 같은 proxy protocol 을 돌린다. RFC 2068.

```
GET http://www.naver.com/index.html HTTP/1.1
```

### Explicit HTTPS

```
CONNECT example.com:443 HTTP/1.1
```

Server 인증서를 위조해야하기때문에, mitmproxy 를 디바이스에다가 trusted CA certificate 으로 등록해야한다. 

```
CONNECT 10.1.1.1:443 HTTP/1.1
```

와 같이 숫자로 오는 경우도 있기때문에, `upstream certificate sniffing` 을 수행한다. 
Client 로부터 `CONNECT` 를 받으면, 진짜 서버에 접속해서 SSL 핸드쉐이크를 돌려서, 서버의 인증서를 가져온다.
여기서 CN 을 추출해, 이 CN 과 동일한 가짜 인증서를 만들고, mitmproxy CA 인증서로 서명한다.
