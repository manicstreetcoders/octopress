---
layout: post
title: "API-centric Marketing"
date: 2012-05-12 22:20
comments: true
categories: 
---

@andrewchen 은 [Growth Hacker is the new VP Marketing](http://andrewchenblog.com/2012/04/27/how-to-be-a-growth-hacker-an-airbnbcraigslist-case-study/) 에서
AirBnB 와 Craigslist 와의 연동에 대해서 이야기한다.

놀라운 점은 공개된 API 를 통해서 연동한 것이 아니라,
웹을 스크레이핑하고 파싱하고, 폼을 대신 채워넣고 하는 방식으로
연동시킨것.

연동한 기능은

*"Airbnb 에 등록한 집을 Craigslist 에 쉽게 포스팅할 수 있게 
'Post to Craigslist' 메뉴를 제공하는 것"*

인데, 

Craigslist 의 사용자 풀에서 사용자를 끌어올 수 있는 
좋은 수단이라고 이야기한다.

기존의 마케터라면 이런 아이디어를 생각해내는 것은 차치하고,
가능한지도 몰랐을 것. 마케팅 마인드의 엔지니어가 필요하다고 이야기한다.

> Airbnb does just this, with a remarkable Craigslist integration.
> They've picked a platform with 10s of millions of users where relatively
> few automated tools exist, and have created a great
> experience to share your Airbnb listing. It's integrated simple and deeply
> into the product, and is one of the most impressive ad-hoc integrations
> I've seen in years. Certainly a traditional marketer would not have come up
> with this, or known it was even possible - instead it'd take a 
> marketing-minded engineer to dissect the product and build an integration
> this smooth.

<br />
뭔가 남이 해놓은 것을 리버스해서 구현하는 것은 쉽지만, 
이렇게 처음으로 만들어 내는 것은 정말 어려움.

> The impressive part is that this is done with no public Craigslist API!
> It turns out, you have to look closely and carefully at Craigslist
> in order to accomplish an integration like this.
> Note that it's 100X easier for me to reverse engineer something that's
> already working versus coming up with the reference implementation -
> and for this reason, I'm super impressed with this integration.

<br />
요즘에는 100M+ 이상의 유저를 가지고 있는 슈퍼 플랫폼들이 있기 때문에,
API-centric 한 마케팅이 중요하다고 이야기한다.

심지어 Airbnb 는 없는 API 도 만들어서 연동을 하는데.

> The stakes are huge because of "superplatforms" giving access to 100M+
> consumers.

> ...

> The fastest way to spread your product is by distributing it on a
> platform using APIs, not MBAs. Business development is now API-centric,
> not people-centric.

<br />
하지만 이 경우에는 공개된 API 가 아니었기 때문에 그 과정은 쉽지 않았을 것이라고 이야기한다.
웹을 스크레이핑한다거나... 입력 폼을 미리 채워넣는다거나 하는 작업들이 필요했을 것.

> The first thing you have to do is to look at how Craigslist allows users
> to post to the site.
> Without an API, you have to write a script that can scrape Craigslist and
> interact with its forms, to pre-fill all the information you want.

<br />
-----

<br />

Quiz: YouTube 의 성공에 \<EMBED\> API 가 기여한 바는?

1. 크다.
2. 작다.
3. 없다.

