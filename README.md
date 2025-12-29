## 登录基础信息
基础URL    http://10.10.10.147/openservice
登录接口路径    /epeortalAuth/login
请求方法    POST
Content-Type    application/json

## 报文格式
custname: 姓名明文
stuempno: 学工号明文
pwd: 密码密文(AES)

## 加密逻辑规格
AES ECB
静态加密密钥：IhaIWKKs9AJpn5ip
Padding：PKCS7
Output：Base64

## 令牌管理
字段：accessToken
有效期：7200 s
Header键: sw-authorization
Header值: Bearer "<accessToken>"

## 请求示例

curl -X POST "http://10.10.10.147/openservice/epeortalAuth/login" \
     -H "Content-Type: application/json" \
     -H "User-Agent: Mozilla/5.0 (Linux; Android 10; Mobile)" \
     -d '{
           "custname": "李田所",
           "stuempno": "22114514",
           "pwd": "xxxxxxxxxxxxxxxx=="
         }'

## 登录成功返回结构体
{
  "retcode": "0",
  "retmsg": "成功",
  "data": {
    "type": "SHU",
    "token": "xxxxxxxxxxxxxxxx.xxxxxxxxxxxxxx"
  }
}

## 额外信息
电费查询路径:/miniprogram/queryroominfo
