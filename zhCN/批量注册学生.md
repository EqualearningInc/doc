
# 批量注册学生的API 接口说明

## 1. API 路径
**请求方法**：`POST`  
**接口地址**：`/api/student/batchregister`

---

## 2. 传输数据格式
```json
{
  "inviteCode": "225406",
  "teamName": "Team_Invite",
  "teamType": "Family",
  "emailConfirmed": true,
  "itemList": [
    {
      "sNo": "013",
      "userName": "i022013",
      "firstName": "student",
      "lastName": "022013",
      "password": "123456",
      "gender": 1,
      "birthDay": "1990-04-20",
      "email": "i022011@gmail.com",
      "phoneNumber": "13812558336"
    },
    {
      "sNo": "014",
      "userName": "i022014",
      "firstName": "student",
      "lastName": "022014",
      "password": "123456",
      "gender": 1,
      "birthDay": "1983-02-19",
      "email": "i022012@gmail.com",
      "phoneNumber": "13812558336"
    }
  ]
}

```

## 3. 字段描述

|      字段名      |      类型       |              描述              |
| :--------------: | :-------------: | :----------------------------: |
|   `inviteCode`   |    `string`     |             邀请码             |
|    `teamName`    |    `string`     |            团队名称            |
|    `teamType`    |    `string`     |            团队类型            |
| `emailConfirmed` |    `boolean`    | 邮箱确认状态（`false`/`true`） |
|    `itemList`    | `array[object]` |      成员列表（对象数组）      |
|      `sNo`       |    `string`     |              学号              |
|    `userName`    |    `string`     |             用户名             |
|   `firstName`    |    `string`     |              名字              |
|    `lastName`    |    `string`     |              姓氏              |
|    `password`    |    `string`     |              密码              |
|     `gender`     |    `number`     |   性别（0:未知, 1:男, 2:女）   |
|    `birthDay`    |    `string`     |   生日（格式：`YYYY-MM-DD`）   |
|     `email`      |    `string`     | 邮箱（示例：`example@qq.com`） |
|  `phoneNumber`   |    `string`     |            电话号码            |

------

## 4. 鉴权说明

### 请求头参数

- **`e-timestamp`**
  当前 Unix 时间戳（秒级）

- **`e-signature`**
  签名生成规则：

  ```text
  method=post&path=/api/student/batchregister&timestamp={timestamp}&apikey={apiKey}
  ```

  1. 所有字母小写
  2. 其中timestamp与e-timestamp参数值一致
  3. 使用 `HMAC-SHA256` 算法以 `apiKey` 为密钥加密

### JavaScript 示例代码

```javascript
// 定义 API Key
const apiKey = "sk-863b8cb0c7b24dc49299a3a7a7f8e57b";

// 生成 Unix 时间戳（秒级）
const unixTimestamp = Math.floor(Date.now() / 1000);

// 设置请求头
apt.setRequestHeader("e-timestamp", unixTimestamp);

// 构造签名数据
const requestData = `method=post&path=/api/student/batchregister&timestamp=${unixTimestamp}&apikey=${apiKey}`.toLowerCase();

// 计算 HMAC-SHA256 签名
const signature = CryptoJS.HmacSHA256(
  CryptoJS.enc.Utf8.parse(requestData),
  CryptoJS.enc.Utf8.parse(apiKey)
).toString(CryptoJS.enc.Hex);

// 添加签名请求头
apt.setRequestHeader("e-signature", signature);
```