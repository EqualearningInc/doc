# WixPay 接口使用文档

## 接口概述

WixPay接口用于处理来自Wix平台的支付订单提交。该接口支持通过URL参数中的signature进行认证，确保请求的安全性。

## 接口信息

- **接口地址**: `/api/wixPay/submitOrder`
- **请求方法**: `POST`
- **认证方式**: URL参数认证 (signature)
- **内容类型**: `application/json`

## 认证机制

### Signature认证

接口使用URL参数中的`signature`进行认证，该signature来源于以下URL中的token参数：

```
https://www.amazinggraceinstitute.org/pricing-plans/list?token=*********
```

### 认证流程

1. 从Wix产品页面URL中提取token参数
2. 将token作为signature参数添加到API请求URL中
3. 系统验证signature的有效性

### 认证示例

```javascript
// 从Wix页面URL中提取token
const wixUrl = "https://www.amazinggraceinstitute.org/pricing-plans/list?token=abc123def456";
const urlParams = new URLSearchParams(wixUrl.split('?')[1]);
const token = urlParams.get('token');

// 构建API请求URL
const apiUrl = `/api/wixPay/submitOrder?signature=${token}`;
```

## 请求参数

### URL参数

| 参数名    | 类型   | 必填 | 说明                                    |
| --------- | ------ | ---- | --------------------------------------- |
| signature | string | 是   | 认证签名，来源于Wix页面URL中的token参数 |

### 请求体参数

| 参数名    | 类型   | 必填 | 说明                                                         |
| --------- | ------ | ---- | ------------------------------------------------------------ |
| productId | Guid   | 是   | 产品ID，要购买的产品唯一标识<br />套餐一：8a02f54f-20b6-42b8-9dda-63be7958eb8a<br />套餐二：24b13c10-b587-4a88-9eaf-0d1f2b027fed<br />套餐三：bc7b39b5-d479-4b8b-bba6-cc2e9e56d3c5 |
| paymentId | string | 是   | 支付ID，Wix平台返回的支付标识                                |

## 响应格式

### 成功响应

```json
{
  "status": "OK",
  "message": "Order submitted successfully"
}
```

### 错误响应

```json
{
  "error": "Authentication failed",
  "message": "Invalid signature"
}
```

## 使用示例

### JavaScript示例

```javascript
// 提交WixPay订单
async function submitWixPayOrder(productId, paymentId, signature) {
    try {
        const response = await fetch(`/api/wixPay/submitOrder?signature=${signature}`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({
                productId: productId,
                paymentId: paymentId
            })
        });

        if (response.ok) {
            const result = await response.json();
            console.log('订单提交成功:', result);
            return result;
        } else {
            throw new Error('订单提交失败');
        }
    } catch (error) {
        console.error('提交订单时发生错误:', error);
        throw error;
    }
}

// 使用示例
const productId = "8a02f54f-20b6-42b8-9dda-63be7958eb8a";
const paymentId = "wix_payment_123456";
const signature = "abc123def456"; // 从Wix页面URL中获取的token

submitWixPayOrder(productId, paymentId, signature)
    .then(result => {
        // 处理成功响应
        console.log('订单已提交:', result);
    })
    .catch(error => {
        // 处理错误
        console.error('提交失败:', error);
    });
```

### 

## 业务逻辑

### 订单创建流程

1. **验证认证**: 检查signature参数的有效性
2. **创建交易**: 使用WixPay支付方式创建新的交易记录
3. **设置订单详情**: 
   - 产品ID: 指定的产品
   - 数量: 固定为1
   - 买家ID: 当前认证用户
4. **更新支付状态**: 将订单状态设置为已支付(Paid)
5. **记录支付ID**: 保存Wix平台返回的paymentId

### 数据验证

- 验证productId是否为有效的GUID格式
- 验证paymentId不能为空
- 验证用户认证状态
- 验证产品是否存在且可购买

## 错误处理

### 常见错误码

| 错误码 | 说明           | 解决方案                     |
| ------ | -------------- | ---------------------------- |
| 401    | 认证失败       | 检查signature参数是否正确    |
| 400    | 请求参数错误   | 验证productId和paymentId格式 |
| 404    | 产品不存在     | 确认产品ID是否正确           |
| 500    | 服务器内部错误 | 联系技术支持                 |

### 错误响应示例

```json
{
  "error": "Bad Request",
  "message": "Invalid product ID format",
  "statusCode": 400
}
```

## 安全注意事项

1. **Token保护**: 确保从Wix页面获取的token不被泄露
2. **HTTPS传输**: 建议在生产环境中使用HTTPS协议
3. **参数验证**: 客户端应验证所有输入参数
4. **错误信息**: 避免在错误响应中暴露敏感信息

