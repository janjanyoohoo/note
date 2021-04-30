# SSO单点登录

## SIngle Sign On

一处登录,其他服务不用登录

## 实现方式

- Session广播 机制
  - 通过Session复制,将Session赋值到每个服务中
  - 耗费内存, 且服务多的情况下,复制量大严重浪费资源
- Cookie+Redis实现
  - 在任何一个项目登录,将Cookie存储唯一id (sessionId)
  - Session信息存储在Redis中, 多个服务使用同一个redis,实现Session共享
  - 访问其他模块时,发送的请求会携带cookie,获取cookie值,拿着cookie值执行相关操作
    - 把cookie获取值,到redis进行查询,根据key进行查询验证
- Token
  - 按照一定规则生成的字符串,把登录之后的用户数据放到字符串中,返回给客户端
    - 可以把字符串通过cookie返回
    - 把字符串通过地址栏作为参数返回
  - 再次访问其他模块时,每次访问其他项目时带着生成的字符串,在访问的模块中获取token验证
- 其他

