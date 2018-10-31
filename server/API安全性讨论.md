#API安全性讨论
安全并单单不意味着加密解密，而是一致性（integrity），机密性（confidentiality）和可用性（availibility）。

##认证
身份认证包含很多种，有HTTP Basic，HTTP Digest，API KEY，Oauth，JWK等方式

##授权

##字典遍历攻击
比如/user/1123 可获取id=1123用户的信息，可对id进行url62或者uuid处理

##速率限制

##重放攻击