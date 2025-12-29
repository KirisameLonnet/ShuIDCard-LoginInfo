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
# 一、校园卡账户密码Token获取法：

## 登录基础信息

- **基础URL**: `http://10.10.10.147/openservice`
- **登录接口路径**: `/epeortalAuth/login`
- **请求方法**: `POST`
- **Content-Type**: `application/json`

## 请求报文格式

- **custname**: 姓名明文
- **stuempno**: 学工号明文
- **pwd**: 密码密文(AES)

## 加密规格说明

- **算法**: AES ECB
- **静态加密密钥**: `IhaIWKKs9AJpn5ip`
- **Padding**: PKCS7
- **Output**: Base64

## 令牌管理

- **字段**: `accessToken`
- **有效期**: 7200 s
- **Header键**: `sw-authorization`
- **Header值**: `Bearer "<accessToken>"`

## 请求示例

```bash
curl -X POST "http://10.10.10.147/openservice/epeortalAuth/login" \
     -H "Content-Type: application/json" \
     -H "User-Agent: Mozilla/5.0 (Linux; Android 10; Mobile)" \
     -d '{
           "custname": "李田所",
           "stuempno": "22114514",
           "pwd": "xxxxxxxxxxxxxxxx=="
         }'
```

## 登录成功返回结构体

```json
{
  "retcode": "0",
  "retmsg": "成功",
  "data": {
    "type": "SHU",
    "token": "xxxxxxxxxxxxxxxx.xxxxxxxxxxxxxx"
  }
}
```

# 二、通过HMAC签名获取token
此方法利用已生成的 HMAC 签名直接换取 Token，绕过 AES 加密逻辑，理论上本方法是不可行的，但笔者发现 HMAC 签名的时间戳虽然存在但并未被校验（似乎），从而实现了该方法。

## 令牌交换接口

- 接口路径: `/epeortalAuth/hmacLogin`
- 请求方法: `POST`
- 优势: 无需处理 AES 密钥及加密过程，直接利用现有签名。

## 请求报文格式

```json
{
  "sign": "0fx6dxxcx6xx1x6x5xexd6xbx1xxxbxxx2xx7xx1",
  "sign_method": "HMAC",
  "stuempno": "22114514",
  "timestamp": "20300120030455"
}
```

## 登录成功返回结构体

同前一个方法的返回结构体。

## 额外接口信息

- **电费查询路径**: `/miniprogram/queryroominfo`

## 有关HMAC签名的获取

- 理论上：
  HMAC签名的生成需要使用一个密钥（通常是一个字符串），结合请求的其他参数（如学工号和时间戳）通过HMAC算法生成签名。具体的密钥和算法细节需要通过分析前端代码或网络请求来获取。笔者暂未分析出具体细节，欢迎有兴趣的同学进行研究。
- 笔者的获取方法：
  打开企业微信-工作台-校园一卡通-电费支付-右上角复制链接，链接格式如下：

  ```
  10.10.10.147/epeortal/pages/h5/elecQuery?stuempno=22114514&timestamp=20300120030455&sign=0fx6dxxcx6xx1x6x5xexd6xbx1xxxbxxx2xx7xx1&sign_method=HMAC
  ```

- url参数解释：
  - stuempno: 学工号
  - timestamp: 时间戳
  - sign: HMAC签名
  - sign_method: 签名方法
- 即便上文已提及HMAC签名的时间戳可能未被校验，但发送报文时仍需携带该参数，因为时间戳是签名生成的一部分。

# 免责声明 / Disclaimer

### 1. 法律及合规性
本项目（ShuIDCard-LoginInfo）仅供 技术研究 和 教育交流 使用。

本项目的初衷是探索 Web 安全及自动化技术，作者不鼓励、不建议且不支持任何违反学校规章制度或法律法规的行为。

用户因使用本项目内容而产生的任何行为（如自动化脚本访问频率过高、非授权登录等）及其导致的后果，由 用户本人完全承担，作者不对此负责。

### 2. 许可证与说明

本项目遵循 [MIT License](./LICENSE)

不保证稳定性：由于学校系统（上海大学校园网及相关子系统）可能会随时进行升级、补丁或逻辑变更，本项目提供的代码或信息可能随时失效。

账号安全风险：使用非官方方法登录访问学校站点可能被系统识别，进而导致你的学工号/校园卡账号被系统封禁、限制访问或面临学校相关部门的问责。用户应充分知晓此类风险。

### 3. 禁止用途
严禁将本项目的内容用于任何 商业用途 或 大规模爬虫 行为。

严禁利用本项目干扰学校服务器的正常运行。

### 4. 知识产权与删除
本项目中提及的接口、密钥及相关算法均通过公开合法渠道（浏览器开发者工具）获取

如果相关方认为本项目侵犯了其合法权益，请联系作者，作者将在核实后立即删除相关内容。
