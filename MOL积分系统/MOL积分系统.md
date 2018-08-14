# MOL积分系统接口

>Version：beta
>
>Author：李勇
>
>E-mail：666233@qq.com
>
>Time：2018.08.01

## 业务简介

本模块设计成一个通用的、独立的模块，以后所有业务中的撒给用户的币都以积分形式表达，用接口对数据进行增删改查。

### 项目url预览

![预览](https://i.imgur.com/TyXrDQd.png) 

站在数据的角度，若想满足各大模块的功能需求，可以将接口分成如下几类：
> base url：https://api.xxx.com/path/

| 分类                                  | 接口地址                                   | 参数       |
| :------------------------------------ | ------------------------------------------ | ---------- |
| 新闻主题（分类）topics                | https://api.xxx.com/path/v1/topics         |            |
| 新闻列表+广告news                     | https://api.xxx.com/path/v1/news           | topic的uid |
| 收益profit                            | https://api.xxx.com/path/v1/profit         |            |
| MOL（积分形式）收益记录profit/records | https://api.xxx.com/path/v1/profit/records |            |
| 摩尔币的记录mol/records               | https://api.xxx.com/path/v1/mol/record     |            |

------

### 收益profit

这个接口可以被设计成一个通用的、独立的模块，以后所有业务中的撒给用户的币都以积分形式表达，用该接口来查询余额状态。

> 地址：https://api.xxx.com/path/v1/profit

##### 请求头

```
GET /path/v1/profit
Accept: application/json
Content-Type: application/json;charset=UTF-8
Token: token_G34G34G34G34G35G5
AppVersion: 1.1.1
Platform: Android
```

##### 参数

无

##### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
Cache-Control: no-store
Pragma: no-cache
```

##### 响应

```
{
    "message": "居然被你查询成功了",
    "code": 200,
    "data": {
        "accumulation": "11124525",
        "balance": "95885",
        "currency": "MOL"
    }
}
```

| key               | 类型   | 是否必须 | 描述                                    |
| ----------------- | ------ | -------- | --------------------------------------- |
| message           | String | 是       | 同上                                    |
| code              | int    | 是       | 同上                                    |
| data.accumulation | String | 是       | 该账户历史累积收益，非raw类型的值       |
| data.balance      | String | 是       | 该账户上可用于提现的收益，非raw类型的值 |
| data.currency     | String | 是       | 对应的币种的单位                        |

---

### MOL（积分形式）收益记录profit/records

这个接口可以被设计成一个通用的、独立的模块，以后所有业务中的撒给用户的币都以积分形式表达，用该接口查询收益记录。

> 地址：https://api.xxx.com/path/v1/profit/records

##### 请求头

```
GET /path/v1/profit/records
Accept: application/json
Content-Type: application/json;charset=UTF-8
Token: token_G34G34G34G34G35G5
AppVersion: 1.1.1
Platform: Android
```

##### 参数

?keyword=热点&sort=des&page=0&pageSize=20

| params   | 类型   | 是否必须 | 描述 |
| -------- | ------ | -------- | ---- |
| keyword  | String | 否       | 同上 |
| sort     | String | 否       | 同上 |
| page     | int    | 否       | 同上 |
| pageSize | int    | 否       | 同上 |

##### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
Cache-Control: no-store
Pragma: no-cache
```

##### 响应

```
{
    "message": "居然被你查询成功了",
    "code": 200,
    "page": 0,
    "pageSize": 20,
    "first": "https://...",
    "next": "https://...",
    "previous": "https://...",
    "last": "https://...",
    "data": [
        {
            "uid": "235235235",
            "title": "邀请好友",
            "description": "二级好友邀请好友一根韭菜",
            "amount": "-50",
            "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
            "currency": "MOL",
            "time": "2018-04-23 17:38:44"
        },
        {
            "uid": "235235235",
            "title": "邀请好友",
            "description": "二级好友邀请好友一根韭菜",
            "amount": "-50",
            "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
            "currency": "MOL",
            "time": "2018-04-23 17:38:44"
        }
    ]
}
```

| key              | 类型   | 是否必须 | 描述                               |
| ---------------- | ------ | -------- | ---------------------------------- |
| message          | String | 是       | 同上                               |
| code             | int    | 是       | 同上                               |
| page             | int    | 否       | 同上                               |
| pageSize         | int    | 否       | 同上                               |
| first            | String | 否       | 同上                               |
| next             | String | 否       | 同上                               |
| previous         | String | 否       | 同上                               |
| last             | String | 否       | 同上                               |
| data             | object | 是       | 当前接口的具体数据由该JSON对象承载 |
| data.uid         | String | 是       | 交易记录主键                       |
| data.title       | String | 是       | 交易记录标题名称                   |
| data.description | String | 是       | 交易记录描述                       |
| data.amount      | String | 是       | 交易记录金额                       |
| data.image       | String | 是       | 交易记录图标                       |
| data.currency    | String | 是       | 交易记录币种                       |
| data.time        | String | 是       | 交易记录时间                       |

------

### 提现（从MOL积分币到MOL链）withdraw

这个接口可以被设计成一个通用的、独立的模块，以后所有业务中的撒给用户的币都以积分形式表达，用该接口让用户提现，从MOL积分币换成MOL链上的真实币。

> 地址：https://api.xxx.com/path/v1/withdraw

##### 请求头

```
POST /path/v1/withdraw
Accept: application/json
Content-Type: application/json;charset=UTF-8
Token: token_G34G34G34G34G35G5
AppVersion: 1.1.1
Platform: Android
```

##### 参数

```
{
    "amount": "50",
    "currency": "MOL"
}
```

| params   | 类型   | 是否必须 | 描述                                                         |
| -------- | ------ | -------- | ------------------------------------------------------------ |
| amount   | String | 否       | 提现额度，允许从积分币到MOL链上的币提现多少通过该参数控制，非raw类型，不传表示默认，默认表示全部提现 |
| currency | String | 否       | 默认传MOL（大写）                                            |

##### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
Cache-Control: no-store
Pragma: no-cache
```

##### 响应

```
{
    "message": "居然被你提现成功了",
    "code": 200,
    "data": {
        "uid": "235235235"
    }
}
```

| key      | 类型   | 是否必须 | 描述                                                    |
| -------- | ------ | -------- | ------------------------------------------------------- |
| message  | String | 是       | 同上                                                    |
| code     | int    | 是       | 同上                                                    |
| data.uid | String | 是       | 该次提现生成的提现记录uid，可通过这个查询到该条提现记录 |

---

### MOL（积分形式）提现记录withdraw/records

这个接口可以被设计成一个通用的、独立的模块，以后所有业务中的撒给用户的币都以积分形式表达，用该接口查询提现记录。

> 地址：https://api.xxx.com/path/v1/withdraw/records

##### 请求头

```
GET /path/v1/withdraw/records
Accept: application/json
Content-Type: application/json;charset=UTF-8
Token: token_G34G34G34G34G35G5
AppVersion: 1.1.1
Platform: Android
```

##### 参数

?keyword=热点&sort=des&page=0&pageSize=20

| params   | 类型   | 是否必须 | 描述 |
| -------- | ------ | -------- | ---- |
| keyword  | String | 否       | 同上 |
| sort     | String | 否       | 同上 |
| page     | int    | 否       | 同上 |
| pageSize | int    | 否       | 同上 |

##### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
Cache-Control: no-store
Pragma: no-cache
```

##### 响应

```
{
    "message": "居然被你查询成功了",
    "code": 200,
    "page": 0,
    "pageSize": 20,
    "first": "https://...",
    "next": "https://...",
    "previous": "https://...",
    "last": "https://...",
    "data": [
        {
            "uid": "235235235",
            "content": "提现到钱包",
            "amount": "+50",
            "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
            "currency": "MOL",
            "time": "2018-04-23 17:38:44",
            "state": "申请中",
            "address": "mol_15tcgzxmwg1d7rs91gc58um8y5jpz85g7iikdemnzimyrm5w6rw7krpeuxec",
            "type": "applying"
        },
        {
            "uid": "235235235",
            "content": "提现到钱包",
            "amount": "+50",
            "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
            "currency": "MOL",
            "time": "2018-04-23 17:38:44",
            "state": "已到账",
            "address": "mol_15tcgzxmwg1d7rs91gc58um8y5jpz85g7iikdemnzimyrm5w6rw7krpeuxec",
            "type": "received"
        }
    ]
}
```

| key           | 类型   | 是否必须 | 描述                                                         |
| ------------- | ------ | -------- | ------------------------------------------------------------ |
| message       | String | 是       | 同上                                                         |
| code          | int    | 是       | 同上                                                         |
| page          | int    | 否       | 同上                                                         |
| pageSize      | int    | 否       | 同上                                                         |
| first         | String | 否       | 同上                                                         |
| next          | String | 否       | 同上                                                         |
| previous      | String | 否       | 同上                                                         |
| last          | String | 否       | 同上                                                         |
| data          | object | 是       | 当前接口的具体数据由该JSON对象承载                           |
| data.uid      | String | 是       | 提现记录主键                                                 |
| data.content  | String | 是       | 提现记录标题名称                                             |
| data.amount   | String | 是       | 提现记录总额                                                 |
| data.image    | String | 否       | 提现记录图标，可以不用这个                                   |
| data.currency | String | 是       | 提现记录币种标识                                             |
| data.time     | String | 是       | 提现记录时间                                                 |
| data.state    | String | 是       | 提现记录状态，数据库改积分数值几乎的秒级的，而从积分到链上，会有一段确认时间，因此需要设计一个状态来给用户展示，两种状态：申请中，已到账 |
| data.address  | String | 是       | 提现地址（提现的收款地址）                                   |
| data.type     | String | 是       | 提现记录状态判断值，根据这个参数来判断当前状态，值为applying表示申请中，值为received表示已到账 |

---

