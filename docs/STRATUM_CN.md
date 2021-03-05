# Stratum Mining Protocol

这是对这个池中使用的stratum协议的描述。

Stratum定义了简单的异常处理。被拒绝的共享的例子是这样的。

```javascript
{ "id": 1, "jsonrpc": "2.0", "result": null, "error": error":{ code: 23, message: "无效共享" } }
```

每一个有异常的响应后都会断开连接。

## 认证

请求是这样的

```javascript
{
  "id": 1,
  "jsonrpc"。"2.0",
  "method". "eth_submitLogin"， "eth_submitLogin",
  "params"。["0xb85150eb365e7df0941f0cf08235f987ba91506a"]
}
```

请求可以包含额外的第2个参数（例如电子邮件）。

```javascript
{
  "id": 1,
  "jsonrpc"。"2.0",
  "method". "eth_submitLogin"， "eth_submitLogin",
  "params"。["0xb85150eb365e7df0941f0cf08235f987ba91506a", "admin@example.net"]
}
```

响应成功。

```javascript
{ "id": 1, "jsonrpc": "2.0", "result": true }。
```

异常情况。

```javascript
{ "id": 1, "jsonrpc": "2.0", "result": null, "error": code: -1, message: "无效登录" } }
```

## 工作请求

请求是这样的

```javascript
{ "id": 1, "jsonrpc": "2.0", "method": "eth_getWork" }
```

响应成功。

```javascript
{
  "id": 1,
  "jsonrpc"。"2.0",
  "结果"。[
      "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
      "0x5eed00000000000000000000000000005eed0000000000000000000000000000",
      "0xd1ff1c01710000000000000000000000d1ff1c01710000000000000000000000"
    ]
}
```

异常情况。

```javascript
{ "id": 10, "result": null, "error": 代码： 0, message: "工作未准备好" } }
```

## 新工作通知

如果有新的作业，服务器会将作业发送到对等体。

```javascript
{
  "jsonrpc"。"2.0",
  "结果"。[
      "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
      "0x5eed00000000000000000000000000005eed0000000000000000000000000000",
      "0xd1ff1c01710000000000000000000000d1ff1c01710000000000000000000000"
    ]
}
```

## 分享提交

请求是这样的

```javascript
{
  "id": 1,
  "jsonrpc"。"2.0",
  "方法"。"eth_submitWork",
  "params": [
    "0xe05d1fd4002d962f",
    "0x6c872e2304cd1e64b553a65387d7383470f22331aff288cbce5748dc430f016a",
    "0x2b20a6c641ed155b893ee750ef90ec3be5d24736d16838b84759385b6724220d"
  ]
}
```

请求可以包含可选的`worker`param。

```javascript
{ "id": 1, "worker": "rig-1" /* ... */ }
```

响应。

```javascript
{ "id": 1, "jsonrpc": "2.0", "result": true }。
{ "id": 1, "jsonrpc": "2.0", "result": false }。
```

例外情况。

Pool可能会在提交无效份额时返回异常，通常会被暂时禁止。

```javascript
{ "id": 1, "jsonrpc": "2.0", "result": null, "error": error":{ code: 23, message: "无效共享" } }
```

```javascript
{ "id": 1, "jsonrpc": "2.0", "result": null, "error": code: 22, message: "重复共享" } }
{ "id": 1, "jsonrpc": "2.0", "result": null, "error": code: -1, message: "无效股份比率高" } }
{ "id": 1, "jsonrpc": "2.0", "result": null, "error": error":{ code: 25, message: "未订阅" } }
{ "id": 1, "jsonrpc": "2.0", "result": null, "error": code: -1, message: "Malformed PoW result" } }
```

## 提交Hashrate

`eth_submitHashrate`是一个无意义的方法。Pool会忽略它，回复总是。

```javascript
{ "id": 1, "jsonrpc": "2.0", "result": true }。
```