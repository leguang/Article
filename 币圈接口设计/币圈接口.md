# 币圈Http协议REST ful接口设计

>Version：beta
Author：李勇
E-mail:666233@qq.com
只是抛砖引玉，不一定完全正确，需要大家一起商议后决定，由于不知道后台具体的表的设计如何，因此只能猜测着用url定义资源，因此url设计的路径部分会有不合理的地方，所以最好商讨、抽象出一套合理的资源路径。

## 要求
### 目标
* 接口“粒度”争取设计得足够小，争取在业务发生变化后，后台接口不需要增减，只需要前端组合接口仍然能满足新的业务需求。
* 任何一个接口都可以获取到数据，哪怕没传参数。

### 命名规范
一名二姓三风水，四积阴德五读书，名不正则言不顺，言不顺则事难成。软件开发其实就是门命名的艺术，所以首先定义一些规范，提出一些硬性要求，大家在命名的时候尽量多花点心思，多参考优秀的命名风格。
* 强烈推荐参考：[阿里巴巴Java开发手册](https://github.com/leguang/Article/blob/master/阿里巴巴Android开发手册.pdf)，[iOS开发手册](https://www.baidu.com/s?ie=utf-8&f=3&rsv_bp=1&ch=3&tn=98010089_dg&wd=iOS%E5%BC%80%E5%8F%91%E7%BC%96%E7%A0%81%E5%8F%8A%E5%91%BD%E5%90%8D%E8%A7%84%E8%8C%83&oq=ios%25E5%25BC%2580%25E5%258F%2591%25E5%2591%25BD%25E5%2590%258D%25E8%25A7%2584%25E8%258C%2583&rsv_pq=861604950004ebc0&rsv_t=58c07RTbI99q5XYs9eJyxgRtHHUk%2FLDHkwLo0Z86N1kK3GeDd1ktZqvYuIIsSazuXxg&rqlang=cn&rsv_enter=1&inputT=3905&rsv_sug3=17&rsv_sug1=16&rsv_sug7=100&rsv_n=2&bs=ios%E5%BC%80%E5%8F%91%E5%91%BD%E5%90%8D%E8%A7%84%E8%8C%83 "iOS开发编码及命名规范")。
* 一个单词尽量选择5--7个字母的，这样才最优美。
* 首字母缩写的单词尽量每个字母都用大写，例如ID。用个小写，人家还以为是一个单词。当然uri、url、urn这种除外，因为大家都知道这个是什么。
* 规范并统一公司的基础包名与项目的关系。
* 前后端的某些名称概念要统一用某一个单词，比如支付的统一订单，支付宝用的是order，微信用的是unifiedorder，那我们统一对订单这个概念用order这个词。再比如主机：后台用gateway，现在我们统一用host。这单词不统一很容易分裂。

### URI规范
URI 表示资源，资源一般对应服务器端领域模型中的实体类，要求如下：
* 不用大写
* 尽量不用横杠分隔符，万一要用，请使用中杠-不用下杠_
* 参数列表要encode
* URI中的名词表示资源集合，使用复数形式
* 路径仅表示资源的路径（位置），以及一些特殊的actions操作。
* 以复数（名词）进行命名资源，不管返回单个或者多个资源。
* 资源的路径从父到子依次如：/{resource}/{resource_id}/{sub_resource}/{sub_resource_id}/{sub_resource_property}。
* 使用?来进行资源的过滤、搜索以及分页等。
* 使用版本号，且版本号在资源路径之前。
* 优先使用内容协商来区分表述格式，而不是使用后缀来区分表述格式。
* url最好越简短越好，结果过滤，排序，搜索相关的功能都应该通过参数实现。
* url失效则返回404 not found 或 410 gone；对迁移的API，返回 301 重定向。

### JSON规范
暂时只考虑json的数据格式，要求如下：
* 不要使用缩写
* 统一用驼峰命名法
* 不要使用_或者-
* 用名词复数表示集合类型
* 为了方便以后的扩展兼容，如果返回的是数组，强烈建议用一个包含如items属性的对象进行包裹。如：{"items":[{},{}]}
* 建议对每个字段设置默认值（数组型可设置为[],字符串型可设置为””，数值可设置为0，对象可设置为{}）,这一条是为了方便前端/客户端进行判断字段存不存在操作。
* 建议资源使用UUID最为唯一标识。同时建议命名为id或者uid。
* 采用UTF-8编码。
* 数据应该拿来就能用，不应该还要进行转换操作。

>**草稿：json返回的格式是分门别类按对象来划分，还是铺大饼的形式铺开，两者利弊各异，比如url，可能一个接口中返回多个url，如果分json对象装的话，则key都可以叫url，否则的话key就得命名成xxxUrl。这个有待商议**

## 业务简介
本项目是在Telegram上的二次开发，新增了部分业务，竞品对象是币用。

### 业务模块
总体功能模块划分如下图所示：
![模块划分](https://i.imgur.com/jJyWCzo.png)

## Http部分
### 使用场景
* App的初始化数据尽量都用http协议获取。
* 页面的初始化数据尽量都用http协议获取。

### URL结构
```
https://{serviceRoot}/{collection}/{id}
```
- {serviceRoot} – 域名+端口号 (site URL) + 根目录
- {collection} – 要访问的资源
- {id} – 要访问的资源的唯一编号

### 公共请求头
通过Content-Type指定请求与返回的数据格式有json和xml,暂时我们只管json的。其中请求数据还要指定Accept。
```
Accept: application/json
Content-Type: application/json;charset=UTF-8
```
### 公共参数
公共参数是指每一个接口应该传的参数，同时后端要指定公共参数的默认值，**且要保证没有传公共参数不会报错，所以需要一定的容错性，比如priceDes这个参数值，如果是用的是全部小写的，只要是不冲突，则可认为是准确的参数并且表达了按价格降序排列这个语意**。我个人建议公共参数放到请求头里，但最终商议结果是放在请求参数里面，因此定义固定格式如下：
```
{
    "extra": "你想填什么就填什么",
    "token": "token_523523523",
    "page": 0,
    "pageSize": 20,
    "data": {
        "uid": "23523523523523"
		其他个性参数……
    }
}
```

|params | 类型 | 描述 |
| - | - | - |
| extra | String | 额外扩展字段 |
| token | String | 检测权限、标识登录状态 |
| page | int | 页码，不传则默认为0 |
| pageSize | int | 要求每一页返回最大个数，不传则默认为20 |
| data | Json对象 | 需要传的具体个性参数 |

### 个性参数
个性参数就是除了公共参数之外的，看能否考虑统一用json将公共参数和个性参数浓缩成一个参数，把想要表达的参数通过json中的key-value形式传递。例如：

https://api.xxx.com/both/v1/search/products?params={"extra":"你想填什么呢","token":"token_523523523","data":{"uid":"23523523523523"}}
或者考虑与业务相关的参数就用json形式包装，而与业务无关的个性参数就还是用传统的方式另立一个参数。例如：
https://api.xxx.com/both/v1/search/products?limit=10&offset=10&params={"keyword":"方便面","sort":"des"}

### 公共响应头
```
Content-Type:application/json; charset=utf-8
Status:200 OK
```
其中状态码要与公共响应体里的json中的code字段一样。

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
|key | 类型 | 描述 |
| - | - | - |
| message | String | 返回给接口调用者的描述，有可能用于显示到界面上，需要进行国际化处理 |
| code | int | 这个与请求头中的状态码一致，是为了满足部分开发者的习惯 |
| page | int | 分页请求中请求的当前页的页码 |
| pageSize | int | 分页请求中一页的个数，默认为20 |
| first | String | 分页请求中第一页的url ，如果没有则返回空字符串|
| next | String | 分页请求中下一页的url，如果没有则返回空字符串 |
| previous | String | 分页请求中上一页的url，如果没有则返回空字符串 |
| last | String | 分页请求中最后一页的url，如果没有则返回空字符串 |
| data | object | 当前接口的具体数据由该json对象承载 |
| uid | String | **对于每一个资源对象，在返回的时候，都应该返回操作这个资源对象的唯一码** |

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
作为 API 的设计者，正确的将 API 执行结果和失败原因用清晰简洁的方式传达给客户程序是十分关键的一步。 我们确实可以在 HTTP 的相应内容中描述是否成功，如果出错是因为什么，然而，这就意味着用户需要进行内容解析，才知道执行结果和错误原因。因此，HTTP响应代码可以保证客户端在第一时间用最高效的方式获知 API 运行结果，并采取相应动作。下表列出了比较常用的响应代码。
常用的http状态码及使用场景：

| 响应代码 |代码含义 |
| - | - |
| 200 |已创建，请求成功且服务器已创建了新的资源。|
| 201 |是否只显示处于警告状态的应用实例。|
| 301 |重定向 , 请求的网页已被永久移动到新位置。服务器返回此响应时，会自动将请求者转到新位置。|
| 302 |重定向 , 请求的网页临时移动到新位置，但求者应继续使用原有位置来进行以后的请求。302 会自动将请求者转到不同的临时位置。|
| 304 |未修改，自从上次请求后，请求的网页未被修改过。服务器返回此响应时，不会返回网页内容。|
| 400 |错误请求 , 服务器不理解请求的语法。|
| 401 |未授权 , 请求要求进行身份验证。|
| 403 |已禁止 , 服务器拒绝请求。|
| 404 |未找到 , 服务器找不到请求的网页。|
| 405 |方法禁用 , 禁用请求中所指定的方法。|
| 406 |不接受 , 无法使用请求的内容特性来响应请求的网页。|
| 408 |请求超时 , 服务器等候请求时超时。|
| 410 |已删除 , 如果请求的资源已被永久删除，那么，服务器会返回此响应。|
| 412 |未满足前提条件 , 服务器未满足请求者在请求中设置的其中一个前提条件。|
| 415 |不支持的媒体类型 , 请求的格式不受请求页面的支持。|
| 500 |内部服务器错误。|

### 分页
分页适用于GET类型且返回集合数据的请求，根据如下参数进行分页操作。分页返回的数据见公共响应体。
```
{
    "extra": "你想填什么就填什么",
    "token": "token_523523523",
    "page": 0,
    "pageSize": 20,
}
```

| params | 类型 | 描述 |
| - | - | - |
| page | int | 页码 |
| pageSize | int | 分页请求中一页的个数，该值固定,默认为20|

### 项目url预览
![url预览](https://i.imgur.com/ITaMSBX.png)

站在数据的角度，若想满足各大模块的功能需求，可以将接口分成如下几类：
> base url：https://api.xxx.com/both/v1/

| 分类 | 接口 | 参数 |
| - | - | - |
| 注册、用户/个人信息users | 地址：https://api.xxx.com/both/v1/users | keyword、sort等 |
| 鉴权、登录、登出等tokens | 地址：https://api.xxx.com/both/v1/tokens | keyword、category等 |
| 密码管理passwords | 地址：https://api.xxx.com/both/v1/passwords | password、authcode等 |
| 验证码grantCodes | 地址：https://api.xxx.com/both/v1/grantCodes | type等 |
| 类型types | 地址：https://api.xxx.com/both/v1/types | sort、uid、keyword等 |
| 资讯news | 地址：https://api.xxx.com/both/v1/news |uid、keyword、sort等 |
| 群groups | 地址：https://api.xxx.com/both/v1/groups | uid、recommend、newest、sort等|
| 规则rules | 地址：https://api.xxx.com/both/v1/rules | uid、type、keyword、sort等|
| 交易记录trades | 地址：https://api.xxx.com/both/v1/trades | uid、keyword、type等|
| 发现discovery | 地址：https://api.xxx.com/both/v1/discovery |  |
| 钱包wallets | 地址：https://api.xxx.com/both/v1/wallets | uid、keyword等 |

### 登录部分接口
创建token（登录）、删除token（登出）、刷新token和获取token状态（鉴权）。

#### 登录
地址：https://api.xxx.com/both/v1/tokens

###### 请求头

```
POST /both/v1/tokens
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数

- 密码创建token

```
{
    "extra": "你想填什么就填什么",
    "token": "",
    "page": 0,
    "pageSize": 20,
    "data": {
        "account": 18012345678,
        "password": 123456,
        "grantType": "password"
    }
}
```
| key | 类型 | 描述 |
| - | - | - |
| extra | String | 额外扩展字段 |
| token | String | 此处传空或者不传 |
| page | int | 页码，不传则默认为0 |
| pageSize | int | 要求每一页返回最大个数，不传则默认为20 |
| data | Json对象 | 需要传的具体个性参数 |
| data.account | String | 账号 |
| data.password | String | 密码 |
| data.grantType | String | 授权类型，OAuth 2.0中的授权方式，此处填password |

- 短信验证码创建token

```
{
    "extra": "你想填什么就填什么",
    "token": "",
    "page": 0,
    "pageSize": 20,
    "data": {
        "account": 18012345678,
        "grantCode": 123456,
        "grantType": "smsCode"
    }
}
```
| key | 类型 | 描述 |
| - | - | - |
| extra | String | 额外扩展字段 |
| token | String | 此处传空或者不传 |
| page | int | 页码，不传则默认为0 |
| pageSize | int | 要求每一页返回最大个数，不传则默认为20 |
| data | Json对象 | 需要传的具体个性参数 |
| data.account | String | 账号 |
| data.password | String | 密码 |
| data.grantType | String | 授权类型，OAuth 2.0中的授权方式，此处填smsCode |

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
Cache-Control: no-store
Pragma: no-cache
```

###### 响应
```
{
    "message": "居然被你创建了token",
    "code": 200,
    "data": {
        "token": "token_1224124124",
        "expires": 30,
        "user": "这个user信息需不需要在这里给，可以商量"
    }
}
```
|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |
| data.token | String | 获取的token |
| data.expires | int | 期限 |

---

#### 登出
地址：https://api.xxx.com/both/v1/tokens

###### 请求头

```
DELETE /both/v1/tokens
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
```
{
    "extra": "你想填什么就填什么",
    "token": "token_3523235",
    "page": 0,
    "pageSize": 20,
}
```
| key | 类型 | 描述 |
| - | - | - |
| token | String | 用户现在使用的token |

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
Cache-Control: no-store
Pragma: no-cache
```

###### 响应
```
{
    "message": "居然被你删除成功",
    "code": 200,
    "data": {
        "token": "token_1224124124"
    }
}
```
|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |
| data.token | String | 这一次请求成功被删除掉的token |

---

#### 刷新token
对token的期限进行刷新等
地址：https://api.xxx.com/both/v1/tokens

###### 请求头

```
PUT /both/v1/tokens
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
```
{
    "extra": "你想填什么就填什么",
    "token": "token_3523235",
    "page": 0,
    "pageSize": 20,
}
```
| key | 类型 | 描述 |
| - | - | - |
| token | String | 用户现在使用的token |

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
Cache-Control: no-store
Pragma: no-cache
```

###### 响应
```
{
    "message": "居然被你创建了token",
    "code": 200,
    "data": {
        "token": "token_1224124124",
        "expires": 30,
        "user": "这个user信息需不需要在这里给，可以商量"
    }
}
```
|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |
| data.token | String | 获取的token |
| data.expires | int | 期限 |

---

#### 查询token状态
地址：https://api.xxx.com/both/v1/tokens/token_255235235

###### 请求头
```
GET /both/v1/tokens/token_255235235
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
无

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
Cache-Control: no-store
Pragma: no-cache
```

###### 响应
```
{
    "message": "居然被你创建了token",
    "code": 200,
    "data": {
        "token": "token_1224124124",
        "expires": 30,
        "user": "这个user信息需不需要在这里给，可以商量"
    }
}
```

**data对象不为空则表示该token是合法的。**

|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |
| data.token | String | 获取的token |
| data.expires | int | 期限 |

---

### 用户部分接口
注册（新增或者创建用户）、查询用户信息和修改用户信息。

#### 注册/修改用户
由于该项目业务的特殊性，将注册和修改用一个接口，后台对前端传过来的数据，先从数据库中查询，如果有该用户就是修改，如果没有该用户就新增（即注册）。
地址：https://api.xxx.com/both/v1/users

###### 请求头

```
POST /both/v1/users
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
```
{
    "extra": "你想填什么就填什么",
    "token": "",
    "data": {
        "nickname": "一声笑",
        "userID": "123456",
        "firstname": "一",
        "lastname": "声笑"
    }
}
```
参数都不是必传的

| key | 类型 | 描述 |
| - | - | - |
| extra | String | 额外扩展字段 |
| token | String | 此处传空或者不传 |
| data | Json对象 | 需要传的具体个性参数 |
| data.nickname | String | 用户昵称 |
| data.userID | String | 用户唯一ID |
| data.firstname | String | 用户姓 |
| data.lastname | String | 用户名 |

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
Cache-Control: no-store
Pragma: no-cache
```

###### 响应
```
{
    "message": "居然被你创建了token",
    "code": 200,
    "data": {
        "uid": "23523525235"
    }
}
```
|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |
| uid | String | 新增/或者修改的这条用户的主键 |

---

#### 查询用户信息
地址：https://api.xxx.com/both/v1/users/324553534534

###### 请求头

```
GET /both/v1/users/324553534534
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
无

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
Cache-Control: no-store
Pragma: no-cache
```

###### 响应
```
{
    "message": "居然被你创建了token",
    "code": 200,
    "data": {
        "nickname": "一声笑",
        "userID": "123456",
        "firstname": "一",
        "lastname": "声笑"
    }
}
```
| key | 类型 | 描述 |
| - | - | - |
| extra | String | 额外扩展字段 |
| token | String | 此处传空或者不传 |
| data | Json对象 | 需要传的具体个性参数 |
| data.nickname | String | 用户昵称 |
| data.userID | String | 用户唯一ID |
| data.firstname | String | 用户姓 |
| data.lastname | String | 用户名 |

---

### 密码管理部分接口
该接口对用户的密码进行管理，包括修改账户密码和支付密码,由于密码与用户是一对一的关系，新增密码和更新密码是一个语义，因此需要一个接口即可，也就是创建于修改统一用POST。
地址：https://api.xxx.com/both/v1/passwords

###### 请求头

```
POST /both/v1/passwords
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
```
{
    "extra": "你想填什么就填什么",
    "token": "token_235235235",
    "data": {
        "password": "123456",
        "type": "payment",
        "grantType": "sms",
        "grantCode": "1234"
    }
}
```
| key | 类型 | 描述 |
| - | - | - |
| extra | String | 额外扩展字段 |
| token | String | 此处传空或者不传 |
| data | Json对象 | 需要传的具体个性参数 |
| data.password | String | 新密码 |
| data.type | 枚举 | 密码类型，payment：支付密码，user：用户登录密码，我们暂时只有支付密码，此处填payment即可 |
| data.grantType | 枚举 | sms：短信验证，password：密码验证 |
| data.grantCode | String | 短信验证填短信验证码，密码验证填旧密码 |

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
```

###### 响应
```
{
    "message": "居然被你修改成功",
    "code": 200
}
```
|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |

---

### 验证码/授权码
该接口用于获取验证码，如短信验证码，邮箱验证码，在接口设计上并未做
地址：https://api.xxx.com/both/v1/grantCodes

###### 请求头

```
POST /both/v1/grantCodes
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
```
{
    "extra": "你想填什么就填什么",
    "token": "",
    "data": {
        "type": "sms",
        "receiver": "13812345678"
    }
}
```
| key | 类型 | 描述 |
| - | - | - |
| extra | String | 额外扩展字段 |
| token | String | 此处传空或者不传 |
| data | Json对象 | 需要传的具体个性参数 |
| data.type | 枚举 | 验证码类型，sms：短信验证码，email：邮箱验证码 |
| data.receiver | String | 接收方，可以是短信或者邮件地址等 |

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
```

###### 响应
```
{
    "message": "居然被你修改成功",
    "code": 200
}
```
|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |

---

### 规则
该接口用于描述邀请好友奖励等规则，由于只有获取规则，所以只列举了GET。
地址：https://api.xxx.com/both/v1/rules

###### 请求头

```
GET /both/v1/rules
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
```
{
    "extra": "你想填什么就填什么",
    "token": "token_2432344",
    "data": {
        "type": "invitation"
    }
}
```
| key | 类型 | 描述 |
| - | - | - |
| extra | String | 额外扩展字段 |
| token | String | 此处传空或者不传 |
| data | Json对象 | 需要传的具体个性参数 |
| data.type | 枚举 | 规则类型，invitation：表示邀请好友奖励的规则，暂时只有这一个 |

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
```

###### 响应
```
{
    "message": "居然被你修改成功",
    "code": 200,
    "data": [
        {
            "key": "邀请好友",
            "value": "5%"
        },
        {
            "key": "登录",
            "value": "5%"
        }
    ]
}
```
之所以设计成这样是为了方便扩展，其实全部用date里的key也可以，value可以不用管。

|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |
| data | Json对象 | 需要传的具体个性参数 |
| data.key | String | 奖励类目 |
| data.value | String | 奖励值 |

---

### 类型部分接口
该接口用于描述一些分类字段，比如新闻资讯的分类；奖励类型等，由于是只获取，所以只列举了GET。
地址：https://api.xxx.com/both/v1/types

###### 请求头

```
GET /both/v1/types
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
?params={"extra":"你想填什么就填什么","token":"token_2432344","keyword":"group","sort":"des","page":0,"pageSize":20}

| key | 类型 | 描述 |
| - | - | - |
| extra | String | 额外扩展字段 |
| token | String | 此处传空或者不传 |
| keyword | string | 用于过滤的关键字，news:资讯的分类，group:群组分类 |
| sort | 枚举 | 排序，des：降序，asc升序 |
| page | int | 页码，不传则默认为0 |
| pageSize | int | 要求每一页返回最大个数，不传则默认为20 |

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
```

###### 响应
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
            "type": "课程",
            "uid": "2342423233"
        },
        {
            "type": "区块链知识",
            "uid": "3646346434"
        },
        {
            "type": "新闻资讯",
            "uid": "3634634633"
        }
    ]
}
```
之所以设计成这样是为了方便扩展，其实全部用date里的key也可以，value可以不用管。

|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |
| data | Json对象 | 需要传的具体个性参数 |
| data.type | String | 该分类的具体意义名称 |
| data.uid | String | 该分类的主键 |

---

### 资讯部分接口
该接口用于描述一些分类字段，比如新闻资讯的分类；奖励类型等，由于是只获取，所以只列举了GET。


#### 获取资讯列表
地址：https://api.xxx.com/both/v1/news

###### 请求头

```
GET /both/v1/news
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
?params={"extra":"你想填什么就填什么","token":"token_2432344","keyword":"方便面","sort":"des","page":0,"pageSize":20,"data":{"type":"235523532"}}

| key | 类型 | 描述 |
| - | - | - |
| extra | String | 额外扩展字段 |
| token | String | 此处传空或者不传 |
| keyword | string | 用于过滤的关键字 |
| sort | 枚举 | 排序，des：降序，asc:升序 |
| page | int | 页码，不传则默认为0 |
| pageSize | int | 要求每一页返回最大个数，不传则默认为20 |
| data | Object | 请求参数Json对象 |
| data.type | String | 请求咨询类型，默认或者不传则表示获取全部，根据类型接口返回的内容中的uid来传这个参数 |

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
```

###### 响应
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
            "description": "牛牛牛，你牛什么牛，牛牛牛，你牛什么牛，牛牛牛，你牛什么牛",
            "source": "币世界",
            "author": "币老爷",
            "time": "2018-04-23 20:00",
            "url": "https://www.baidu.com/",
            "type": "课程",
            "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg"
        },
        {
            "uid": "2342423233",
            "title": "牛市要来了，赶紧上车",
            "description": "牛牛牛，你牛什么牛，牛牛牛，你牛什么牛，牛牛牛，你牛什么牛",
            "source": "币世界",
            "author": "币老爷",
            "time": "2018-04-23 20:00",
            "url": "https://www.baidu.com/",
            "type": "课程",
            "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg"
        },
        {
            "uid": "2342423233",
            "title": "牛市要来了，赶紧上车",
            "description": "牛牛牛，你牛什么牛，牛牛牛，你牛什么牛，牛牛牛，你牛什么牛",
            "source": "币世界",
            "author": "币老爷",
            "time": "2018-04-23 20:00",
            "url": "https://www.baidu.com/",
            "type": "课程",
            "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg"
        }
    ]
}
```

|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |
| page | int | 分页请求中请求的当前页的页码 |
| pageSize | int | 分页请求中一页的个数，默认为20 |
| first | String | 分页请求中第一页的url ，如果没有则返回空字符串|
| next | String | 分页请求中下一页的url，如果没有则返回空字符串 |
| previous | String | 分页请求中上一页的url，如果没有则返回空字符串 |
| last | String | 分页请求中最后一页的url，如果没有则返回空字符串 |
| data | object | 当前接口的具体数据由该json对象承载 |
| data.uid | String | 该条资讯的唯一标示，可用于获取详情 |
| data.title | String | 文章标题 |
| data.description | String | 文章描述 |
| data.source | String | 文章来源 |
| data.author | String | 文章作者 |
| data.time | String | 时间 |
| data.url | String | 该条资讯web连接 |
| data.type | String | 所属的类型名称，对应着类型接口中的名称 |
| data.image | String | 文章配图 |

---

#### 获取资讯详情
地址：https://api.xxx.com/both/v1/news/34636436
###### 请求头

```
GET /both/v1/news/34636436
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
无

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
```

###### 响应
```
{
    "message": "居然被你查询成功了",
    "code": 200,
    "data": {
        "uid": "2342423233",
        "title": "牛市要来了，赶紧上车",
        "description": "牛牛牛，你牛什么牛，牛牛牛，你牛什么牛，牛牛牛，你牛什么牛",
        "source": "币世界",
        "author": "币老爷",
        "time": "2018-04-23 20:00",
        "url": "https://www.baidu.com/",
        "type": "课程",
        "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg"
    }
}
```
之所以设计成这样是为了方便扩展，其实全部用date里的key也可以，value可以不用管。

|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |
| data | object | 当前接口的具体数据由该json对象承载 |
| data.uid | String | 该条资讯的唯一标示，可用于获取详情 |
| data.title | String | 文章标题 |
| data.description | String | 文章描述 |
| data.source | String | 文章来源 |
| data.author | String | 文章作者 |
| data.time | String | 时间 |
| data.url | String | 该条资讯web连接 |
| data.type | String | 所属的类型名称，对应着类型接口中的名称 |
| data.image | String | 文章配图 |

---

### 群组部分接口

#### 获取群列表
地址：https://api.xxx.com/both/v1/groups

###### 请求头

```
GET /both/v1/groups
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
?params={"extra":"你想填什么就填什么","token":"token_2432344","keyword":"方便面","sort":"recommend","page":0,"pageSize":20,"data":{"type":"recommend"}}

| key | 类型 | 描述 |
| - | - | - |
| extra | String | 额外扩展字段 |
| token | String | 此处传空或者不传 |
| keyword | string | 用于过滤的关键字 |
| sort | 枚举 | 排序，des：降序，asc:升序 |
| page | int | 页码，不传则默认为0 |
| pageSize | int | 要求每一页返回最大个数，不传则默认为20 |
| data | Object | 请求参数Json对象 |
| data.type | String | recommend：热度，newest：新旧 |

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
```

###### 响应
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
            "type": "recommend",
            "weight": "9999",
            "time": "2018-04-23 15:12:05",
            "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
            "name": "MOL链官方群",
            "description": "MOL链官方群是个牛B哄哄的群",
            "memberAmount": "1000",
            "state": "已删除",
            "groupLink": "t.me/molmol",
            "action": {
                "type": "group",
                "param": "https://www.baidu.com/"
            },
            "reward": {
                "isReward": true,
                "remainder": "56322",
                "deadline": "2018-04-23 15:12:05",
                "reward": "10",
                "currency": "mol"
            }
        },
        {
            "uid": "2342423233",
            "type": "recommend",
            "weight": "9999",
            "time": "2018-04-23 15:12:05",
            "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
            "name": "MOL链官方群",
            "description": "MOL链官方群是个牛B哄哄的群",
            "memberAmount": "1000",
            "state": "已删除",
            "action": {
                "type": "group",
                "param": "https://www.baidu.com/"
            },
            "reward": {
                "isReward": true,
                "remainder": "56322",
                "deadline": "2018-04-23 15:12:05",
                "reward": "10",
                "currency": "mol"
            }
        }
    ]
}
```

|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |
| page | int | 分页请求中请求的当前页的页码 |
| pageSize | int | 分页请求中一页的个数，默认为20 |
| first | String | 分页请求中第一页的url ，如果没有则返回空字符串|
| next | String | 分页请求中下一页的url，如果没有则返回空字符串 |
| previous | String | 分页请求中上一页的url，如果没有则返回空字符串 |
| last | String | 分页请求中最后一页的url，如果没有则返回空字符串 |
| data | object | 当前接口的具体数据由该json对象承载 |
| data.uid | String | 该群组的主键 |
| data.type | 枚举 | recommend：热度，newest：新旧  |
| data.weight | String | 权重 |
| data.time | String | 申请时间 |
| data.image | String | 群头像 |
| data.name | String | 群名称 |
| data.description | String | 群描述 |
| data.memberAmount | String | 群成员总数 |
| data.state | 枚举 | deleted：已删除 |
| data.action | Object | 跳转动作 |
| data.action.type | String | 跳转类型，group：加群，web：跳转到网页 |
| data.action.param | String | 跳转所需参数，如加群连接，web连接 |
| data.reward.isReward | String | 是否奖励 |
| data.reward.remainder | String | 剩余奖励总数 |
| data.reward.deadline | String | 奖励截止时间 |
| data.reward.reward | String | 单个奖励币数量 |
| data.reward.currency | String | 奖励币种 |

---

#### 获取群详情
地址：https://api.xxx.com/both/v1/groups/23532523
后面省略

---

#### 申请（上传）推荐群（允许批量操作）
地址：https://api.xxx.com/both/v1/groups

###### 请求头

```
POST /both/v1/groups
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
```
{
    "extra": "扩展字段，你想填什么就填什么",
    "token": "token_312331",
    "data": [
        {
            "time": "2018-04-23 15:12:05",
            "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
            "name": "MOL链官方群",
            "groupLink": "t.me/molmol",
            "memberAmount": "1000",
            "reward": {
                "remainder": "56322",
                "deadline": "2018-04-23 15:12:05",
                "reward": "10",
                "currency": "mol"
            }
        },
        {
            "time": "2018-04-23 15:12:05",
            "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
            "name": "MOL链官方群",
            "groupLink": "t.me/molmol",
            "memberAmount": "1000",
            "reward": {
                "remainder": "56322",
                "deadline": "2018-04-23 15:12:05",
                "reward": "10",
                "currency": "mol"
            }
        }
    ]
}
```

|key | 类型 | 描述 |
| - | - | - |
| extra | String | 扩展字段 |
| token | String | 权限标志 |
| data | object | 当前接口的具体数据由该json对象承载 |
| data.uid | String | 该群组的主键 |
| data.time | String | 申请时间 |
| data.image | String | 群头像 |
| data.name | String | 群名称 |
| data.memberAmount | String | 群成员总数 |
| data.groupLink | String | 文章配图 |
| data.reward.remainder | String | 剩余奖励总数 |
| data.reward.deadline | String | 奖励截止时间 |
| data.reward.reward | String | 单个奖励币数量 |
| data.reward.currency | String | 奖励币种 |

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
```

###### 响应
```
{
    "message": "居然被你上传成功了",
    "code": 200,
    "data": [
        {
            "uid": "2342423233"
        },
        {
            "uid": "2342423233"
        }
    ]
}
```

|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |
| data | object | 当前接口的具体数据由该json对象承载 |
| data.uid | String | 上传成功的群组的主键 |

---
#### 钱包部分接口

#### 获取钱包列表
地址：https://api.xxx.com/both/v1/wallets

###### 请求头

```
GET /both/v1/wallets
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
?params={"extra":"你想填什么就填什么","token":"token_2432344","keyword":"摩尔币","sort":"des","page":0,"pageSize":20,"data":{"currency":"CNY"}}

| key | 类型 | 描述 |
| - | - | - |
| extra | String | 额外扩展字段 |
| token | String | 此处传空或者不传 |
| keyword | string | 用于过滤的关键字 |
| sort | 枚举 | 排序，des：降序，asc:升序 |
| page | int | 页码，不传则默认为0 |
| pageSize | int | 要求每一页返回最大个数，不传则默认为20 |
| data | Object | 请求参数Json对象 |
| data.currency | String | 请求折算的币种 |

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
```

###### 响应
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
            "name": "MOL币钱包",
            "amount": "565.0989",
            "currency": "MOL",
            "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
            "worth": {
                "price": "0.12",
                "value": "0.24",
                "currency": "￥ "
            }
        },
        {
            "uid": "235235235",
            "name": "以太币钱包",
            "amount": "565.0989",
            "currency": "MOL",
            "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
            "worth": {
                "price": "0.12",
                "value": "0.24",
                "currency": "￥ "
            }
        }
    ]
}
```

|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |
| page | int | 分页请求中请求的当前页的页码 |
| pageSize | int | 分页请求中一页的个数，默认为20 |
| first | String | 分页请求中第一页的url ，如果没有则返回空字符串|
| next | String | 分页请求中下一页的url，如果没有则返回空字符串 |
| previous | String | 分页请求中上一页的url，如果没有则返回空字符串 |
| last | String | 分页请求中最后一页的url，如果没有则返回空字符串 |
| data | object | 当前接口的具体数据由该json对象承载 |
| data.uid | String | 钱包主键 |
| data.name | String | 该币钱包名称 |
| data.amount | String | 该币总数 |
| data.currency | String | 该币符号 |
| data.image | String | 该币图标 |
| data.worth.price | String | 该币单价 |
| data.worth.value | String | 用户所拥有总价值 |
| data.worth.currency | 枚举 | 对应的币种符号 |
---

#### 获取钱包详情
地址：https://api.xxx.com/both/v1/wallets/23423423
后面省略

---
### 交易部分接口

#### 获取交易记录列表
地址：https://api.xxx.com/both/v1/trades

###### 请求头

```
GET /both/v1/trades
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
?params={"extra":"你想填什么就填什么","token":"token_2432344","keyword":"摩尔币","sort":"des","page":0,"pageSize":20,"data":{"category":"invitation"}}

| key | 类型 | 描述 |
| - | - | - |
| extra | String | 额外扩展字段 |
| token | String | 此处传空或者不传 |
| keyword | string | 用于过滤的关键字 |
| sort | 枚举 | 排序，des：降序，asc:升序 |
| page | int | 页码，不传则默认为0 |
| pageSize | int | 要求每一页返回最大个数，不传则默认为20 |
| data | Object | 请求参数Json对象 |
| data.category | String | 交易记录的类型，默认或者不传则表示获取全部，invitation0：邀请好友的奖励（其中0表示自己这一级，同理1表示自己的好友邀请好友注册的奖励），register：自己注册奖励，signin：自己签到奖励 |

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
```

###### 响应
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

|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |
| page | int | 分页请求中请求的当前页的页码 |
| pageSize | int | 分页请求中一页的个数，默认为20 |
| first | String | 分页请求中第一页的url ，如果没有则返回空字符串|
| next | String | 分页请求中下一页的url，如果没有则返回空字符串 |
| previous | String | 分页请求中上一页的url，如果没有则返回空字符串 |
| last | String | 分页请求中最后一页的url，如果没有则返回空字符串 |
| data | object | 当前接口的具体数据由该json对象承载 |
| data.uid | String | 该条交易主键 |
| data.title | String | 交易标题名称 |
| data.description | String | 交易描述 |
| data.amount | String | 该笔交易金额 |
| data.image | String | 交易图标 |
| data.currency | String | 交易币种 |
| data.time | String | 交易时间 |

---
#### 获取交易记录详情
地址：https://api.xxx.com/both/v1/trades/234324
后面省略

---

### 发现页部分接口
本来该接口是不需要的，可以通过多个接口拼凑出我们想要的数据，但是考虑到性能问题，设计出一个集合接口来。
地址：https://api.xxx.com/both/v1/discovery

###### 请求头

```
GET /both/v1/discovery
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
无

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
```

###### 响应
```
{
    "message": "居然被你查询成功了",
    "code": 200,
    "data": {
        "ads": [
            {
                "image": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                "title": "安防小卫士",
                "uid": "13212133313",
                "action": {
                    "type": "web",
                    "param": "https://www.baidu.com/"
                }
            },
            {
                "image": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                "title": "安防小卫士",
                "uid": "13212133313",
                "action": {
                    "type": "web",
                    "param": "https://www.baidu.com/"
                }
            },
            {
                "image": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                "title": "安防小卫士",
                "uid": "13212133313",
                "action": {
                    "type": "web",
                    "param": "https://www.baidu.com/"
                }
            }
        ],
        "recommendations": {
            "title": "热门推荐群",
            "groups": [
                {
                    "uid": "2342423233",
                    "type": "recommend",
                    "weight": "9999",
                    "time": "2018-04-23 15:12:05",
                    "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
                    "name": "MOL链官方群",
                    "description": "MOL链官方群是个牛B哄哄的群",
                    "memberAmount": "1000",
                    "state": "已删除",
                    "reward": {
                        "isReward": true,
                        "remainder": "56322",
                        "deadline": "2018-04-23 15:12:05",
                        "reward": "10",
                        "currency": "mol"
                    },
                    "action": {
                        "type": "web",
                        "param": "t.me/molmol"
                    }
                },
                {
                    "uid": "2342423233",
                    "type": "recommend",
                    "weight": "9999",
                    "time": "2018-04-23 15:12:05",
                    "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
                    "name": "MOL链官方群",
                    "description": "MOL链官方群是个牛B哄哄的群",
                    "memberAmount": "1000",
                    "state": "已删除",
                    "reward": {
                        "isReward": true,
                        "remainder": "56322",
                        "deadline": "2018-04-23 15:12:05",
                        "reward": "10",
                        "currency": "mol"
                    },
                    "action": {
                        "type": "web",
                        "param": "t.me/molmol"
                    }
                }
            ]
        },
        "discoverys": [
            {
                "uid": "32523523",
                "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
                "title": "币圈咨询",
                "action": {
                    "type": "web",
                    "param": "https://www.baidu.com/"
                }
            },
            {
                "uid": "32523523",
                "image": "http://a3.peoplecdn.cn/fbcba40035ae5f2ad90c19abe58560a2.jpg",
                "title": "币圈咨询",
                "action": {
                    "type": "web",
                    "param": "https://www.baidu.com/"
                }
            }
        ]
    }
}
```

|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |
| data | object | 当前接口的具体数据由该json对象承载 |
| data.ads | Object数组 | 广告对象 |
| data.ads.image | String | 轮播图图片链接 |
| data.ads.title | String | 轮播图描述 |
| data.ads.uid | String | 该广告主键 |
| data.ads.action | Object | 动作对象，用于描述跳转这类的动作 |
| data.ads.action.type | String | 动作类型，web：跳转到web |
| data.ads.action.param | String | 执行该动作所需要的参数，例如跳转web的链接 |
| data.recommendations | Object | 广告对象 |
| data.recommendations.title | String | 该分类的标题，比如此处的“热门群推荐” |
| data.recommendations.groups | Object数组 | 描述推荐的群列表 |
| data.recommendations.groups.uid | String | 该群组的主键 |
| data.recommendations.groups.type | 枚举 | recommend：热度，newest：新旧 |
| data.recommendations.groups.weight | String | 权重 |
| data.recommendations.groups.time | String | 申请时间 |
| data.recommendations.groups.image | String | 群头像 |
| data.recommendations.groups.name | String | 群名称 |
| data.recommendations.groups.description | String | 群描述 |
| data.recommendations.groups.memberAmount | String | 群成员总数 |
| data.recommendations.groups.state | 枚举 | deleted：已删除 |
| data.recommendations.groups.action | Object | 跳转动作 |
| data.recommendations.groups.action.type | String | 跳转类型，group：加群，web：跳转到网页 |
| data.recommendations.groups.action.param | String | 跳转所需参数，如加群连接，web连接 |
| data.recommendations.groups.reward.isReward | String | 是否奖励 |
| data.recommendations.groups.reward.remainder | String | 剩余奖励总数 |
| data.recommendations.groups.reward.deadline | String | 奖励截止时间 |
| data.recommendations.groups.reward.reward | String | 单个奖励币数量 |
| data.recommendations.groups.reward.currency | String | 奖励币种 |
| data.items | Object数组 | 功能项数组 |
| data.items.uid | String | 该功能项的主键 |
| data.items.image | String | 该功能项的图标 |
| data.items.title | String | 该功能项的标题 |
| data.items.groups.action | Object | 跳转动作 |
| data.items.groups.action.type | String | 跳转类型，group：加群，web：跳转到网页 |
| data.items.groups.action.param | String | 跳转所需参数，如加群连接，web连接 |

---

#### 代理服务器部分接口
该接口用于获取sokcs5服务器列表
地址：https://api.xxx.com/both/v1/servers 

###### 请求头

```
GET /both/v1/servers
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
?params={"extra":"你想填什么就填什么","token":"token_2432344"}

| key | 类型 | 描述 |
| - | - | - |
| extra | String | 额外扩展字段 |
| token | String | 此处传空或者不传 |

###### 响应头

```
HTTP/1.1 200 OK
Content-Type:application/json; charset=utf-8
```

###### 响应
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
            "password": "",
            "port": "10497",
            "ip": "13.251.28.110",
            "username": ""
        },
        {
            "password": "",
            "port": "10497",
            "ip": "13.251.28.110",
            "username": ""
        },
        {
            "password": "",
            "port": "10497",
            "ip": "13.251.28.110",
            "username": ""
        }
    ]
}
```

|key | 类型 | 描述 |
| - | - | - |
| message | String | 随便写，也可以作为服务器给前端固定提示的内容 |
| code | int | 与响应头里的Status一样 |
| page | int | 分页请求中请求的当前页的页码 |
| pageSize | int | 分页请求中一页的个数，默认为20 |
| first | String | 分页请求中第一页的url ，如果没有则返回空字符串|
| next | String | 分页请求中下一页的url，如果没有则返回空字符串 |
| previous | String | 分页请求中上一页的url，如果没有则返回空字符串 |
| last | String | 分页请求中最后一页的url，如果没有则返回空字符串 |
| data | object | 当前接口的具体数据由该json对象承载 |
| data.password | String | 代理服务器密码 |
| data.port | String | 代理服务器端口号 |
| data.ip | String | 代理服务器IP地址 |
| data.user | String | 代理服务器用户名 |

---

## 错误/异常处理

* 不要发生了错误但给2XX响应，客户端可能会缓存成功的http请求；
* 正确设置http状态码，不要自定义；
* Response body 提供 1) 错误的代码（日志/问题追查）；2) 错误的描述文本（展示给用户）。

对第三点的实现稍微多说一点：

Java 服务器端一般用异常表示 RESTful API 的错误。API 可能抛出两类异常：业务异常和非业务异常。业务异常由自己的业务代码抛出，表示一个用例的前置条件不满足、业务规则冲突等，比如参数校验不通过、权限校验失败。非业务类异常表示不在预期内的问题，通常由类库、框架抛出，或由于自己的代码逻辑错误导致，比如数据库连接失败、空指针异常、除0错误等等。

业务类异常必须提供2种信息：

1. 如果抛出该类异常，HTTP 响应状态码应该设成什么；
2. 异常的文本描述；

```
错误描述
{
    "message": "又特么错了",
    "code": 500,
    "document": "https://developer.xxx.com/v1/errors/500",
    "exception": [
        {
            "code": "NullValue",
            "target": "PhoneNumber",
            "message": "Phone number must not be null"
        },
        {
            "code": "NullValue",
            "target": "LastName",
            "message": "Last name must not be null"
        },
        {
            "code": "MalformedValue",
            "target": "Address",
            "message": "Address is not valid"
        }
    ]
}
```

```
错误请求头
{
  "date": "Mon, 08 Jan 2018 03:07:08 GMT",
  "server": "nginx/1.10.3 (Ubuntu)",
  "connection": "keep-alive",
  "content-length": "237",
  "content-type": "application/json"
}
```

## 接口版本（Versioning）
个人倾向于将版本号放在HTTP头信息中，虽然不如放入URL中更直观，但是不方便我们统一管理，因为在前端URL是拼出来的String，请求头是统一个对象去设置，除非有特殊情况，某一个接口需1.0版本，某一个接口需2.0版本，这就另当别论，到时候统一商量，在拼这个URL的时候，放到固定目录（位置），如：api.xxx.com:8080/both<u>**/版本（一般用v1、v2）/**</u>user  统一放在一级目录，这样的前端在拼接的时候，统一放到某个位置，也就方便管理了。

## URL失效
随着系统发展，总有一些API失效或者迁移，对失效的API，返回404 not found 或 410 gone；对迁移的API，返回 301 重定向。

## 对于后台文档的要求
![文档要求](https://i.imgur.com/aQ3qb4f.png)
文档要求描述详尽，尽可能的引导接口使用者理解接口设计，这样才能减少接口的改动，又能适应多变的业务。

## 安全
待定

## 统计、历史记录等埋点业务
待定

