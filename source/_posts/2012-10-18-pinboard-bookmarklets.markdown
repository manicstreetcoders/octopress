---
layout: post
title: "Pinboard bookmarklets"
date: 2012-10-18 14:45
comments: true
categories: 
---

[hi. i'm naveen.](http://naveenium.com) 에서 필요한 bookmarklets 를 발견했다.

위 blog 에는 del.icio.us 버전이 적혀있는데

`javascript:(function(){l=location.href;t=document.title;if(!t) t = prompt('title:');tags=prompt('tags:');location.href='https://api.del.icio.us/v1/posts/add?url='+escape(l)+'&description='+escape(t)+'&tags='+escape(tags);})()`

Pinboard 의 경우 delicious API 를 그대로 지원하므로, 아래와 같이
https://api.del.icio.us/v1 대신에 https://api.pinboard.in/v1/ 사용.

`javascript:(function(){l=location.href;t=document.title;if(!t) t = prompt('title:');tags=prompt('tags:');location.href='https://api.pinboard.in/v1/posts/add?url='+escape(l)+'&description='+escape(t)+'&tags='+escape(tags);})()`

크롬을 쓰기 때문에 핫키를 지정하기가 까다로운데, *기타 검색엔진* 에 등록하기로 했다. CMD-, 를 누르면 설정 페이지가 나오는데, 검색 -> 검색엔진 관리... 에서 "da / da / 그리고 위의 자바스크립코드" 를 입력하면 된다.

그 이후에,
CMD-L 을 누르고 da 라고 치면 tag 를 묻는 prompt 가 뜨고, 바로 pinboard 에 북마크가 추가된다.

P.S. Bookmarklets 가 Pinterest 처럼 여기저기서 이미지나 자료를 모아 큐레이션 하는 사이트에 잘 어울리는 기법인 것 같다.

### Further works:

* Pinterest 의 *Pin it* bookmarklets 의 코드도 한번 봐야겠다.
* [100 useful bookmarklets for better productivity](http://www.hongkiat.com/blog/100-useful-bookmarklets-for-better-productivity-ultimate-list/)
