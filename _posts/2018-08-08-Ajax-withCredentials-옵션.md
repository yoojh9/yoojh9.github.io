---
layout: post
title: "Ajax withCredentials 옵션"
tags: [ajax,CORS]
comments: true
---

## 1. withCredentials  
크로스 도메인의 API를 사용할 일이 생겨, 서버단에서 CORS 설정을 했는데도 쿠키에 있는 사용자 인증 정보가 넘어오지 않는 문제점이 발생했다.  
쿠키는 브라우저의 same-origin 정책을 따르기 때문에 같은 도메인에서만 cookie를 read/write 할 수 있다.  
이 문제를 해결 하기 위해 withCredentials = true로 설정하게 되면 Set-Cookie 헤더를 통해 원격 서버에 쿠키를 set 할 수 있게 된다.


---
#### 참고
[스택오버플로우](https://stackoverflow.com/questions/14462423/cross-domain-post-request-is-not-sending-cookie-ajax-jquery) <br/>
