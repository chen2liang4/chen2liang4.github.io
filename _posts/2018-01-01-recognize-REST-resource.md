---
title: 识别RESTful API资源
---

为这么一个功能实现API：用户在电商网站可以点击某产品，向系统发起一个请求，系统根据商家预定的规则或者大数据，产生一个组合产品供消费者选择。用户的请求需要记录，以供以后为客户做更精准的推送。

根据需求，设计了一个RESTful风格的 API，下面的例子为方便探讨，删去了部分字段。
```json
Request:
POST /deal-requests
{
  "customerId": "jboss",
  "productSku": "123"
}

Response:
201 created
{
  "id": "1101",
  "customerId": "jboss",
  "productSku": "123",
  "deal": {
    "id":"2001",
    "price": 200.00, 
    "productSkus": ["123", "124"]
  }
}
```

先后有两位同事提出异议，不理解为何这样做。一种更直白的方案应该像这样：
```json
Request:
POST /get-deal
{
  "customerId": "jboss",
  "productSku": "123"
}

Response:
200 ok
{
  "id": "2001",
  "price": 200.00,
  "productSkus": ["123", "124"]
}
```

看起来很容易理解，发起一个getDeal的请求，然后返回一个deal内容。但这种并不符合RESTful风格。至于是否一定要用RESTful风格，是另外一个话题，假设我们是要做一个RESTful风格的API。

RESTful是以资源为中心的，这也决定了一个API的URL。那么这个AP应该对应的是哪一个资源？

Client是想获取deal资源的，如果client知道是哪一个deal的话，比如id=2001， 那么可以通过标准的GET方法来获取该deal的详细信息
```
GET /deals/2001
```
可是client并不知道要返回的是哪一个deal，这个是由server端的逻辑来决定的。甚至在client发出请求的时候，这个deal对象可能还不存在。那么是否该
```json
POST /deals
{
  "customerId": "jboss",
  "productSku": "123"
}
```
这看起来是一个标准创建deal的API，但是并不包含创建一个deal所需要的信息，如这个deal的price和相关的产品信息productSku。

没有必需信息怎么能创建一个新的资源呢？

此外，我们的deal也可能是由一个外部引擎或者另外一个服务来产生，如果我们需要提供一个API来保存这些外部系统生成的deal资源，我们就不能再使用标准的格式来创建deal了，因为我们不能通过body中的内容来区别API。
```json
POST /deals
{
  "price": 200.00, 
  "productSkus": ["123", "124"]
}
```
经过分析，我们发现应该还存在一种deal-request资源，包含customerId和productSku属性（实际上还有时间属性）。这些信息我们也是要求记录下来的，以供系统学习使用。

deal-request资源还有一个重要的属性就是，和其关联的deal资源。这也是在我们的场景中client需要使用的信息。Client并不负责分配哪个deal资源给deal-request，更不用去关心这个deal是如何产生的。这个由Server来完成，可能是根据deal-request信息立即计算产生的；也可以由专有服务提前产生了一批放在pool中，然后从中选取一个和deal-request关联。

所以过程变成了：Client发出一个deal-request创建请求，server回复201创建成功，并返回deal-request资源，里面包含了关联的deal资源。这样理解下来API就清晰了。

识别资源是RESTful API设计中的重要工作。我们往往会采取比较直白，容易实现和理解的设计，避免过度设计。但随着发展，系统慢慢就陷入了混乱，因为一开始我们并没有深层次的分析清楚。实际上更正确的做法是抓住问题的本质，才能保持简单和稳定。