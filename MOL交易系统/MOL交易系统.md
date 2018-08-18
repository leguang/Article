# MOL交易系统接口

>Version：beta
>
>Author：李勇
>
>E-mail：666233@qq.com
>
>Time：2018.08.01

## 业务简介

本模块设计成一个通用的、独立的模块，以后所有业务中与链进行交互都使用该模块完成，用户提现实际上是我们从链上某个地址打币给用户的地址，充值实际上是用户的地址打币到链上指定账户。

### 项目url预览

![](https://i.imgur.com/PHCHX6N.png) 

站在数据的角度，若想满足各大模块的功能需求，可以将接口分成如下几类：
> base url：https://api.xxx.com/path/

| 分类                         | 接口地址                                   | 参数                                            |
| :--------------------------- | ------------------------------------------ | ----------------------------------------------- |
| 充值chain/deposit            | https://api.xxx.com/path/v1/c              | amount、fromAccount、toAccount、currency、block |
| 提现chain/withdraw           | https://api.xxx.com/path/v1/chain/withdraw | currency、amount                                |
| MOL（积分）记录chain/records | https://api.xxx.com/path/v1/chain/records  | type                                            |

------

#### 充值chain/deposit（从MOL链到MOL积分）

~~该接口用于告诉后台用户已经打币给官方指定地址了，让官方去确认，后台就可以利用hash这个参数结合blocks_info这个action去查询后对比。~~

之后大家一起商议后，认为该方案不合理，有双花风险，因为看了nano的链的代码逻辑：双花的块都会放到链上，但是最后有一个会失效，然后从链上删除。因为考虑到在短时间内是无法通过hash块来确定一定是支付成功的，所以还是得后台收账后，并且5分钟内无误则可认为是真实转账。因此相应界面也得更改：客户端存入成功界面得提示5分钟后确认等字样。

> 地址：https://api.xxx.com/path/v1/chain/deposit

##### 请求头

```
POST /path/v1/chain/deposit
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
    "fromAccount": "mol_114x611xyehnj767bmxia1d67m3kqrtukxtdxkcptf4mzj8sea4ru779i78h",
    "toAccount": "mol_34qdk454gaxhic3w3a9i7pzn3hhkhxpaqgqg7gp3syupyf5gp9mmdsqg5zca",
    "currency": "MOL",
    "block": "{\"type\":\"state\",\"account\":\"mol_15tcgzxmwg1d7rs91gc58um8y5jpz85g7iikdemnzimyrm5w6rw7krpeuxec\",\"previous\":\"65FA210BF9578C8CA0C1F5BECAF2EAE70C2D6B8CFDC839F897E11C6DB6CAE8F3\",\"representative\":\"mol_3ytentj15q44he4c778317r868wdttwufp96fsccjux4tuqc59ojgrwn6d4w\",\"balance\":\"1731194500000000000000000000000\",\"link\":\"0F4A77FB3E380B2E3270394336E66F0E36F986E2C2125B274FC27EC4C7C26385\",\"signature\":\"7310F2E1C0BB32F668796C92CDD564CE88214F6C065615EC821CEE1CFA124A1CFBEE4F0584FD488CCF8AC0AC1EA0E5574FA4D137CB0E2A433956C7839EEB0B04\",\"work\":\"5bdbe26964c621d0\"}"
}
```
| params   | 类型   | 描述                                                         |
| -------- | ------ | ------------------------------------------------------------ |
| amount   | String | 充值总额                                                     |
| from     | String | 用户的打款地址                                               |
| to       | String | 官方的收款地址                                               |
| currency | String | 单位或币种，暂时固定填MOL                                    |
| block    | String | 就是平时转账时，使用process这个action来处理block块时所需要的参数 |

##### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
```

##### 响应
```
{
    "message": "居然被你充值成功了",
    "code": 200
}
```
| key  | 类型   | 描述             |
| ---- | ------ | ---------------- |
| uid  | String | 该条数据的唯一ID |

------

### 提现chain/withdraw（从MOL积分币到MOL链）

这个接口可以被设计成一个通用的、独立的模块，以后所有业务中的撒给用户的币都以积分形式表达，用该接口让用户提现，从MOL积分币换成MOL链上的真实币。

> 地址：https://api.xxx.com/path/v1/chain/withdraw

##### 请求头

```
POST /path/v1/chain/withdraw
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

### MOL（积分形式）提现记录chain/records

这个接口可以被设计成一个通用的、独立的模块，以后所有业务中的撒给用户的币都以积分形式表达，用该接口查询提现记录。

> 地址：https://api.xxx.com/path/v1/chain/records

##### 请求头

```
GET /path/v1/chain/records
Accept: application/json
Content-Type: application/json;charset=UTF-8
Token: token_G34G34G34G34G35G5
AppVersion: 1.1.1
Platform: Android
```

##### 参数

?type=deposit&keyword=热点&sort=des&page=0&pageSize=20

> 此处type这个参数，本来想定义值为send和receive，但是发现这两个词意表达是具有相对方向性的，并无法表达到底是固定哪一方给哪一方打款，因此改用deposit和withdraw。

| params   | 类型   | 是否必须 | 描述                                                         |
| -------- | ------ | -------- | ------------------------------------------------------------ |
| keyword  | String | 否       | 同上                                                         |
| sort     | String | 否       | 同上                                                         |
| page     | int    | 否       | 同上                                                         |
| pageSize | int    | 否       | 同上                                                         |
| type     | String | 否       | 用于区分记录类型，不传表示默认，默认表示获取全部类型的记录，值为deposit表示充值记录（用户地址打款给官方指定地址），值为withdraw表示提现记录（官方指定地址打款给用户地址）。 |

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

