---
layout: post
title: "Drive-by-Download: Manager's Perpective"
date: 2013-09-13 13:22
comments: true
categories: 
---

Drive-by-Download 를 수행하는 팀을 구성한다면 리쿠루팅은 어떤 뽑아야 하고, 성과 평가는
어떻게 해야 할까? 하는 관점에서 적어보았다.

==========

#### Traffic Acquisition Engineer

트래픽이 많은 사이트를 해킹하여, 웹 페이지에 iframe 태그등을 삽입하여, 악성 웹사이트로 트래픽을 돌려야 한다. __얼마나 많은 Traffic 을 가져오냐가 KPI__

1. TCP/IP, Linux and web frameworks 에 대한 지식
* Linux, Windows, DNS and routing
* Apache, PHP, WordPress, MySQL, Rails and Django.
2. 웹 해킹 경험
* SQL Injection, XSS


#### Exploit Engineer

Browser, Java, Flash 등의 취약점을 찾아내어 쉘코드를 수행시킬 수 있는 수준까지 코드를 개발해야 한다. __Acquired 된 트래픽 대비 몇 % 를 감염시키냐가 KPI. 10~20% 가 업계 표준__

* x86 Assembly/Disassembly
* Strong knowledge of Windows memory management
* JavaScript / DOM 
* 알려진 CVE 를 exploit 할 수 있는 코드 작성 능력. 
* Browser, ActiveX control Fuzzer 개발 능력


#### Payload Engineer

Exploit 가 성공하는 순간 실행될 Payload 들을 개발해야 한다. 실행화일을 다운로드하여 실행한다던가, 다운로드한 수행코드를 기존 프로세스의 메모리에 집어넣어 돌린다던가 하는 등의 다양한 Payload 를 개발해야 한다. Payload 에는 Exploit 에 들어갈 쉘코드와 1st/2nd stage Dropper 가 포함된다. __Anti-virus 나 IDS 에 걸리지 않고 다양한 네트워크 환경 (i.e. proxy) 에서 동작하는 Payload 개발이 KPI__

* Windows API (Process, Memory, Thread, API Hooking, PE format)
* Visual C++/x86 Assembly


#### Rootkit Engineer

제작된 다양한 소프트웨어에 rootkit 기능을 추가한다. 

* Proficient in Windows SDK & internals
* Experience in writing usermod & kernelmod rootkits


#### Undetectability Engineer

JavaScript 난독화, Payload 난독화, 서버 PHP 난독화, 실행화일 난독화를 수행하는 Tool 을 제작하거나 선정한다. 빌드때마다, 3~40 개의 AV 에 직접 테스트하여 탐지되지 않을 것을 보증해야 한다. __Tool 을 거쳐 작전에 투입되는 화일들이 AV 에 걸리지 않는 것이 KPI__

1. JavaScript and x86 assembly 에 대한 지식
2. 다양한 JavaScript runtime (v8, spidermonkey, rhino) 에 대한 지식
3. Lexer / Paser 제작 경험
* ANTLR, BISON 지식이 있으면 플러스
4. AV /IDS Evasion 테크닉
* 기본적인 암호 지식
* 난독화


#### C&C / Botnet Engineer

C&C 서버를 제작한다. Massive 한 좀비 클라이언트들을 관리할 수 있는 안정적이고 스케일러블한 서버와 API 를 개발해야 한다. 

* 대용량의 웹 서버 / Socket 서버 프로그래밍
* PHP 프로그래밍
* P2P 프로토콜에 대한 지식
* Twitter 나 DNS 등의 다양한 통신 채널에 대한 API 활용 능력


#### Remote Administration Tool Engineer

피해자의 PC 에 설치되어 키스트로크나 화면 감시등을 할 RDP/VnC 와 같은 리모트 툴을 개발해야 한다. Exploit Engineer 와 협력하여 High value target 에 대한 작전을 수행할 때 사용할 툴 개발을 총괄. Spear phishing 에 사용될 무기화된 PDF/DOC 등을 만드는 일도 겸임한다. Rookit Engineer 와 협력하여 개발된 리모트 툴에 rookit 기능을 추가한다.

* Windows 프로그래밍에 능숙 (UI 포함)


#### Monetization Manager

돈을 버는 것을 총괄하는 일종의 프로덕트 매니저. 엔지니어들을 총괄 관리하여 게임/뱅킹등 버티컬에 특화된 악성 소프트웨어를 제작할 책임이 있다. 메모리에서 정보를 꺼낸다던가, 인증을 우회하여 결제를 일으키는 기술, 광고 사기 기술들을 연구해야 한다. 파밍 사이트 셋업도  Monetization Manager 의 책임이다. __전체 매출을 책임지며, 특히 감염된 PC 대당 매출을 얼마나 끌어올리는가가 KPI__

* Social Engineering 이 강해야 함
* People skill
* 지불 결제, 가상화폐, 아이템 거래, 광고네트워크(i.e. Ad-sense), 상품권 유통, CPC 광고등에 대한 지식과 경험
* PKI, challenge-response, 2 factor / 2 channel 인증에 대한 지식
* 통장/mule/courier 관리.
* 가상 재화를 현금화 할 수 있는 채널/네트워크 보유


#### Network Operator

Traffic Acquisition Engineer 에게 신규로 접수된 호스트들을 전달받아, 공격 체인에 투입될 수백/수천개의 호스트 풀을 관리한다. Google safe-browsing 이나 Microsoft SmartScreen 등의 서비스에 블랙리스팅된 호스트는 체인에서 빼고 신규 호스트를 투입한다. C&C 호스트를 관리하고 C&C 호스트를 NIC/DNS 에 등록/관리하는것도 Network Operator 의 책임이다. 

* Linux/Windows Administration Skills
* DNS / Nameserver / Hosting 에 대한 경험

==========

JD 를 만들어 따져보니, 생각보다 다양한 스킬을 가진 사람들이 필요하다.

이런 팀을 운영하려면 인건비 + 경비가 일년에 20 억 정도가 들 듯 하다. 
