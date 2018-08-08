# 新闻模块REST ful接口设计

>Version：beta
>
>Author：李勇
>
>E-mail:666233@qq.com
>
>Time:2018.08.01

## 业务简介

本模块是针对钱包业务的扩充，希望用户能通过钱包活动获取币，增加流通。

### 业务模块

总体功能模块划分如下图所示：

![模块](https://i.imgur.com/CZ971KO.png) 

### URL结构

```
https://{serviceRoot}/{module}/{collection}/{id}
```
- {serviceRoot} – 域名+端口号 (site URL) + 根目录
- {module} – 模块名称
- {collection} – 要访问的资源
- {id} – 要访问的资源的唯一编号

### 公共请求头
通过Content-Type指定请求与返回的数据格式有json和xml,暂时我们只管json的。其中请求数据还要指定Accept。其中额外添加的请求头里的参数注意大小写。
```
Accept: application/json
Content-Type: application/json;charset=UTF-8
Token: token_G34G34G34G34G35G5
AppVersion: 1.1.1
Platform: Android
```

| params     | 类型   | 是否必须 | 描述                                   |
| ---------- | ------ | -------- | -------------------------------------- |
| Token      | String | 是       | 这个不用解释了吧？                     |
| AppVersion | String | 是       | App当前版本的版本名称                  |
| Platform   | String | 是       | Android表示Android平台，iOS表示iOS平台 |

### 公共参数（部分公共参数建议公放到请求头里）

公共参数是指每一个接口都可能有的，为了减少篇幅，我在这里统一定义，同时后端要指定公共参数的默认值，**且要保证没有传公共参数不会报错，所以需要一定的容错性，比如priceDes这个参数值，如果是用的是全部小写的，只要是不冲突，则可认为是准确的参数并且表达了按价格降序排列这个语意**。常用公共参数如下：

| params   | 类型   | 是否必须 | 描述                                                         |
| -------- | ------ | -------- | ------------------------------------------------------------ |
| keyword  | String | 否       | 用于检索，不传或者传空，表示默认，默认不检索该关键字         |
| sort     | String | 否       | 用于对列表排序，不传则表示默认，默认按配置顺序，asc升序，desc降序 |
| page     | int    | 否       | 用于分页描述，不传则表示默认，默认是第1页                    |
| pageSize | int    | 否       | 表示该页显示的条数，不传则表示默认，默认为20条               |

### 个性参数

~~个性参数就是除了公共参数之外的，看能否考虑统一用JSON将公共参数和个性参数浓缩成一个参数，把想要表达的参数通过json中的key-value形式传递。例如：https://api.xxx.com/both/v1/search/products?params={"extra":"你想填什么呢","token":"token_523523523","data":{"uid":"23523523523523"}} 或者考虑与业务相关的参数就用json形式包装，而与业务无关的个性参数就还是用传统的方式另立一个参数。例如：https://api.xxx.com/both/v1/search/products?limit=10&offset=10&params={"keyword":"方便面","sort":"des"}~~

通过商议将个性参数统一按原始的URL传参形式传递，不考虑上面所说的那种方式。

### 公共响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
```
### 公共响应体

默认会有以下字段，不需要全部都有。
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
    "data": {
        "uid": "6565656565665"
    }
}
```
| key      | 类型   | 是否必须 | 描述                                                         |
| -------- | ------ | -------- | ------------------------------------------------------------ |
| message  | String | 是       | 返回给接口调用者的描述，有可能用于显示到界面上，需要进行国际化处理 |
| code     | int    | 是       | 这个与请求头中的状态码一致，是为了满足部分开发者的习惯       |
| page     | int    | 否       | 分页请求中请求的当前页的页码                                 |
| pageSize | int    | 否       | 分页请求中一页的个数，默认为20                               |
| first    | String | 否       | 分页请求中第一页的url ，如果没有则返回空字符串               |
| next     | String | 否       | 分页请求中下一页的url，如果没有则返回空字符串                |
| previous | String | 否       | 分页请求中上一页的url，如果没有则返回空字符串                |
| last     | String | 否       | 分页请求中最后一页的url，如果没有则返回空字符串              |
| data     | object | 否       | 当前接口的具体数据由该json对象承载                           |
| uid      | String | 否       | **对于每一个资源对象，在返回的时候，都应该返回操作这个资源对象的唯一码** |

### HTTP动词表示操作。
常用的HTTP动词有下面五个（括号里是对应的SQL命令）。
* GET（SELECT）：从服务器取出资源（一项或多项）。
  例如:GET /zoos：列出所有动物园。
* POST（CREATE）：在服务器新建一个资源。
  例如:POST /zoos：新建一个动物园。
* PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
  例如:PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）。
* DELETE（DELETE）：从服务器删除资源。
  例如:DELETE /zoos/ID：删除某个动物园。

### 状态码
作为API的设计者，正确的将API执行结果和失败原因用清晰简洁的方式传达给客户程序是十分关键的一步。我们确实可以在HTTP的相应内容中描述是否成功，如果出错是因为什么，然而，这就意味着用户需要进行内容解析，才知道执行结果和错误原因。因此，HTTP响应代码可以保证客户端在第一时间用最高效的方式获知 API 运行结果，并采取相应动作。下表列出了比较常用的响应代码。
常用的http状态码及使用场景：

| 响应代码 | 代码含义                                                     |
| -------- | ------------------------------------------------------------ |
| 200      | 已创建，请求成功且服务器已创建了新的资源。                   |
| 201      | 是否只显示处于警告状态的应用实例。                           |
| 301      | 重定向 , 请求的网页已被永久移动到新位置。服务器返回此响应时，会自动将请求者转到新位置。 |
| 302      | 重定向 , 请求的网页临时移动到新位置，但求者应继续使用原有位置来进行以后的请求。302 会自动将请求者转到不同的临时位置。 |
| 304      | 未修改，自从上次请求后，请求的网页未被修改过。服务器返回此响应时，不会返回网页内容。 |
| 400      | 错误请求 , 服务器不理解请求的语法。                          |
| 401      | 未授权 , 请求要求进行身份验证。                              |
| 403      | 已禁止 , 服务器拒绝请求。                                    |
| 404      | 未找到 , 服务器找不到请求的网页。                            |
| 405      | 方法禁用 , 禁用请求中所指定的方法。                          |
| 406      | 不接受 , 无法使用请求的内容特性来响应请求的网页。            |
| 408      | 请求超时 , 服务器等候请求时超时。                            |
| 410      | 已删除 , 如果请求的资源已被永久删除，那么，服务器会返回此响应。 |
| 412      | 未满足前提条件 , 服务器未满足请求者在请求中设置的其中一个前提条件。 |
| 415      | 不支持的媒体类型 , 请求的格式不受请求页面的支持。            |
| 500      | 内部服务器错误。                                             |

### 分页
分页适用于GET类型且返回集合数据的请求，根据如下参数进行分页操作。分页返回的数据见公共响应体。
```
{
    "extra": "你想填什么就填什么",
    "page": 0,
    "pageSize": 20,
}
```

| params   | 类型 | 是否必须 | 描述 |
| -------- | ---- | -------- | ---- |
| page     | int  | 否       | 同上 |
| pageSize | int  | 否       | 同上 |

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

### 主题（分类）topics

针对新闻进行分类的接口，对应一般新闻页面中顶部的分类列表。

> 地址：https://api.xxx.com/path/v1/topics

##### 请求头

```
GET /path/v1/topics
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
    "message": "居然被你看到了",
    "code": 200,
    "data": [
        {
            "uid": "32t23t23sdvsdbse324h34h",
            "key": "",
            "value": "推荐"
        },
        {
            "uid": "32t23t23sdvsdbse324h34h",
            "key": "",
            "value": "体育"
        },
        {
            "uid": "32t23t23sdvsdbse324h34h",
            "key": "",
            "value": "科技"
        },
        {
            "uid": "32t23t23sdvsdbse324h34h",
            "key": "",
            "value": "教育"
        },
        {
            "uid": "32t23t23sdvsdbse324h34h",
            "key": "",
            "value": "本地"
        }
    ]
}
```
| key     | 类型   | 是否必须 | 描述                           |
| ------- | ------ | -------- | ------------------------------ |
| message | String | 是       | 同上                           |
| code    | int    | 是       | 同上                           |
| uid     | String | 是       | 该主题的唯一识别               |
| key     | String | 是       | 用于标记该主题，**暂时用不到** |
| value   | String | 是       | 该主题的显示文本               |

---

### 新闻列表+广告news 

由于新闻列表中会夹杂广告，因此把广告当新闻处理。

> 地址：https://api.xxx.com/path/v1/news

##### 请求头

```
GET /path/v1/news
Accept: application/json
Content-Type: application/json;charset=UTF-8
Token: token_G34G34G34G34G35G5
AppVersion: 1.1.1
Platform: Android
```

##### 参数

?topic=23mksd9ssdpoi0weg&keyword=热点&sort=des&page=0&pageSize=20

| params   | 类型   | 是否必须 | 描述                                                         |
| -------- | ------ | -------- | ------------------------------------------------------------ |
| topic    | String | 否       | 获取某一个主题（分类），对应上文topics接口中的uid，不传或者传空字符表示默认值，默认返回可能的所有新闻 |
| keyword  | String | 否       | 同上                                                         |
| sort     | String | 否       | 同上                                                         |
| page     | int    | 否       | 同上                                                         |
| pageSize | int    | 否       | 同上                                                         |

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
            "uid": "2342423233",
            "title": "牛市要来了，赶紧上车",
            "type": "image=0",
            "abstract": "牛牛牛，你牛什么牛，牛牛牛，你牛什么牛，牛牛牛，你牛什么牛",
            "source": "币世界",
            "author": "币老爷",
            "time": "2018-04-23 20:00",
            "url": "https://www.baidu.com/",
            "topic": "课程",
            "images": [
                "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
                "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
                "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg"
            ],
            "share": "https://app.mol.one/news/qfr23r23r23r2r3?url=http://toutiao.com/group/6536299962081214990/",
            "like": {
                "isLike": true,
                "number": "200"
            }
        },
        {
            "uid": "2342423233",
            "title": "牛市要来了，赶紧上车",
            "type": "image=0",
            "abstract": "牛牛牛，你牛什么牛，牛牛牛，你牛什么牛，牛牛牛，你牛什么牛",
            "source": "币世界",
            "author": "币老爷",
            "time": "2018-04-23 20:00",
            "url": "https://www.baidu.com/",
            "topic": "课程",
            "images": [
                "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
                "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
                "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg"
            ],
            "share": "https://app.mol.one/news/qfr23r23r23r2r3?url=http://toutiao.com/group/6536299962081214990/",
            "like": {
                "isLike": true,
                "number": "200"
            }
        }
    ]
}
```

| key              | 类型     | 是否必须 | 描述                                                         |
| ---------------- | -------- | -------- | ------------------------------------------------------------ |
| message          | String   | 是       | 同上                                                         |
| code             | int      | 是       | 同上                                                         |
| page             | int      | 否       | 同上                                                         |
| pageSize         | int      | 否       | 同上                                                         |
| first            | String   | 否       | 同上                                                         |
| next             | String   | 否       | 同上                                                         |
| previous         | String   | 否       | 同上                                                         |
| last             | String   | 否       | 同上                                                         |
| data             | object   | 是       | 当前接口的具体数据由该json对象承载                           |
| data.uid         | String   | 是       | 该条资讯的唯一标示，可用于获取详情                           |
| data.title       | String   | 是       | 文章标题                                                     |
| data.type        | String   | 是       | 表达该item的类型，其实是标记样式类型的，我们规定每一个item都有且只有4中样式，详细查看设计稿。值为0表示无图片的形式，值为1表示左边只有一张图片的形式，值为2表示只有一张大图片形式（通常为video的预览图），值为3表示3张小图片形式 |
| data.description | String   | 是       | 文章描述                                                     |
| data.source      | String   | 是       | 文章来源                                                     |
| data.author      | String   | 是       | 文章作者                                                     |
| data.time        | String   | 是       | 时间                                                         |
| data.url         | String   | 是       | 该条资讯web连接                                              |
| data.type        | String   | 是       | 所属的类型名称，对应着类型接口中的名称                       |
| data.images      | JSON数组 | 是       | 文章配图，图片个数与type这个类型规定的一致                   |
| data.share       | String   | 是       | 分享的url，url=https://www.baidu.com/ 参数是真实的今日头条对应的url。 |
| data.like        | JSON对象 | 是       | 用于表示点赞数和点赞状态                                     |
| data.like.isLike | boolean  | 是       | true表示已点赞，false表示未点赞                              |
| data.like.number | String   | 是       | 点赞数                                                       |

---

### 点赞like 

此处的点赞是要消耗摩尔币，优先消耗未提现的部分，如果未提现部分不够，则code返回某一个值，当前端收到该值则走支付10MOL的流程。这里是不允许取消点赞的，因为此时可能已经分了后来人的钱。

> 地址：https://api.xxx.com/path/v1/like

##### 请求头

```
POST /path/v1/like
Accept: application/json
Content-Type: application/json;charset=UTF-8
Token: token_G34G34G34G34G35G5
AppVersion: 1.1.1
Platform: Android
```

##### 参数

?newsUid=13r13rt1312412er12r1&isLike=true

| params | 类型    | 是否必须 | 描述                                                         |
| ------ | ------- | -------- | ------------------------------------------------------------ |
| uid    | String  | 是       | 新闻item的uid                                                |
| isLike | boolean | 否       | 不传表示默认，默认表示点赞，值为true表示点赞，false表示取消点赞 |

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
        "uid": "2342423233"
    }
}
```

| key      | 类型   | 是否必须 | 描述                               |
| -------- | ------ | -------- | ---------------------------------- |
| message  | String | 是       | 同上                               |
| code     | int    | 是       | 同上                               |
| data     | object | 是       | 当前接口的具体数据由该json对象承载 |
| data.uid | String | 是       | 所操作的那个新闻item的uid          |

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
| data             | object | 是       | 当前接口的具体数据由该json对象承载 |
| data.uid         | String | 是       | 交易记录主键                       |
| data.title       | String | 是       | 交易记录标题名称                   |
| data.description | String | 是       | 交易记录描述                       |
| data.amount      | String | 是       | 交易记录金额                       |
| data.image       | String | 是       | 交易记录图标                       |
| data.currency    | String | 是       | 交易记录币种                       |
| data.time        | String | 是       | 交易记录时间                       |

------

## 实现方案

### 新闻来源

新闻来源由欧阳通过分析得出一个结论是直接用今日头条之前的一个域名接口

> http://m.365yg.com/list/?tag=news_military&as=A125BB5695B10A9&format=json_raw 

其中参数部分欧阳要具体列出来，并标明意思。于是整体思路如下：

1. 后台通过该接口获取到如下格式数据（由于太长了，我只截取其中某一个新闻的JSON）

   ```
   {
       "media_name": "央视网新闻",
       "ban_comment": 0,
       "abstract": "“人民是历史的创造者，人民是真正的英雄。从乡村振兴到脱贫攻坚，从高质量发展到科技创新，从贯彻新发展理念到实战化练兵。",
       "image_list": [],
       "datetime": "2018-03-25 17:05",
       "article_type": 0,
       "tag": "news_politics",
       "has_m3u8_video": 0,
       "keywords": "人民军队,两会,习近平,四梁八柱,中华民族",
       "display_dt": 1521850927,
       "has_mp4_video": 0,
       "aggr_type": 1,
       "cell_type": 0,
       "article_sub_type": 0,
       "bury_count": 0,
       "title": "习近平两会“话中画”",
       "source_icon_style": 6,
       "tip": 0,
       "has_video": false,
       "share_url": "http://m.toutiao.com/a6536299962081214990/?iid=0&app=news_article",
       "label": "置顶",
       "source": "央视网新闻",
       "comment_count": 101,
       "article_url": "http://toutiao.com/group/6536299962081214990/",
       "publish_time": 1521850927,
       "group_flags": 0,
       "middle_mode": true,
       "has_image": true,
       "action_extra": "{\"channel_id\": 0}",
       "tag_id": "6536299962081214990",
       "source_url": "/i6536299962081214990/",
       "display_url": "http://toutiao.com/group/6536299962081214990/",
       "is_stick": true,
       "item_id": "6536299962081214990",
       "repin_count": 31024,
       "cell_flag": 262155,
       "source_open_url": "sslocal://profile?uid=50025817786",
       "level": 0,
       "digg_count": 9,
       "behot_time": 1521968742,
       "hot": 0,
       "cursor": 1521968742999,
       "url": "http://toutiao.com/group/6536299962081214990/",
       "image_url": "https://p3.pstatp.com/list/15218509162387765f09457",
       "user_repin": 0,
       "label_style": 6,
       "video_style": 0,
       "media_info": {
           "avatar_url": "http://p1.pstatp.com/large/bc20000b91968707dab",
           "media_id": 50044041847,
           "name": "央视网新闻",
           "user_verified": true
       },
       "group_id": "6536299962081214990",
       "gallary_image_count": 3
   }
   ```

2. 后台拿到数据后转成新闻列表接口中要求的格式，然后返回给前端。

这么做的原因是需要做容错机制，因为考虑到该接口不是永远都能用，也没有经过检验，因此我们还要考虑其他方案，万一不能用了，起码不至于强制前端更新App。

### 阅读奖励的请求

阅读一段时间会有奖励，此时就要告诉后台请求奖励，考虑安全因素，因此使用长连接来发送该消息，即多一个消息类型，如下所示：

##### 请求

```
{
    "uid": "A3BA3BA3BA3BC",
    "uri": "v1/requestNewsAwards",
    "data": {
        "uid": "3r213r23r23r142151234"
    }
}
```

| params   | 类型   | 是否必须 | 描述                                      |
| -------- | ------ | -------- | ----------------------------------------- |
| uid      | String | 是       | 该条消息的uid                             |
| uri      | String | 是       | 该消息类型的标识                          |
| data.uid | String | 是       | 该条新闻的uid，告诉后台是对哪条新闻的奖励 |

同时一个用户一天能通过阅读获取的MOL币的个数是有上限的，以及本人针对该文章能获取的币也是有上限的，这两个上限是配置化。本来想将这种消息类型做成通用的，即请求多少奖励通过参数传给后台，这样未来其他项目就只要直接改奖励的MOL的个数，但是考虑到安全和可控等因素，还是独立一个uri类型了。

##### 响应

```
{
    "message": "恭喜你，连接成功",
    "code": 200,
    "uid": "A3BA3BA3BA3BC",
    "uri": "v1/requestNewsAwards",
    "data": {
        "amount": "1",
        "currency": "MOL"
    }
}
```
### 分享的链接实现

由于我们分享的链接中的数据本质不是我们自己的，连HTML都是别人写好了的，但是想嵌入我们自己的业务逻辑代码，因此分享给用户点击的url必须是mol.one域名下的：

> https://mol.one/news/grwefgwq423t2t23?url=https://www.baidu.com

该url中news路径下对应的是熊老师写的程序，将 https://www.baidu.com 的内容显示在里面，同时底部加一个领取红包的按钮。