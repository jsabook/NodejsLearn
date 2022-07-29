# 分布式登录系统

这里通过jwt来实现分布式系统登录。通过前面的[jwt认证文章](Jwt.md)我们可以指导JWT协议可以保证用户数据的完整性，因此我们这里可以用这个架构来实现登录验证。

下面的架构图讲述了如何利用JWT来进行登录验证。

![罗氏minigram2-JWT认证.drawio (1)](https://raw.githubusercontent.com/jsabook/githubPic/main/myblog/%E7%BD%97%E6%B0%8Fminigram2-JWT%E8%AE%A4%E8%AF%81.drawio%20(1).svg)

**流程介绍**

- 在用户注册阶段，用户发送用户名和密码给网站，同时网站存储用户名和密码，将用户信息发送给KMS凭证最后返回JWT。
- 在用户登录阶段，用户发送用户名和密码给网站，同时网站判定用户名和密码是否相同，如果相同将用户信息发送给KMS服务器生成JWT凭证，将JWT凭证返回。
- 在用户鉴权阶段，用户发送JWT给网站，网站向KMS服务器申请e、n 根据这两项生成公钥，根据公钥校验校验JWT是否被修改，如果校验成功则返回资源。

## 系统架构

按照JWT登录流程，设置了如下系统架构流程。

![罗氏minigram2-站点.drawio-3](https://raw.githubusercontent.com/jsabook/githubPic/main/myblog/%E7%BD%97%E6%B0%8Fminigram2-%E7%AB%99%E7%82%B9.drawio-3.svg)

通过这种方式，可以完成基于JWT的分布式登录。