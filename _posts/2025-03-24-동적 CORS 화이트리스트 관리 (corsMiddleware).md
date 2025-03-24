---
layout: post
title:  "동적 CORS 화이트리스트 관리"
date:   2025-03-24 11:00:00 +0900
category: [node.js]
tags: [node.js, cors, express]
lastmod : 2025-03-24 11:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---
> cors는 응답헤더에 client 서버 도메인을 허용한다는 내용이 없으면 발생한다.<br>
> 즉, response 헤더에 client 도메인을 포함해주면 해결된다.<br>
> `res.setHeader('Access-Control-Allow-origin', '허용할 도메인');`

<br>
<br>

**Node.js에서 Express를 사용하여 CORS를 동적으로 허용하는 방법**

```bash
npm install cors
```

```javascript
const cors = require('cors');
const whitelist = ['https://example.com', 'https://www.otherdomain.com']; // 허용된 도메인 목록

const corsMiddleware = (req, res, next) => {
  const origin = req.headers.origin;

  if (whitelist.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin);
  }

  next();
};

```
<br><br>


**전역으로 적용하기**
```javascript
app.use(corsMiddleware);
```
<br>

**특정 API만 적용하기**
```javascript
app.get('/api/data', corsMiddleware, (req, res) => {
  // ... API 로직
});
```

<br>

**추가 고려 사항:**
- **허용 메서드:** `corsMiddleware`의 options에 `methods`를 추가하여 허용되는 HTTP 메서드를 지정할 수 있습니다. (예: `methods: ['GET', 'POST']`)
- **허용 헤더:** `corsMiddleware`의 options에 `allowedHeaders`를 추가하여 허용되는 HTTP 헤더를 지정할 수 있습니다.
- **자격 증명:** `corsMiddleware`의 options에 `credentials: true`를 추가하여 쿠키와 같은 자격 증명을 허용할 수 있습니다.

<br>

**보안:**
- 화이트리스트에 신뢰할 수 있는 도메인만 포함하도록 주의하세요.
- 필요하지 않은 경우 `credentials` 옵션을 사용하지 마세요.
