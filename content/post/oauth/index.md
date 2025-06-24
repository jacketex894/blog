+++
date = '2025-06-23T16:56:03+08:00'
draft = false
title = 'Oauth'
categories = [
  "System Design"
]
+++
# Oauth

Open Authorization 為一種開放標準，讓使用者授權第三方應用程式存取在其他服務(e.g. google) 上的資源，而不用將使用者名稱和密碼給第三方應用程式。

Oauth 讓使用者透過 token 來取得在其他服務上的資源，所以也能透過 Oauth 來驗證使用者在其他服務上的身分，並透過此身分來登入第三方應用程式。

但要注意的是 Oauth 的重點並不是身分驗證，而是授權第三方應用程式存取資源。

## Oauth 2.0

[RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749) 定義了以下角色

* resource owner : 資源擁有者。
* resource server : 受保護的資源所存放的server。
* client : 代表資源擁有者並在其授權下發出存取保護資源請求的應用程式。
* authorization server : 驗證伺服器，當伺服器驗證成功並獲得授權後，會向 client 發出 token。

{{< figure src="images/flow.png" title="flow" width="90%">}}

* Client -> Resource owner: Client 向 Resource owner 請求授權，授權請求可以直接由 Client 發給 Resource owner ，或是透過 Authorization server 作為中間人來發出請求給 Resource owner(e.g. redirect 到 Authorization server 提供的授權介面)。
* Resource owner --> Client : 資源擁有者同意授權許可。
* Client -> Authorization server : Client 向 Authorization server 要求 access token。
* Authorization server --> Client : Authorization server 在驗證授權許可通過後，會產生 access token 給 Client。
* Client -> Resource Server : Client 使用 access token 要求存取受保護的資源。
* Resource Server --> Client : Resource server 驗證過 access token 後回覆要求。 
[RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749)  也有定義了幾種授權模式。

## Authorization Code Grant
用於取得 access token 跟 refresh token。當使用透過 User agent (web browser)要使用 client 服務的時候，透過重新導向的方式導向 Authorization server 來進行授權。

{{< figure src="images/Authorization Code.png" title="flow" width="90%">}}

優點: access token 是在 server 端(client) 交換 token，所以不易被竊取 token。

## Implicit Grant



## 參考:
https://zh.wikipedia.org/zh-tw/%E5%BC%80%E6%94%BE%E6%8E%88%E6%9D%83
https://datatracker.ietf.org/doc/html/rfc6749
https://ithelp.ithome.com.tw/m/articles/10291817
https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2