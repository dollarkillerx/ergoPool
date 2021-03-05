**首先要确保你的Redis实例和备份配置正确http://redis.io/topics/persistence.**。

请记住，pool会在**Shannon**中维护所有余额。

# 处理和解决支付问题

**你必须在单独的进程中运行赔付模块**，最好不要以守护进程的形式运行，每天处理赔付2-3次，然后观察情况。**你必须配置日志**，否则会导致大问题。

模块将获取账户并依次处理支付。

对于每一个达到最小阈值的账户。

* 检查一个节点上是否有足够的对等体。
* 检查账户是否已解锁

如果任何一项检查失败，模块甚至不会尝试继续。

* 检查我们是否有足够的钱来支付（正常情况下不应该发生）。
* 锁定付款

如果不能锁定支付（通常在失败后存在另一个锁定），模块将停止支付。

* 扣除矿工的余额，并记录待支付的款项。
* 通过`eth_sendTransaction`向一个节点提交交易。

**如果交易提交失败，支付将保持锁定和停止的错误状态。

如果交易提交成功，我们有一个TX哈希值。

* 把这个TX哈希值写入数据库
* 解锁赔款

以此类推。对每个账户都重复。

在支付环节结束后，如果您启用了 "bgsave "选项，支付模块将在Redis上执行 "BGSAVE"（后台保存）。

## 解决支付失败的问题（自动）。

如果你的支付没有被记录，也没有被Ethereum网络确认，你可以自动解决它。你需要在维护模式下通过设置`RESOLVE_PAYOUT=1`或`RESOLVE_PAYOUT=True`环境变量来解决支付问题。

`RESOLVE_PAYOUT=1 ./build/bin/open-ethereum-pool payouts.json`。

Payout模块将从Redis中获取所有带有键`eth:paysations:pending`的行，并将余额贷记回给矿工。通常情况下，你将只有一个条目。

如果你看到 "没有待解决的付款"，我们没有关于失败的借记数据。

如果有一个借记操作被执行，但之后没有实际的汇款（在`eth_sendTransaction`返回错误之后），你可能会看到。

```
会将以下余额打回。
地址： 0xb85150eb365e7df0941f0cf08235f987ba91506a, Amount: 166798415 Shannon, 2016-05-11 08:14:34
```

其次

```
记入166798415香农回0xb85150eb365e7df0941f0cf08235f987ba91506a。
```

通常每次维护运行结束时，都会出现以下信息并停止。

```
解锁的赔款
现在，你必须重新启动支付模块，RESOLVE_PAYOUT=0才能正常运行。
```

取消设置`RESOLVE_PAYOUT=1`或用`RESOLVE_PAYOUT=0`运行支付。

## 解决支付失败的问题(手动)

你可以使用 "geth "和 "redis-cli "实用程序进行手动维护。

### 检查失败的交易。

在`redis-cli`中执行以下命令。

```
ZREVRANGE "eth:paysings:pending" 0 -1 WITHSCORES
```

结果将是这样的。

> 1) "0xb85150eb365e7df0941f0cf08235f987ba91506a:25000000"

是一对`LOGIN:AMOUNT`。

>2) "1462920526"

这是一个 "UNIXTIME"。

### 手动提交付款

**使用区块资源管理器确保没有发送TX。如果付款确实存在于区块链中，则跳过此步骤。

```javascript
eth.sendTransaction({
  来自：eth.coinbase。
  到。'0xb85150eb365e7df0941f0cf08235f987ba91506a',
  值：web3.toWei(25000000, 'shannon')
})

// => 0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331
```

**记下TX哈希**.

###在Redis中存储支付

也可用于修正缺失的付款条目。

```
ZADD "eth:payments:all" 1462920526 0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331:0xb85150eb365e7df0941f0cf08235f987ba91506a:25000000
```

```
ZADD "eth:payments:0xb85150eb365e7df0941f0cf08235f987ba91506a" 1462920526 0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331:25000000
```

###删除错误的付款条目

```
ZREM "eth:payments:pending" "0xb85150eb365e7df0941f0cf08235f987ba91506a:25000000"
```

### 更新内部统计

```
HINCRBY "eth:finances" pending -25000000
HINCRBY "eth:finances "支付了2.5亿。
```

### 解锁赔款

```
DEL "eth:paysations:lock"
```

## 解决缺失的付款条目

如果池确实支付了，但没有记录交易，滚动到 "Store Payment in Redis "部分。你应该有一个交易哈希从块资源管理器。

## 交易没有确认

如果你确定，就手动重复，你应该有所有的日志。