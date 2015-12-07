---
layout: post
title: "Simple Introduction to OAuth"
date: 2012-12-14 10:34
comments: true
categories: 
---

### OAuth: user, client, and server

[Eran Hammer](http://hueniverse.com) 가 OAuth 2.0 을 떠난다고 한다. Web 이 아니라 Enterprise 의 요구에 끌려가면서 복잡해지고 거대해지고 등등의 사퇴의 변을 블로그에 올렸다.

* [Eran Hammer's blog](http://hueniverse.com) 에 OAuth 1.0 에 대한 좋은 글들이 많다.

* Heroku 에서 일하는 Schneems 의 [OAuth: A Tale of Two Servers](http://schneems.com/post/23164666348/oauth-a-tale-of-two-servers) 를 읽고 정리해본다.

우선 이 글에 나오는 Facebook 이나 instagram 은 이해를 돕기위한 예일뿐, 그들이 이런 프로토콜이나 flow 나 URL 을 사용한다는 것은 절대 아님.

* 리소스를 보관하고 있는 서버 (i.e. Facebook 의 Social Graph)
* OAuth Client (i.e. Facebook 의 Social Graph 를 import 하려는 instagram 서버)

우선 OAuth Client (i.e. instagram) 를 OAuth Provider (i.e. Facebook) 에
등록해야한다.

예를 들어 instagram 개발자는 Facebook 개발자 페이지에 가서

    client_id=instagram

를 등록하고 

    client_secret=bar

를 발급받는다.

실제 유저가 instagram 을 사용하는 경우를 상상해보자.

instagram 에서 

    GET facebook.com/oauth/authorize? 
    client_id=instagram
    callback=instagram.com/oauth_accept

를 facebook.com 에 쏜다.

facebook 은 이제 유저한테서 허가를 받아야겠다고 생각한다.

facebook 은 instagram 이 유저 XXX 의 어카운트를 접근하려고 한다...
허가해줄거냐 말거냐 물어보는 화면을 출력.

유저가 YES 하면 facebook 은 

    GET instagram.com/oauth_accept?
    code=foobahz

를 쏜다. 유저의 허락을 받았기에 `code=foobahz` 를 쏴주는 것이다.

이제 instagram 은 다음을 facebook.com 에 쏘고, access_token 을 받는다.
3 가지 정보를 같이 보냄으로써 유저의 허락도 받았고, legitimate 한 instagram 프로그램이라는 것을 입증하는 것이다. 
    
    POST facebook.com/oauth/access_token
    code=foobahz
    client_id=instgram
    client_secret=bar

    Result:

    { 'access_token': 'supersecret' }

이제 access_token 을 받았으니 사용자의 Social Graph 를 요청한다던가 하는 리퀘스트를 access_token 과 같이 보낼 수 있다. Facebook 은 이 access_token 이 authorized 된 것이고 자기가 발급해준 것이라는 것을 확인할 수 있기때문에, instagram 의 요청을 받아들인다.

    GET facebook.com/user/me/friends
    access_token=supersecret

### OAuth Provider

* http://api.babo.com/oauth/request_token
* http://api.babo.com/oauth/authorize
* http://api.babo.com/oauth/access_token

Further readings:

* [The sorry state of Python OAuth providers](http://pydanny.com/the-sorry-state-of-python-oauth-providers.html)
* [Apigee homepage](http://www.apigee.com)
