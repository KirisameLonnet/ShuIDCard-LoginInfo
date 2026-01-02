# ShuIDCard-LoginInfo

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

本仓库旨在研究与记录上海大学校园卡系统的登录认证机制，包括 SM4 加密逻辑及 HMAC 签名登录方式。

---

## 目录
- [一、校园卡账户密码 Token 获取法](#一校园卡账户密码-token-获取法)
    - [1.1 登录基础信息](#11-登录基础信息)
    - [1.2 加密规格说明](#12-加密规格说明)
    - [1.3 Python 加密实现](#13-python-加密实现)
    - [1.4 令牌管理](#14-令牌管理)
    - [1.5 请求与响应示例](#15-请求与响应示例)
- [二、通过 HMAC 签名获取 Token](#二通过-hmac-签名获取-token)
    - [2.1 登录原理](#21-登录原理)
    - [2.2 请求报文格式](#22-请求报文格式)
- [三、其他接口信息](#三其他接口信息)
- [免责声明 / Disclaimer](#免责声明--disclaimer)

---

## 一、校园卡账户密码 Token 获取法

此方法通过模拟网页端的加密逻辑，使用学工号和密码换取访问令牌。

### 1.1 登录基础信息

- **基础 URL**: `http://10.10.10.147/openservice`
- **登录接口**: `/epeortalAuth/login`
- **请求方法**: `POST`
- **Content-Type**: `application/json`

#### 请求参数
| 字段 | 说明 |
| :--- | :--- |
| `custname` | 姓名明文 |
| `stuempno` | 学工号明文 |
| `pwd` | 密码密文 (SM4加密) |

### 1.2 🔐 加密规格说明

该系统使用的是 **国密 SM4** 算法（ECB 模式）。

- **算法标准**: SM4 (GM/T 0002-2012)
- **加密模式**: ECB (Electronic Codebook)
- **静态密钥 (Key)**: `IhaIWKKs9AJpn5ip` (128-bit)
- **填充方式 (Padding)**: PKCS7
- **输出编码**: Base64

> [!CAUTION]
> Python 标准库默认不支持 SM4，需安装 `gmssl` 库：`pip install gmssl`。

### 1.3 Python 加密实现

```python
import base64
from gmssl.sm4 import CryptSM4, SM4_ENCRYPT

def encrypt_password(password: str) -> str:
    key = b'IhaIWKKs9AJpn5ip'
    crypt_sm4 = CryptSM4()
    crypt_sm4.set_key(key, SM4_ENCRYPT)
    
    # 手动 PKCS7 填充
    block_size = 16
    padding_len = block_size - (len(password) % block_size)
    padded_text = password.encode() + bytes([padding_len] * padding_len)
    
    # ECB 加密并 Base64 编码
    encrypt_value = crypt_sm4.crypt_ecb(padded_text)
    return base64.b64encode(encrypt_value).decode('utf-8')
```

### 1.4 令牌管理

- **字段**: `accessToken`
- **有效期**: 7200 s
- **Header 键**: `sw-authorization`
- **Header 值**: `Bearer "<accessToken>"`

### 1.5 请求与响应示例

#### 请求示例
```bash
curl -X POST "http://10.10.10.147/openservice/epeortalAuth/login" \
     -H "Content-Type: application/json" \
     -H "User-Agent: Mozilla/5.0 (Linux; Android 10; Mobile)" \
     -d '{
           "custname": "李田所",
           "stuempno": "22114514",
           "pwd": "Generate_By_SM4_Script_Above=="
         }'
```

#### 响应结构 (成功)
```json
{
  "retcode": "0",
  "retmsg": "成功",
  "data": {
    "type": "SHU",
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.xxxxxxxxxxxxxx"
  }
}
```

---

## 二、通过 HMAC 签名获取 Token

此方法利用已生成的 HMAC 签名直接换取 Token，绕过密码加密逻辑。

### 2.1 登录原理

- **接口路径**: `/epeortalAuth/hmacLogin`
- **请求方法**: `POST`
- **特性分析**: 经测试，服务端对 HMAC 签名中的 `timestamp` 似乎未进行严格的时效性校验。这意味着只要捕获一次有效的签名数据包，理论上可以长期重放使用。

### 2.2 请求报文格式

```json
{
  "sign": "0fx6dxxcx6xx1x6x5xexd6xbx1xxxbxxx2xx7xx1",
  "sign_method": "HMAC",
  "stuempno": "22114514",
  "timestamp": "20251229132819"
}
```

---

## 三、其他接口信息

- **电费查询路径**: `/miniprogram/queryroominfo`
- **鉴权方式**: 需携带 `sw-authorization` 请求头。

---

## 免责声明 / Disclaimer

### 1. 法律及合规性
本项目（ShuIDCard-LoginInfo）仅供 **技术研究** 和 **教育交流** 使用。

本项目的初衷是探索 Web 安全及自动化技术，作者不鼓励、不建议且不支持任何违反学校规章制度或法律法规的行为。

用户因使用本项目内容而产生的任何行为（如自动化脚本访问频率过高、非授权登录等）及其导致的后果，由 **用户本人完全承担**，作者不对此负责。

### 2. 许可证与说明
本项目遵循 [MIT License](LICENSE)。

- **不保证稳定性**：由于学校系统可能会随时进行升级、补丁或逻辑变更，本项目提供的代码或信息可能随时失效。
- **账号安全风险**：使用非官方方法登录访问学校站点可能被系统识别，进而导致你的账号被封禁或面临问责。用户应充分知晓此类风险。

### 3. 禁止用途
- 严禁将本项目的内容用于任何 **商业用途** 或 **大规模爬虫** 行为。
- 严禁利用本项目干扰学校服务器的正常运行。

### 4. 知识产权与删除
本项目中提及的接口、密钥及相关算法均通过公开合法渠道获取。如果相关方认为本项目侵犯了其合法权益，请联系作者，作者将在核实后立即删除相关内容。
