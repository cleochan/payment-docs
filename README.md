# 恒盈四方支付网关文档（极简版）

### <span id="1-----">1 概要</span>

#### <span id="11-----">1.1 接口用途</span>

本接口用于接入恒盈四方支付系统。本接口纯Restful风格，传入参数及返回参数全部为JSON格式。

#### <span id="12-----">1.2 接口申请</span>

请联系客服申请接口，申请通过之后您可获得以下信息：

    1. 专属接口请求地址(apiAddress)
    2. 应用ID(appId)
    3. 密钥(key)

#### <span id="13-----">1.3 Header 参数</span>

请求方式：POST

| 参数名       | 必选 | 类型/参数值      | 说明         |
| ------------ | ---- | ---------------- | ------------ |
| Content-Type | 是   | application/json | 请求参数类型 |

#### <span id="14-----">1.4 签名</span>

  1. 将全部传入参数（除了sign）的参数名按照字典序排列，请注意值为空的参数无需传入
  2. 构建为链接参数格式
  3. 在最后加上key（密钥请联系客服）
  4. 将字符串进行md5加密
  5. 转换为小写

##### <span id="141-----">1.4.1 签名示例</span>

 - 1.传入参数

```json
{
    "appId": "B32D954CC4E25491F99EFE42DF1CCBBF",
    "channelId": 1,
    "actionValue": 1200.00,
    "outOrderId": "ESP837647136232",
    "outTips": "测试订单"
}
```

 - 2.参数名按字典序排列

```json
{
    "actionValue": 1200.00,
    "appId": "B32D954CC4E25491F99EFE42DF1CCBBF",
    "channelId": 1,
    "outOrderId": "ESP837647136232",
    "outTips": "测试订单"
}
```

 - 3.构建为链接参数格式

actionValue=1200.00&appId=B32D954CC4E25491F99EFE42DF1CCBBF&channelId=1&outOrderId=ESP837647136232&outTips=测试订单

 - 4.在最后加上key（假设key为aaabbbccc）

actionValue=1200.00&appId=B32D954CC4E25491F99EFE42DF1CCBBF&channelId=1&outOrderId=ESP837647136232&outTips=测试订单&key=aaabbbccc

 - 5.转为小写的md5，即为sign

6cf32dc0412ff0614d3f0091b556883f

​    

### <span id="2-----">2 接口列表</span>

​    

#### <span id="21-----">2.1 代收</span>

请求地址：`{apiAddress}/payment`

##### <span id="211-----">2.1.1 传入参数</span>

| 参数名      | 必选 | 类型    | 字段长度 | 例子     | 说明                     |
| ----------- | ---- | ------- | -------- | -------- | ------------------------ |
| appId       | 是   | string  | 32       |          | 应用ID                   |
| channelId   | 是   | int     | 5        | 1        | 支付渠道ID，参考渠道章节 |
| actionValue | 是   | decimal | 18, 2    | 2100.10  | 申请代付的金额(元)       |
| outOrderId  |      | string  | 100      |          | 第三方订单号             |
| outTips     |      | string  | 100      | 测试订单 | 第三方备注               |
| sign        | 是   | string  | 32       |          | 参考签名章节             |

##### <span id="212-----">2.1.2 返回参数</span>

| 参数名 | 类型   | 字段长度 | 例子    | 说明                                         |
| ------ | ------ | -------- | ------- | -------------------------------------------- |
| result | int    | 1        | 1       | 调用结果，1=成功 0=失败                      |
| url    | string | 255      |         | 用于跳转至支付页面的链接，请在应用中直接打开 |
| msg    | string | 200      | success | 如出错时，返回出错原因，成功时为success      |

##### <span id="213-----">2.1.3 返回示例</span>

 - 成功

```json
{
    "result": 1,
    "url": "https://aaa.bbb.ccc/kdshfaksjdfkwhejfs",
    "msg": "success"
}
```

 - 失败

```json
{
    "result": 0,
    "url": null,
    "msg": "签名不正确"
}
```

##### <span id="214-----">2.1.4 支付渠道列表</span>

| ID   | 名称     |
| ---- | -------- |
| 1    | 微信支付 |


​    

#### <span id="22-----">2.2 代收订单查询</span>

请求地址：`{apiAddress}/payment-orders`

##### <span id="231-----">2.2.1 传入参数</span>

| 参数名    | 必选 | 类型     | 字段长度 | 例子 | 说明                                          |
| --------- | ---- | -------- | -------- | ---- | --------------------------------------------- |
| appId     | 是   | string   | 32       |      | 应用ID                                        |
| startTime | 是   | datetime | -        |      | 搜索下单时间：开始时间(包含这一秒)            |
| endTime   | 是   | datetime | -        |      | 搜索下单时间：结束时间(不包含这一秒)          |
| pageId    | 否   | int      | 5        | 12   | 页码，留空则自动拉取第1页，每页最多返回1000条 |
| sort      | 否   | string   | 4        | asc  | asc=正序，desc=倒序，留空则默认为desc         |
| sign      | 是   | string   | 32       |      | 签名                                          |

##### <span id="222-----">2.2.2 返回参数</span>

| 参数名     | 类型   | 字段长度 | 例子           | 说明                                      |
| ---------- | ------ | -------- | -------------- | ----------------------------------------- |
| result     | int    | 1        | 0              | 调用结果，1=成功 0=失败                   |
| data       | array  | -        |                | 订单数据，参考以下表格                    |
| totalPages | int    | 5        | 12             | 总页数，如果大于1说明后面还有页数，需翻页 |
| msg        | string | 100      | 开始时间为必填 | 如出错时，返回出错原因，成功时为success   |

  >> data订单数据格式

| 参数名      | 类型     | 字段长度 | 例子         | 说明                                                         |
| ----------- | -------- | -------- | ------------ | ------------------------------------------------------------ |
| appId       | string   | 32       |              |                                                              |
| balanceId   | int      | 11       | 98126517     | 对账流水号                                                   |
| channelId   | int      | 5        | 1            | 支付渠道ID                                                   |
| actionValue | decimal  | 18, 2    | 1000.00      | 代收金额                                                     |
| chargeValue | decimal  | 18, 2    | 20.00        | 手续费                                                       |
| actualValue | decimal  | 18, 2    | 980.00       | 实际到账金额                                                 |
| outOrderId  | string   | 100      | ES3891879987 | 第三方订单号                                                 |
| outTips     | string   | 100      | 测试订单     | 第三方备注                                                   |
| status      | Int      | 3        | 100          | 订单状态<br />0=待处理<br />100=成功<br />200=失败<br />300=已取消 |
| createTime  | datetime |          |              | 创建时间                                                     |
| updatedTime | datetime |          |              | 状态最后更新时间                                             |

##### <span id="223-----">2.2.3 调用示例</span>

 - 传入参数

```json
{
    "appId": "B32D954CC4E25491F99EFE42DF1CCBBF",
    "startTime": "2021-10-01 00:00:00",
    "endTime": "2021-10-01 23:59:59",
    "pageId": 1,
    "sort": "desc",
    "sign": "cbc0b11733b785b0317f1cc7d6f20fd8"
}
```

 - 返回参数（成功）

```json
{
    "result": 1,
    "data": [{
        "appId": "B32D954CC4E25491F99EFE42DF1CCBBF",
        "balanceId": 832647386,
        "channelId": 1,
        "actionValue": 1000.00,
        "chargeValue": 20.00,
        "actualValue": 980.00,
        "outOrderId": "ESP8723687432628",
        "outTips": NULL,
        "status": 100,
        "createTime": "2021-10-01 12:30:35",
        "updatedTime": "2021-10-01 12:35:01"
    }, {
        "appId": "B32D954CC4E25491F99EFE42DF1CCBBF",
        "balanceId": 832647333,
        "channelId": 1,
        "actionValue": 2500.00,
        "chargeValue": 35.40,
        "actualValue": 2464.60,
        "outOrderId": "ESP17236836478",
        "outTips": "测试",
        "status": 200,
        "createTime": "2021-10-01 11:25:20",
        "updatedTime": "2021-10-01 12:35:01"
    }],
    "totalPages": 2, //还有第2页，传入参数里pageId改为2，可获取第2页内容
    "msg": "success"
}
```

 - 返回参数（失败）

```json
{
    "result": 0,
    "data": [],
    "totalPages": null,
    "msg": "页码不存在"
}
```
