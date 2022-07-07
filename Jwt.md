## 概览

JWT，全称JSON Web Token，是一种包含信息的Token，相较于普通的Token，唯一多的内容是：包含部分信息。与JWT相关的协议比较简单，但数量较多，本文只是对此加以总结。

## 组成

JWT包含三部分，头部、负载、签名。

**头部**

JWT的头部承载两个信息：

- 声明类型，这里是JWT

- 声明加密的算法

下面就是JWT头部的一个样例

```
{
  'typ': 'JWT',
  'alg': 'HS256'
}
```

然后将头部进行Base64编码（该编码是可以对称解码的），构成了第一部分。

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```

**载荷**

载荷就是存放有效信息的地方。定义细节如下：

- iss：令牌颁发者。表示该令牌由谁创建，该声明是一个字符串 
- sub:  Subject Identifier，iss提供的终端用户的标识，在iss范围内唯一，最长为255个ASCII个字符，区分大小写 
- aud：Audience(s)，令牌的受众，分大小写的字符串数组 
- exp：Expiration time，令牌的过期时间戳。超过此时间的token会作废， 该声明是一个整数，是1970年1月1日以来的秒数 
- iat: 令牌的颁发时间，该声明是一个整数，是1970年1月1日以来的秒数 
- jti: 令牌的唯一标识，该声明的值在令牌颁发者创建的每一个令牌中都是唯一的，为了防止冲突，它通常是一个密码学随机值。这个值相当于向结构化令牌中加入了一个攻击者无法获得的随机熵组件，有利于防止令牌猜测攻击和重放攻击。

也可以新增用户系统需要使用的自定义字段，比如下面的例子添加了`name` 用户昵称：

```
{
    "sub": "1",
    "iss": "http://localhost:8000/auth/login",
    "iat": 1451888119,
    "exp": 1454516119,
    "nbf": 1451888119,
    "jti": "37c107e4609ddbcc9c096ea5ee76c667",
    "name":Jeff
}
```

然后将其进行Base64编码，得到Jwt的第二部分：

```
eyJzdWIiOiAiMSIsImlzcyI6ICJodHRwOi8vbG9jYWxob3N0OjgwMDAvYXV0aC9sb2dpbiIsImlhdCI6IDE0NTE4ODgxMTksImV4cCI6IDE0NTQ1MTYxMTksICJuYmYiOiAxNDUxODg4MTE5LCJqdGkiOiAiMzdjMTA3ZTQ2MDlkZGJjYzljMDk2ZWE1ZWU3NmM2NjciLCJuYW1lIjoiSmVmZiJ9
```

**签名**

这个部分需要Base64编码后的Header和Base64编码后的Payload使用 `.` 连接组成的字符串，然后通过Header中声明的加密方式进行加密（`$secret` 表示用户的私钥），然后就构成了jwt的第三部分。这部分是JWT的核心。

```javascript
// javascript
let encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload);
let signature = HMACSHA256(encodedString, '$secret');
```

这里的$secret是密钥，因此只有通过密钥才能生成相同的签名。

**验证机制**

JWT的的签名机制保证了非人为的非授权篡改，保护的信息的安全性，验证的时候只要将头部和负载进行重新base64再和JWT原本的签名进行比较如果是相等的，那么数据的完整性就得到了保证。如果不相等，就可以得知数据的负载或者头部被人非授权篡改了。

```javascript
let JWTCode = "<JWT Code>"
let header =JWTCode.split('.')[0]
let playload = JWTCode.split('.')[1]
let signature1 = JWTCode.split('.')[2]

let encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload);
let signature2 = HMACSHA256(encodedString, '$secret');
if (signature1===signature2){
    console.log("验证成功")
} else {
    console.log("验证失败")
}
```

因此只有通过密钥才能够进行重新签名。因此必须要保证密钥的存储的安全性，密钥不能被泄露。
## JWT的几个特点

- JWT 默认是不加密，不能将秘密数据写入 JWT。
- JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。JWT的安全特3. JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
- JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。
- 为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用HTTPS 协议传输。

