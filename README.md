## DEx 前端面试题

### 题目概述

设计一个为证券交易市场显示 **order book**  以及 **成交列表**的Web UI。

一个原始订单(Order)由（单号(number)，买(bid)/卖(ask)，数量(quantity)，价格(price)，时间(timestamp))表示，例如
- (1001, ask, 6, 480, 1506744158059)
- (1002, bid, 8, 470, 1506744168241)
- (1003, bid, 5, 460, 1506744168541)

当市场上出现一个买单的价格高于一个卖单的时候，两单（互为对手单）可以成交，成交价格为两单价格的平均数。当一个单可以和多个对手单成交时，优先和价格更优的先成交；价格相同的优先和编号更小的成交。

比如上面的例子，最低的卖单价(480)比最高的买单价(470)还要高，所以无成交。若此时新报一单（1004, ask, 10, 460)，则该单先与1002成交，成交价为avg(460, 470) = 465，成交量为min(10, 8) = 8；而后再与1003成交，成交价为460，成交量为min(10 - 8, 5) = 2。之后Order book变为
- (1001, ask, 6, 480)
- (1003, bid, 3, 460)


### 后端API：
后端每1秒生成一个新的order, order number从0开始连续递增；价格均为整数。后端为前端在8080端口提供两个http API：
```javascript
GET "/api/getOrders?start=<start order number>&size=<how many orders to fetch>"
```
例如：http://0.0.0.0:8080/api/getOrders?start=10&size=100
获取从编号为10的order开始的100个order。若当前最后一个order编号n < 110，则返回编号从10到n的order；若n < 10，则返回空。
不加参数获取会获取最新的20个订单
注意：此api生成的为按顺序的原始订单，并非orderbook

返回结果示例:
```javascript
[{"number":10,"side":"ask","quantity":13,"price":160, "timestamp":1506744158059},{"number":11,"side":"ask","quantity":10,"price":87,"timestamp": 1506744168241}]
```
GET "/reset"
重置后端状态：清空所有order；新order从1开始重新编号。该命令主要用于调试和演示。

启动index.js可以访问这两个接口


### 前端设计需求：
UI展示：
- Order Book，展示多空分别未成交的5单，以正确的排序展示，加分项完成0.5或者1单位的合并深度的需求
- 成交队列，显示最近30笔成交，按照成交的时间由新到旧排序。点击成交队列中的item，可展开显示成交信息，包括成交时间，成交价格，对手买卖单信息等。
- （加分项）动态效果：加入新的成交单时，现有队列中的元素向下移动，腾出位置。
- （加分项）有简单的UI样式，不限制使用Bootstrap等样式库或者手写。
- （加分项）适配移动端，能在移动设备正常显示。
- 可以根据情况修改接口字段，可以增加，不能减少
- 不限制前端框架的使用，甚至不用框架纯js完成也是可以

构建需求
- 前端代码写在src中，并且能够使用npm build命令把构建后的代码放到static目录中，可使用gulp，grunt，webpack等工具
- 使用npm start可以启动index.js的服务，访问http://localhost:8080/ 可以看到对应需求页面，可参考启动后http://localhost:8080/example
