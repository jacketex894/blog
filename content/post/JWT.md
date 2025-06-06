+++
date = '2025-04-15T15:34:43+08:00'
draft = false
title = 'JWT Note'
categories = [
  "System Design"
]
+++

## JWT 是什麼
JWT 全名是 JSON Wenb Token，用來以 Json 的方式來傳遞資訊。
JWT 由三個部分組成，並以.做為分隔:

1. Header
  由以下兩個欄位組成，並且使用 Base64 進行編碼，要注意的是 Base64 編碼是可以被逆向解碼的。
     * alg : token 用來加密的演算法
     * typ : token 的類型
2. Payload
   存放要的傳遞資料，除了客製化資料外，也可以使用 JWT 所定義的變數名稱，Payload 一樣是經過 Base64 編碼，不適合用來存放機敏的資料。
   以下是一些常見變數，更詳細可以參考 [wiki](https://en.wikipedia.org/wiki/JSON_Web_Token)。
   ```
   iss(Issuer) : JWT 的簽發者
   sub(Subject) : 主題，可以放用戶id
   aud(Audience) : JWT 的接收者
   exp(Expiration Time) : JWT 的過期時間
   nbf(Not Before) : JWT 在此欄位的時間點，都是不可以使用的
   iat(Issued at) : JWT 的簽發時間
   jti(JWT ID) : JWT 的唯一身分標示
   ```
3. Signature
    用於保證資料未被傳改，由 header + . + payload + secret 加密後產生。
    secret 為一組字串，存放於伺服器端，如果 secret 外漏，使用者便可以自己產生 JWT

## 如何使用 JWT
1. 使用者登入後，Server 驗證成功後會產生一組 JWT。
2. Server 將 JWT 回傳給 Client，Client 會儲存 JWT ，可以存放在 localStorage 或 cookie。
3. 當 Client 要請求時，必須附上 JWT。
4. Server 收到後會去檢查 JWT Token 是否可用。

## 優缺點
### 優點
* JWT 可以存放在 Client 端， Server 不用維護 Session。
* 可以在不同的網域中使用，因為在前後端分離的話，前後端可能位於不同網域，cookie 無法跨網域使用，所以不能用 cookie 來做身分驗證。使用 JWT 可以在不需登入的情況下，串接多個不同網域的服務。

### 缺點
* JWT 無法由 Server 端主動撤銷。
* JWT 的內容並不是加密。

## Reference  
- https://kucw.io/blog/jwt/
- https://en.wikipedia.org/wiki/JSON_Web_Token
- https://medium.com/%E4%BC%81%E9%B5%9D%E4%B9%9F%E6%87%82%E7%A8%8B%E5%BC%8F%E8%A8%AD%E8%A8%88/jwt-json-web-token-%E5%8E%9F%E7%90%86%E4%BB%8B%E7%B4%B9-74abfafad7ba
- https://www.explainthis.io/zh-hant/swe/jwt
- https://medium.com/@smart_iceberg_goat_568/%E8%A7%80%E5%BF%B5%E7%AD%86%E8%A8%98-jwt-%E8%AA%8D%E8%AD%89%E6%A9%9F%E5%88%B6-5cb7e4e69736