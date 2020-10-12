# 喬叔教 Elastic - 20 - Elastic Cloud 比免費版還多的功能 (5/6) - Elasticsearch Token Service

**Elastic Cloud 比免費版還多的功能** 系列文章索引

- [(1/6) Elastic Stack 的方案比較與銷售方式](https://ithelp.ithome.com.tw/articles/10247538)

- [(2/6) Centralized Beats Management](https://ithelp.ithome.com.tw/articles/10248028)

- [(3/6) Centralized Pipeline Management](https://ithelp.ithome.com.tw/articles/10248584)

- [(4/6) Watcher](https://ithelp.ithome.com.tw/articles/10248884)

- [(5/6) Elasticsearch Token Service](https://ithelp.ithome.com.tw/articles/10249181)

- [(6/6) Multi-stack monitoring & Automatic stack issue alerts](https://ithelp.ithome.com.tw/articles/10249630)

---

## 前言

Elasticsearch 的 X-Pack Security 機制中，Token Based Authentication services 包含了兩種方式， `token-service` 與 `api-key-service` ，而  `api-key-service` 是 Basic License 就能使用的服務，不過 `token-service` 是必需要 Gold License 或是 Elastic Cloud service Standard 以上的授權才能使用的功能，這篇文章主要會介紹 `token-service` 的用法。

### 進入此章節的先備知識

- OAuth2 的運作方式。(可參考這篇：[簡單易懂的 OAuth 2.0](https://speakerdeck.com/chitsaou/jian-dan-yi-dong-de-oauth-2-dot-0))

### 此章節的重點學習

- 如何使用 Elasticsearch Token Service，也就是 Get token API。

---

## Elasticsearch Token Service

進到這篇文章時，已經假設讀者了解 OAuth2 的運作方式、也知道 OAuth2 的好處，而 Elastic Stack 在 X-Pack Security 的套件中，是以 `token-service` 的機制來實作 OAuth2 的 token 核發與更新機制。

### 基本介紹

在 Elastic Stack 中 `token-service` 的運作，主要就是以 [get token API](https://www.elastic.co/guide/en/elasticsearch/reference/master/security-api-get-token.html) 來產生 `access token` 和 `refresh token` ，這個 `access token` 是個短時效性的 token，預設是 20分鐘 過期，最長可以設到 1小時 過期，而核發 `access token` 時會同時提供一個 `refresh token` ，這個 `refresh token` 是當 `access token` 過期時，換發新的 `access token` 用的，預設是 24小時 有效。

### 啟用 Token Service

Elasticsearch 的架設上，如果有開啟 TLS (HTTPS) 的話，預設就會啟用 `token-service` ，或是可以手動在 `elasticsearch.yml` 的 config 中開啟：

```
xpack.security.authc.token.enabled: true
```

> 這邊要注意， 因為安全性的考量， Elasticsearch 在 production 環境提供的 bootstrap 啟動檢查時，會特別檢查如果有開啟 `token-service` 的話，必須要啟用 TLS ，否則 token 在不安全的傳輸過程中，是非常危險的。

如先前提到的，這個 `token-service` 必須是

- **Gold License** Subscription
- Elastic Cloud service **Standard** version

這兩種或是以上的授權等級，才能使用這項服務，如果是自架的 **Basic License** ，在使用 get token API 時，會得到以下的錯誤訊息：

![self-managed elasticsearch - basic license](https://i.imgur.com/byuZwuO.png)



### Get Token API

get token API 提供四種的 `grant_type`，分別的使用情境如下：

- `client_credentials`: 這是實作了 **OAuth2** 中的 **Client Credentials Grant** 的機制，主要是用在 machine to machine 的使用情境中，並不是提供給一般使用者操作的情境使用的，而簽發這個 token 時，會依照本來的 credential 的權限，產生一組 token 並付予完全一樣的權限，而這個機制只會提供 `access token` 不會提供 `refresh token`。
- `_kerberos`: 這是實作 **SPNEGO Kerberos** 機制，要使用的話，需要在 Elasticsearch 中設定好 [Kerberos realm 的配置](https://www.elastic.co/guide/en/elasticsearch/reference/current/kerberos-realm.html)。
- `password`: 這是實作 **Oauth2** 中的 **Resource Owner Password Credentials Grant** 的機制，這個機制是用在某一個已授權的用戶代表另一個授權的用戶來產生 `access token`，白話一點的說法就是，你要 call 這個 get token API 必須是已授權的某個使用者，但你帶入的 `username` 和 `password` 會是另一個人的帳號，並且是為了這個帳號來產生適合他權限的 `access token`，這個機制產生的 `access token` 會同時包含 `refresh token`，適合長時間的使用。
- `refresh_token`: 這是使用 `refresh_token` 來取的新的 `access token` 及 `refresh_token` 的時候所使用的。

以下有幾個例子：

#### Grant Type: `client_credentials`

這時因為在 call 這個 get token API 時，本身就會是以登入的身份 (或是帶某個 token 在 header 中)，所以只要帶這個 `grant_type` ，就會直接以當下使用者的身份權限，來產生出 `access token`。

```
POST /_security/oauth2/token
{
  "grant_type" : "client_credentials"
}
```

以下是回傳的結果，包含了 `access_token` 以及過期的時間，這種 `grant_type` 是不會有 `refresh_token` 的。

```
{
  "access_token" : "dGhpcyBpcyBub3QgYSByZWFsIHRva2VuIGJ1dCBpdCBpcyBvbmx5IHRlc3QgZGF0YS4gZG8gbm90IHRyeSB0byByZWFkIHRva2VuIQ==",
  "type" : "Bearer",
  "expires_in" : 1200
}
```

取得這個 `access token` 後，要使用的時候，在 HTTP Authroization 中帶入這個 `Bearer` 的 `token` 即可：

```
curl -H "Authorization: Bearer dGhpcyBpcyBub3QgYSByZWFsIHRva2VuIGJ1dCBpdCBpcyBvbmx5IHRlc3QgZGF0YS4gZG8gbm90IHRyeSB0byByZWFkIHRva2VuIQ==" http://localhost:9200/_cluster/health
```

#### Grant Type: `password`

下面是使用 `password` grant type 的例子，需帶入 `username` 與 `pasword`：

```
POST /_security/oauth2/token
{
  "grant_type" : "password",
  "username" : "test_admin",
  "password" : "x-pack-test-password"
}
```

回傳的結果如下：

```
{
  "access_token" : "dGhpcyBpcyBub3QgYSByZWFsIHRva2VuIGJ1dCBpdCBpcyBvbmx5IHRlc3QgZGF0YS4gZG8gbm90IHRyeSB0byByZWFkIHRva2VuIQ==",
  "type" : "Bearer",
  "expires_in" : 1200,
  "refresh_token": "vLBPvmAB6KvwvJZr27cS"
}
```

可以看到這種 grant type 包含了 `refresh_token`，這個 `refresh_token` 的有效時間是 24小時，在這時限內，若 `access token` 過期，都能使用 `refresh_token` 來取得新的 `access_token`。

#### Grant Type: `refresh_token`

當 `access token` 過期時，若還在 `refresh_token` 有效的 24小時 之內，可以直接使用這種方式來取得新的 `access token` 和 `refresh token`。

```
POST /_security/oauth2/token
{
  "grant_type": "refresh_token",
  "refresh_token": "vLBPvmAB6KvwvJZr27cS"
}
```

以下是回傳結果：

```
{
  "access_token" : "dGhpcyBpcyBub3QgYSByZWFsIHRva2VuIGJ1dCBpdCBpcyBvbmx5IHRlc3QgZGF0YS4gZG8gbm90IHRyeSB0byByZWFkIHRva2VuIQ==",
  "type" : "Bearer",
  "expires_in" : 1200,
  "refresh_token": "vLBPvmAB6KvwvJZr27cS"
}
```



### Token Service Settings

上面的例子的 `expires_in` 都是 `1200` ，如果我們要改變 `access token` 的有效時間，可以到 `elasticsearch.yml` 來改變這個時間的設定：

```
xpack.security.authc.token.timeout: 20m
```

預設是 `20m` ，最大可以設定的值是 1小時。



### Invalidate Token API

基本上 `access_token` 和 `refresh_token` 都有各自的過期時限，過期之後就無法再繼續使用。

如果要立即讓某個 token 失效、或某個使用者所有的 token 都失效時，就可以使用 `invalidate Token API`。

#### 讓某個 `access token` 立刻失效

```
DELETE /_security/oauth2/token
{
  "token" : "dGhpcyBpcyBub3QgYSByZWFsIHRva2VuIGJ1dCBpdCBpcyBvbmx5IHRlc3QgZGF0YS4gZG8gbm90IHRyeSB0byByZWFkIHRva2VuIQ=="
}
```

#### 讓某個 `refresh token` 立刻失效

```
DELETE /_security/oauth2/token
{
  "refresh_token" : "vLBPvmAB6KvwvJZr27cS"
}
```

#### 讓某個使用者的 tokens 立刻失效

```
DELETE /_security/oauth2/token
{
  "username" : "myuser"
}
```

依照官方的範例，回傳結果如下，會告訴我們總共 invalidate 多少個 tokens，以及若遇到錯誤各自的原因為何：

```
{
  "invalidated_tokens":9, 
  "previously_invalidated_tokens":15, 
  "error_count":2, 
  "error_details":[ 
    {
      "type":"exception",
      "reason":"Elasticsearch exception [type=exception, reason=foo]",
      "caused_by":{
        "type":"exception",
        "reason":"Elasticsearch exception [type=illegal_argument_exception, reason=bar]"
      }
    },
    {
      "type":"exception",
      "reason":"Elasticsearch exception [type=exception, reason=boo]",
      "caused_by":{
        "type":"exception",
        "reason":"Elasticsearch exception [type=illegal_argument_exception, reason=far]"
      }
    }
  ]
}
```

> **小插曲：**我在 Elastic Cloud service 試用這個 invlidate by username 時，發生以下的錯誤，看來是踩到某個 bug 了，看到 Github 上好像已經有類似的 issues，相信在不久後的 release 應該就會修掉了吧。
>
> ![image-20201005054248853](https://i.imgur.com/rj3dDzq.png)

## 結語

以上是 Elasticsearch Token Service 的介紹，在 Micro-services 的架構之下，若是將 Elasticsearch 當成其中一個微服務來使用時，透過 Token based 的方式來整合與使用其資源，是個蠻好的資源管理方式，這個機制只要在 Elastic Cloud 中就能直接使用，若是 Self-managed 的版本，會需要購買 **Gold License** 才能使用，或是要走向其他 [Open Source 的 solution](https://opendistro.github.io/for-elasticsearch-docs/docs/security/)了。



## 參考資料

- [官方文件 - Token-based authentcation services](https://www.elastic.co/guide/en/elasticsearch/reference/master/token-authentication-services.html)
- [官方文件 - Get token API](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-get-token.html)
- [官方文件 - Invalidate token API](https://www.elastic.co/guide/en/elasticsearch/reference/master/security-api-invalidate-token.html)
- [官方文件 - Security Settings - Token service settings](https://www.elastic.co/guide/en/elasticsearch/reference/master/security-settings.html#token-service-settings)

