# 系统安全概要设计

## 主要内容
* 接口安全
* WEB前端安全
* 内部系统安全
* 数据存储安全
* 应用服务器安全


## 接口安全
接口是系统提供给用户的唯一入口，无论APP还是WEB都是通过接口使用系统功能，解决了接口的安全问题就解决了系统大部分的安全问题。   
青书学堂各个系统提供的接口可以分为`内网接口`和`外网接口`，`内网接口`外网不能访问，我们暂且认为内网环境是安全的，所以此处只讨论`外网接口`的安全性问题。

### 潜在安全问题
* authFree接口漏洞
* 弱口令漏洞
* 爆破破解密码
* 敏感信息泄漏  
  `比如短信验证码登陆时，接口不能返回说当前手机号尚未注册，否则会暴露出青书学堂所有用户的手机号；  
  再比如账号密码登陆时，不应准确提示'账号不存在'或提示'密码错误'，而应笼统的提示账号或密码错误`
* 越权攻击（水平越权、垂直越权）
* 重放攻击
* 中间人攻击
* 接口恶意频繁调用

### 方案
* 限制authFree接口
* 认证&授权
* 禁用弱口令
* 接口幂等性设计
* 接口入参校验  
* 防恶意频繁调用
* HTTPS

### 限制authFree接口
authFree接口即不需要认证&授权的接口，比如任何用户都可以访问青书学堂主页、学校主页，不需要任何授权，这类接口应该严格管控，具体原则：

* 明确划分哪些接口是不需要认证授权就可以访问的，尽量缩小这个范围
* authFree接口不能涉及敏感数据，如学生名单/学生成绩等
* 提供给外部系统调用的接口（比如与学校对接成绩），不能够authFree

### 认证&授权
青书学堂公开接口的用户可以分为系统内用户和系统外用户两类，针对这两类接口，其认证授权策略也有不同，具体思路如下：

#### 1、JWT Token
提供给青书学堂用户的接口，需要先通过登陆接口获取token后使用。token为JWT格式，其中payload内容大致如下：
```json5
{
  "client": "APP",
  "uId": 12312,
  "anonymous": false,
  "roles": [
    {
      "org": 12312,   // 用户所属组织，比如某学校、教师平台、云享课
      "subOrg": 123,  // 用户所属子组织，如某函授站、某个人教师平台、某云享课站点
      "uId": 1231424, // 用户在所属组织中的ID，如studentId, teacherId
      "roles": [2,4], // 用户角色
      "modules": [12,3,34,4,645,876] // 用户有权访问的功能模块
    }
  ]
}
```

##### 校验
1. 不同client的token相互独立，`APP`类型的token不能调用`WEB`接口，反之亦然
2. 不同环境的token相互独立，不能互用
3. server根据token中角色判断其权限范围，防止越权访问，比如A学校老师操作B学校资源、教学老师访问教务老师接口等

#### 2、SIGN
提供给第三方用户的接口，一律需要使用sign机制实现认证授权，具体实现思路为：  
1. 为每个第三方分配唯一的`APPID`和`APPSECRET`
2. server设置每个`APPID`允许调用的接口 
3. 第三方调用接口时，使用`APPSECRET`对请求内容进行`HmacSHA256`签名，待签名内容除业务参数外，还需带有`APPID`, `timestamp`

```http request
// 请求示例
GET https://api.qingshuxuetnag.com/open/v1/order/search?appId=2k320s0s&payStatus=1&timestamp=122392839239&userId=112&sign=lsakfjq4jr234j52134i51p24121
```
```JAVA
class SignHelper{
    
    public String sign(Request request){
      // 所有请求参数按字母顺序排列生成待签名字符串，如：
      // appId=2k320s0s&payStatus=1&timestamp=122392839239&userId=112
      String text = request.getTextToSign();
      return HMAC256(text, getAppSecret());
    }
}
```
##### 校验
```java
class RequestAuthFilter(){
  
    public boolean auth(Request request){
        if (!isSignValid(request)){
            return false;
        }

        List<Permission> permissions = getPermissionByAppId(request.getAppId());
        return hasPermission(request.getPath(), permissions);
    }
}
```

### 禁用弱口令
* 使用弱口令登陆时，server生成一个只能用于`修改密码`、`绑定手机号`、`获取个人信息`的token
* 适用该token，如果尚未绑定手机号，提示其绑定，绑定后提示修改密码；如果已绑定手机号，则直接提示修改密码
* 修改密码仅支持手机号验证码方式

### 接口幂等性设计
防止重放攻击，主要是新增数据类接口，比如：下单、创建用户、上传学习记录、交卷等接口

### 接口入参校验
* 参数格式校验：如字段是否为空、手机号格式、身份证格式等
* 参数业务校验：如ENUM是否合理、指定资源是否存在、是否违反业务规则等

### 防恶意频繁调用
不同接口根据其业务特点，能接受的调用频率是不同的，多频繁算恶意，应有不同的定义。在接口破坏力程度上，我们可以把接口分为以下几类：
1. authFree类接口
2. 查询类接口
3. 数据管理类接口
   `举例：注册用户、下单、点赞、发帖、修改成绩等接口`
4. 三方服务调用相关接口
   `举例：发送短息、人脸识别、CDN上传等`
5. 账号安全相关接口  
    `举例：登陆、更新密码、找回密码、身份信息绑定等`

以上五类，对恶意调用的容忍度应当是越来越低的。在这个原则的指导下，我们可以采用`IP/用户限流`、`业务安全检查`、`IP BLOCK`三种方案相结合的方式来应对接口恶意调用，具体如下：

#### IP/用户限流
在NGINX对几类接口配置兜底限流方案，在较粗粒度上进行较宽松的限流
```
// authFree接口
limit_req_zone $binary_remote_addr zone=per_ip_50rps:50m rate=50r/s;
limit_req_zone $binary_remote_addr zone=per_ip_10rps:50m rate=10r/s;

// 查询类接口
limit_req_zone $cookie_token zone=peruser_20rps:200m rate=20r/s;
limit_req_zone $http_token zone=peruser_20rps:200m rate=20r/s;

// 数据管理类接口
limit_req_zone $cookie_token zone=peruser_2rps:100m rate=2r/s;
limit_req_zone $http_token zone=peruser_2rps:100m rate=2r/s;

// 三方服务调用类接口
limit_req_zone $cookie_token zone=peruser_30rpm:50m rate=30r/m;
limit_req_zone $http_token zone=peruser_30rpm:50m rate=30r/m;

// 账号安全相关接口
limit_req_zone $cookie_token zone=peruser_10rpm:50m rate=10r/m;
limit_req_zone $http_token zone=peruser_10rpm:50m rate=10r/m;
```

#### 业务安全检查
针对安全性要求更高的接口，比如三方服务调用相关接口和账号安全类接口，接口设计时，引入更多验证参数，同时结合业务场景进行精准限流，具体措施有：
* 要求必须输入图片验证码  
  `适用场景举例：发送短息、上传CDN(调用超过阈值时）`
* 要求必须输入短信验证码  
  `适用场景举例：注册、绑定手机号、注销账号、找回密码、修改密码等`
* 设置单用户连续错误次数阈值  
  `适用场景举例：输入图片验证码错误、输入短信验证码错误、输入账号密码错误等`
* 限制单用户调用频率，超过阈值时提示其输入验证码证明不是机器人  
  `适用场景举例：频繁调用上传CDN接口、频繁调用人脸识别`
  
#### IP BLOCK
防止长时间的持续恶意调用，比如持续以低于上文限流阈值的频率请求接口  
实现：在NGINX主机部署shell脚本监控access log，当某IP请求某接口频率超过阈值时，自动将该IP加入deny list.


### HTTPS
防止中间人攻击


## WEB前端安全
### 潜在问题
* JS注入
* CSRF
* 文件上传攻击
* 前端存储敏感信息

### 方案
* 使用HTTP security header
* 表单、AJAX提交执行CSRF安全验证
* 代码不硬编码敏感信息
* 代码不做加密操作
* cookie/storage不存明文敏感信息
* 使用security cookie


## 内部系统安全
### 潜在风险
* OP系统账号被盗用
* build系统账号被盗用

### 方案
* OP登陆改为手机号验证码方式
* build系统改为局域网部署，外部使用VPN访问


## 数据存储安全
* 数据服务器全部内网部署
* 按最小特权原则设定访问账号权限
* 数据库replication  
  `主要解决磁盘损坏问题`
* 数据库每日自动备份  
  `建立兜底的safe point`
* 记录核心数据修改/删除  
  `用于责任人追溯和数据恢复`


## 应用服务器安全
* 除网关外，所有主机部署在内网
* 公网主机防火墙，只开放需要的端口
* 内网主机分环境隔离，生产/测试环境主机不能相互访问、使用独立对外VPN
* 所有主机远程登陆使用证书方式

