# 电商项目RESTful接口设计思考

>主要依赖HTTP+MQTT完成整个电商项目的前后端通讯和协作。当然只是抛砖引玉，不一定完全正确，需要大家一起商议后决定，由于不知道后台具体的表的设计如何，因此只能猜测着用url定义资源，因此url设计的路径部分会有不合理的地方，所以最好商讨、抽象出一套合理的资源路径，要注意这一点,可参考** [github接口](https://developer.github.com/v3/ "github接口") **和** [淘宝Api](http://open.taobao.com/doc2/apiList.htm "淘宝Api") **的设计。

## 要求
### 目标
* 接口“粒度”争取设计得足够小，争取在业务发生变化后，后台接口不需要增减，只需要前端组合接口仍然能满足新的业务需求。
* 任何一个接口都可以获取到数据，哪怕没传参数。

### 命名规范
一名二姓三风水，四积阴德五读书，名不正则言不顺，言不顺则事难成。软件开发其实就是门命名的艺术，所以首先定义一些规范，提出一些硬性要求，大家在命名的时候尽量多花点心思，多参考优秀的命名风格。
* 强烈推荐参考：[参考阿里巴巴Java开发手册](https://github.com/leguang/Article "阿里巴巴Java开发手册")，[iOS开发手册](https://www.baidu.com/s?ie=utf-8&f=3&rsv_bp=1&ch=3&tn=98010089_dg&wd=iOS%E5%BC%80%E5%8F%91%E7%BC%96%E7%A0%81%E5%8F%8A%E5%91%BD%E5%90%8D%E8%A7%84%E8%8C%83&oq=ios%25E5%25BC%2580%25E5%258F%2591%25E5%2591%25BD%25E5%2590%258D%25E8%25A7%2584%25E8%258C%2583&rsv_pq=861604950004ebc0&rsv_t=58c07RTbI99q5XYs9eJyxgRtHHUk%2FLDHkwLo0Z86N1kK3GeDd1ktZqvYuIIsSazuXxg&rqlang=cn&rsv_enter=1&inputT=3905&rsv_sug3=17&rsv_sug1=16&rsv_sug7=100&rsv_n=2&bs=ios%E5%BC%80%E5%8F%91%E5%91%BD%E5%90%8D%E8%A7%84%E8%8C%83 "iOS开发编码及命名规范")。
* 一个单词尽量选择5--7个字母的，这样才最优美。
* 首字母缩写的单词尽量每个字母都用大写，例如ID。用个小写，人家还以为是一个单词。当然uri、url、urn这种除外，因为大家都知道这个是什么。
* 规范我们公司的基础包名与项目的关系，看需不需要前后端统一一下，现在的亿社区是：com.aglhz.yicommunity，智能家居还是用美伦安保为蓝本：com.meilun.security.smart。还有很多细节需要完善啊，现在不改等到后面引用的地方越来越多，比如地图，支付、社会化分享等，这些虽然改不改无所谓，但是会让后来人引起不必要的误会，尤其是像我这种有强迫症的人，完全难以忍受。
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
本项目是我司的一次电商重新探索，之前的宅宜购项目的试探未成功，此次在其基础上重新探索，希望做成嵌入式的商城，嵌入到其他项目或者能独立运行，承载着我司的电商渠道，为销售我司智能硬件和外包打下基础。

### 业务模块
总体参考了主流电商的功能后划分出7大部分，如下图所示：
![模块划分](https://i.imgur.com/ids8hGR.png)

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
>Accept: application/json
>Content-Type: application/json;charset=UTF-8

### 公共参数（建议公共参数放到请求头里）
公共参数是指每一个接口应该传的参数，同时后天要指定公共参数的默认值，**且要保证没有传公共参数不会报错，所以需要一定的容错性，比如priceDes这个参数值，如果是用的是全部小写的，只要是不冲突，则可认为是准确的参数并且表达了按价格降序排列这个语意**。

公共参数的位置有以下几种：
* 拼在url后

```
https://xxx.com/products/token=token_G34G34G34G34G35G5
```

* 拼在请求体中

```
{
    "token": "token_G34G34G34G34G35G5",
	"fromPoint": "YsqApp"
	其他参数……
}
```
* 放在请求头里

```
    token: token_G34G34G34G34G35G5
	"fromPoint": "YsqApp"
	其他参数……
```

|params | 类型 | 描述 |
| - | -| - |
|token | String | 检测权限、标识登录状态 |
|fromPoint|String|用于区分该次请求到底是从哪个App发起的，以方便后台区分|

|fromPoint |  对应项目 |
| - | - |
|亿社区 |  YsqApp |
|宅宜购 |  MallApp |
|美好生活 |  YsqMeilunApp |
|威士丹利社区 |  YsqVensiApp |
|高诚智能家居 |  SmartApp |
|美伦安保 |  SmartMeilunApp |
|猫眼 | SmartCateyeApp |

### 个性参数（该方案待定）
个性参数就是除了公共参数之外的，看能否考虑统一用json浓缩成一个参数，把想要表达的参数通过json中的key-value形式传递。
> 例如：https://xxx.com/ec/v1/search/products?params={keyword:方便面,order:des}
这种方式我暂时还不确定是否合理，或者考虑与业务相关的参数就用json形式包装，而与业务无关的个性参数就还是用传统的方式另立一个参数。
> 例如：https://xxx.com/ec/v1/search/products?limit=10&offset=10&params={keyword:方便面,order:des}

### 公共响应头
其中状态码要与公共响应体里的json中的code字段一样。

### 公共响应体
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
| - | -| -|
|message | String | 返回给接口调用者的描述，有可能用于显示到界面上，需要进行国际化处理 |
|code | int | 这个与请求头中的状态码一致，是为了满足部分开发者的习惯 |
|page | int | 分页请求中请求的当前页的页码 |
|pageSize | int | 分页请求中一页的个数，默认为20 |
|first | String | 分页请求中第一页的url ，如果没有则返回空字符串|
|next | String | 分页请求中下一页的url，如果没有则返回空字符串 |
|previous | String | 分页请求中上一页的url，如果没有则返回空字符串 |
|last | String | 分页请求中最后一页的url，如果没有则返回空字符串 |
|data | object | 当前接口的具体数据由该json对象承载 |
|uid | String | **对于每一个资源对象，在返回的时候，都应该返回操作这个资源对象的唯一码** |

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

|响应代码|代码含义|
| - | -|
|200|已创建，请求成功且服务器已创建了新的资源。|
|201|是否只显示处于警告状态的应用实例。|
|301|重定向 , 请求的网页已被永久移动到新位置。服务器返回此响应时，会自动将请求者转到新位置。|
|302|重定向 , 请求的网页临时移动到新位置，但求者应继续使用原有位置来进行以后的请求。302 会自动将请求者转到不同的临时位置。|
|304|未修改，自从上次请求后，请求的网页未被修改过。服务器返回此响应时，不会返回网页内容。|
|400|错误请求 , 服务器不理解请求的语法。|
|401|未授权 , 请求要求进行身份验证。|
|403|已禁止 , 服务器拒绝请求。|
|404|未找到 , 服务器找不到请求的网页。|
|405|方法禁用 , 禁用请求中所指定的方法。|
|406|不接受 , 无法使用请求的内容特性来响应请求的网页。|
|408|请求超时 , 服务器等候请求时超时。|
|410|已删除 , 如果请求的资源已被永久删除，那么，服务器会返回此响应。|
|412|未满足前提条件 , 服务器未满足请求者在请求中设置的其中一个前提条件。|
|415|不支持的媒体类型 , 请求的格式不受请求页面的支持。|
|500|内部服务器错误。|

### 电商项目url预览
![url预览](https://i.imgur.com/3VKI6Oy.png)

站在数据的角度，若想满足7大模块的功能需求，可以将接口分成如下几类：
* base url：https://xxx.com/ec/v1/
* 注册登录（暂不考虑）

|分类|接口|参数|
| - | -| - |
|搜索search|https://xxx.com/ec/v1/search/{搜索分类：如products}|keyword、order等|
|首页home|https://xxx.com/ec/v1/home|keyword、category等|
|配送delivery|https://xxx.com/ec/v1/delivery/{配送uid}|keyword、order、name、address、phone等|
|分类（或者叫“更多”）category|https://xxx.com/ec/v1/category/{种类名称或者id，包括具体的种类和子种类}|keyword、order等|
|订单order|https://xxx.com/ec/v1/order/{订单uid}|keyword、sort、order、productID等|
|商品product|https://xxx.com/ec/v1/products/{具体商品uid}|keyword、category、order等
|店铺shop|https://xxx.com/ec/v1/shop/{店铺uid}|keyword、order等|
|关键字keyword|https://xxx.com/ec/v1/keyword/{具体关键字}|keyword、order等|
|购物车cart|https://xxx.com/ec/v1/cart/{购物车id}|keyword、order等|

### ~~登录注册部分接口~~
~~暂不考虑~~

### 关键字部分
提取关键字只需要“增删改查”中的“查”，所以只有GET
> 例如：https://xxx.com/ec/v1/keyword?params={keyword:方便面,order:des}
不传参数则获取热门搜索，或者推荐搜索，这个由于界面上限制的是最多9个，所以不需要分页操作。

###### 请求头

```
GET /ec/v1/keyword
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数

>?params={keyword:方便面,order:des}

|params | 类型 | 描述 |
| - | -| -|
|keyword | String | 查询的关键字 |
|sort | String | 排序参数，不传或者default-->综合排序，des-->降序，asc-->升序，priceDes-->价格从高到低，priceAsc-->价格从低到高，costDes-->总价从高到低，costAsc-->总价从低到高|

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
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
            "query": "方便面",
            "keyword": "方便面的危害",
            "extra": "10"
        },
        {
            "query": "方便面",
            "keyword": "方便面批发市场",
            "extra": "10"
        },
        {
            "query": "方便面",
            "keyword": "方便面生产线",
            "extra": "10"
        },
        {
            "query": "方便面",
            "keyword": "方便面怎么煮好吃",
            "extra": "10"
        }
    ]
}
```
|key | 类型 | 描述 |
| - | -| -|
|keyword | String | 返回匹配的关键字，显示到界面上 |
|extra | int | 作为扩展字段，比如返回当前关键字的商品个数 |

### 搜索部分
提取关键字只需要“增删改查”中的“查”，所以只有GET
>由于智能硬件和便利店搜索的内容不一样，但是数据格式一样，所以：
智能硬件：https://xxx.com/ec/v1/search/smarthome?params={keyword:方便面,order:des}
便利店/商场/超市：https://xxx.com/ec/v1/search/shop?params={keyword:方便面,order:des}

如果未来统一的搜索是商品，那接口应为：https://xxx.com/ec/v1/search/products?params={keyword:方便面,order:des}
还可以扩展一下，比如搜索用户，搜索店铺等。不传参数是不允许的。

###### 请求头

```
GET /ec/v1/search/products
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数

>?params={keyword:方便面,sort:des}

|params | 类型 | 描述 |
| - | -| -|
|keyword | String | 查询的关键字 |


###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
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
            "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
            "detailUrl": "https://item.jd.com/4264502.html",
            "title": "安防小卫士",
            "description": "wifi/电话双网 您的智能小卫士",
            "uid": "13212133313",
            "type": "smarthome",
            "price": "589.0",
            "currency": "¥"
        },
        {
            "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
            "detailUrl": "https://item.jd.com/4264502.html",
            "title": "安防小卫士",
            "description": "wifi/电话双网 您的智能小卫士",
            "uid": "13212133313",
            "type": "smarthome",
            "price": "589.0",
            "currency": "¥"
        },
        {
            "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
            "detailUrl": "https://item.jd.com/4264502.html",
            "title": "安防小卫士",
            "description": "wifi/电话双网 您的智能小卫士",
            "uid": "13212133313",
            "type": "smarthome",
            "price": "589.0",
            "currency": "¥"
        }
    ]
}
```
|key | 类型 | 描述 |
| - | -| -|
|imageUrl | String | 该商品缩略图url |
|detailUrl | String | 跳转到该商品详情页的web url|
|name | String | 该商品的名称|
|description | String | 对商品的简单描述 |
|uid | String | 该商品唯一识别id |
|type | String | 表示当前商品的类型：智能家居smarthome、便利店/超市/商场shop |
|price | String | 价格 |
|currency | String | 标识币种，可以是符号，也可以是文字，看前后端的需求，也可以再立一个字段表示 |

### 递送部分
递送部分接口有“增删改查”
>地址：https://xxx.com/ec/v1/deliveries/{递送uid}

#### 新增一条或者多条递送
>新增一条递送：https://xxx.com/ec/v1/deliveries/{递送uid}
>新增多条递送：https://xxx.com/ec/v1/deliveries

###### 请求头

```
POST /ec/v1/deliveries
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
如果是新增一条，data中则不需要传数组，只需要传一个对象就行了。
```
{
    "message": "上传这几个地址",
    "data": [
        {
            "name": "BeJson",
            "gender": "male",
            "phoneNumber": "13888888888",
            "location": "凯宾斯基",
            "address": "C栋801",
            "longitude": "85.66",
            "latitude": "36.33",
			"isDefault": true
        },
        {
            "name": "BeJson",
            "gender": "male",
            "phoneNumber": "13888888888",
            "location": "凯宾斯基",
            "address": "C栋801",
            "longitude": "85.66",
            "latitude": "36.33",
			"isDefault": true
        },
        {
            "name": "BeJson",
            "gender": "male",
            "phoneNumber": "13888888888",
            "location": "凯宾斯基",
            "address": "C栋801",
            "longitude": "85.66",
            "latitude": "36.33",
			"isDefault": true
        }
    ]
}
```

|params | 类型 | 描述 |
| - | -| -|
|name | String | 新增的这个收货人的姓名 |
|gender | String | 性别：只能取male或者female或者secrecy，默认是不用选择性别的，也允许保存成功，没选性别就是secrecy |
|phoneNumber | String | 电话号码 |
|location | String | 定位地址，只是粗略地址 |
|address | String | 详细地址 |
|longitude | String | 经度，用于后台搜索店铺使用 |
|latitude | String | 纬度，用于后台搜索店铺使用 |

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
```

###### 响应
```
{
    "message": "居然被你保存成功了",
    "code": 200,
    "data": [
        {
            "uid": "1333644113313131"
        },
        {
            "uid": "1333644113313131"
        },
        {
            "uid": "1333644113313131"
        }
    ]
}
```

#### 删除一条或者多条递送
* 删除一条：https://xxx.com/ec/v1/deliveries/{递送uid}
* 删除多条：https://xxx.com/ec/v1/deliveries

###### 请求头

```
DELETE /ec/v1/deliveries
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
删除一条则不需要参数，因为url中已经包含了地址uid，删除多条则需要以下参数。
```
{
    "message": "删除这几个递送",
    "data": [
        {
            "uid": "1333644113313131"
        },
        {
            "uid": "1333644113313131"
        },
        {
            "uid": "1333644113313131"
        }
    ]
}
```

|params | 类型 | 描述 |
| - | -| -|
|uid | String | 表示某条递送信息的主键 |

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
```

###### 响应
```
{
    "message": "居然被你删除成功了",
    "code": 200,
    "data": [
        {
            "uid": "1333644113313131"
        },
        {
            "uid": "1333644113313131"
        },
        {
            "uid": "1333644113313131"
        }
    ]
}
```
|params | 类型 | 描述 |
| - | -| -|
|uid | String | 表示已删除递送信息的主键 |

#### 修改一条或者多条递送
* 修改一条：https://xxx.com/ec/v1/deliveries/{递送uid}
* 修改多条：https://xxx.com/ec/v1/deliveries
###### 请求头

```
PUT /ec/v1/address
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
如果是修改某一条可以不传uid，因为url中已经包含uid，如果是修改多条则传一个json数组，数组中的每一个对象都得包含uid。
```
{
    "message": "修改这几个递送",
    "data": [
        {
            "uid": "655656133131313",
            "name": "BeJson",
            "gender": "male",
            "phoneNumber": "13888888888",
            "location": "凯宾斯基",
            "address": "C栋801",
            "longitude": "85.66",
            "latitude": "36.33",
            "isDefault": true
        }
    ]
}
```

|params | 类型 | 描述 |
| - | -| -|
|name | String | 新增的这个收货人的姓名 |
|gender | String | 性别：只能取male或者female或者secrecy，默认是不用选择性别的，也允许保存成功，没选性别就是secrecy |
|phoneNumber | String | 电话号码 |
|location | String | 定位地址，只是粗略地址 |
|uid | String | 地址主键 |
|address | String | 详细地址 |
|longitude | String | 经度，用于后台搜索店铺使用 |
|latitude | String | 纬度，用于后台搜索店铺使用 |

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
```

###### 响应
```
{
    "message": "居然被你修改成功了",
    "code": 200,
    "data": [
        {
            "uid": "1333644113313131"
        }
    ]
}
```
|params | 类型 | 描述 |
| - | -| -|
|uid | String | 表示已修改递送信息的主键 |

#### 查询一条或者多条递送
* 查询一条：https://xxx.com/ec/v1/deliveries/{递送uid}
* 查询多条：https://xxx.com/ec/v1/deliveries
###### 请求头

```
GET /ec/v1/delivery
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
>?params={keyword:方便面,sort:des}
如果是查询某一条递送时，由于uid已经在url上，所以返回一条指定递送数据。
|sort | String | 排序参数，不传或者default-->综合排序，des-->降序，asc-->升序，priceDes-->价格从高到低，priceAsc-->价格从低到高，costDes-->总价从高到低，costAsc-->总价从低到高|

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
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
            "uid": "655656133131313",
            "name": "BeJson",
            "gender": "male",
            "phoneNumber": "13888888888",
            "location": "凯宾斯基",
            "address": "C栋801",
            "longitude": "85.66",
            "latitude": "36.33",
            "isDefault": true
        },
        {
            "uid": "655656133131313",
            "name": "BeJson",
            "gender": "male",
            "phoneNumber": "13888888888",
            "location": "凯宾斯基",
            "address": "C栋801",
            "longitude": "85.66",
            "latitude": "36.33",
            "isDefault": false
        },
        {
            "uid": "655656133131313",
            "name": "BeJson",
            "gender": "male",
            "phoneNumber": "13888888888",
            "location": "凯宾斯基",
            "address": "C栋801",
            "longitude": "85.66",
            "latitude": "36.33",
            "isDefault": false
        }
    ]
}
```

|params | 类型 | 描述 |
| - | -| -|
|name | String | 新增的这个收货人的姓名 |
|gender | String | 性别：只能取male或者female或者secrecy，默认是不用选择性别的，也允许保存成功，没选性别就是secrecy |
|phoneNumber | String | 电话号码 |
|location | String | 定位地址，只是粗略地址 |
|uid | String | 地址主键 |
|address | String | 详细地址 |
|longitude | String | 经度，用于后台搜索店铺使用 |
|latitude | String | 纬度，用于后台搜索店铺使用 |
|isDefault | boolean | 用于表达是否为默认地址 |

### 订单部分
订单部分接口有“增删改查”中的增、删、查。
> 地址：https://xxx.com/ec/v1/orders/{订单uid}

#### 新增一条或者多条订单
> 新增一条订单：https://xxx.com/ec/v1/orders
> 新增多条订单：https://xxx.com/ec/v1/orders/{订单uid}

###### 请求头

```
POST /ec/v1/orders
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
如果是只新增一条，则data中不需要传数组。

```
{
    "message": "上传这几个订单",
    "data": [
        {
            "uid": "6666565656",
            "note": "带一个勺子",
            "products": [
                {
                    "uid": "13212133313",
                    "amount": "5"
                },
                {
                    "uid": "13212133313",
                    "amount": "5"
                }
            ]
        },
        {
            "uid": "6666565656",
            "note": "带一个勺子",
            "products": [
                {
                    "uid": "13212133313",
                    "amount": "5"
                },
                {
                    "uid": "13212133313",
                    "amount": "5"
                }
            ]
        }
    ]
}
```

|params | 类型 | 描述 |
| - | -| -|
|外层uid | String | 递送信息的主键，由这个ID可以查询递送信息，包括人的信息和地址信息 |
|note | String | 留言 |
|uid | String | 商品主键 |
|amount | String | 商品个数 |

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
```

###### 响应
此处返回支付的统一订单信息。
**这里有待商榷，看你们是想在这一步就生成统一订单去支付，还是先上传成功后，返回订单id，然后再用这个订单id去请求支付接口。**

```
{
    "message": "居然被你保存成功了",
    "code": 200,
    "data": {
       返回支付信息还是订单信息有待商榷
    }
}
```
以下为我暂时考虑的
```
{
    "message": "居然被你保存成功了",
    "code": 200,
    "data": [
        {
            "uid": "33331313"
        },
        {
            "uid": "33331313"
        }
    ]
}
```
|params | 类型 | 描述 |
| - | -| -|
|uid | String | 返回刚保存的订单主键，根据这个主键可以查询该订单的详细信息 |

#### 删除一条或者多条订单
> 删除一条订单：https://xxx.com/ec/v1/orders
> 删除多条订单：https://xxx.com/ec/v1/orders/{订单uid}

###### 请求头

```
DELETE /ec/v1/orders
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
如果是删除某一个，则不需要参数，只需要接口改成https://xxx.com/ec/v1/orders/{订单uid}即可。
```
{
    "message": "删除这几个订单",
    "data": [
        {
            "uid": "13212133313"
        },
        {
            "uid": "13212133313"
        }
    ]
}
```

|params | 类型 | 描述 |
| - | -| -|
|uid | String | 订单主键 |

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
```

###### 响应
```
{
    "message": "居然被你删除成功了",
    "code": 200,
    "data": [
        {
            "uid": "33331313"
        },
        {
            "uid": "33331313"
        }
    ]
}
```
| params | 类型 | 描述 |
| - | -| - |
|uid | String | 已经删除的订单主键 |

#### 修改一条或者多条订单
**由于订单是不能修改内容，只能修改状态分类的，即由一种状态修改成另一种状态，比如从“待付款”状态修改成“已取消”状态，因此只传这个category字段即可**
> 修改一条订单：https://xxx.com/ec/v1/orders
> 修改多条订单：https://xxx.com/ec/v1/orders/{订单uid}

###### 请求头

```
PUT /ec/v1/orders
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
如果是修改某一个，只需要接口改成https://xxx.com/ec/v1/orders/{订单uid}即可。
```
{
    "message": "修改这几个订单的状态",
    "data": [
        {
            "category": "323532235",
            "uid": "13212133313"
        },
        {
            "category": "323532235",
            "uid": "13212133313"
        }
    ]
}
```

|params | 类型 | 描述 |
| - | -| -|
|uid | String | 订单主键 |
|category | String | 要修改的状态分类uid |

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
```

###### 响应
```
{
    "message": "居然被你修改成功了",
    "code": 200,
    "data": [
        {
            "category": "已取消",
            "uid": "13212133313"
        },
        {
            "category": "已删除",
            "uid": "13212133313"
        }
    ]
}
```
| params | 类型 | 描述 |
| - | -| - |
|uid | String | 已经修改的订单主键 |
|category | String | 修改成功后的状态分类uid |

#### 查询一条订单
> 查询多条https://xxx.com/ec/v1/orders

###### 请求头

```
GET /ec/v1/orders
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
不传参数表达查询全部。
```
?params={category:23552355}
```

|params | 类型 | 描述 |
| - | -| -|
|category | String | 默认不传表示获取全部订单，**待付款、已取消、待收货、已完成、退款/售后**等类型的uid，通过分类接口获取 |
|sort | String | 排序参数，不传或者default-->综合排序，des-->降序，asc-->升序，priceDes-->价格从高到低，priceAsc-->价格从低到高，costDes-->总价从高到低，costAsc-->总价从低到高|

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
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
            "uid": "43433331313",
            "deliveryType": "送货上门",
            "amount": "4",
            "category": "待付款",
            "pay": {
                "cost": "10",
                "discount": "10",
                "price": "10",
                "currency": "¥"
            },
            "actions": [
                {
                    "action": "取消订单",
                    "type": "cancel",
                    "category": "2353552352",
                    "link": ""
                },
                {
                    "action": "去付款",
                    "category": "2353343535",
                    "type": "pay",
                    "link": ""
                },
                {
                    "action": "删除订单",
                    "category": "",
                    "type": "delete",
                    "link": ""
                }
            ],
            "shop": {
                "name": "克拉家园店",
                "type": "shop",
                "cartUid": "235353552352",
                "uid": "54545454545"
            },
            "products": [
                {
                    "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                    "detailUrl": "https://item.jd.com/4264502.html",
                    "title": "优乐美奶茶",
                    "description": "wifi/电话双网 您的智能小卫士",
                    "uid": "13212133313"
                },
                {
                    "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                    "detailUrl": "https://item.jd.com/4264502.html",
                    "title": "优乐美奶茶",
                    "description": "wifi/电话双网 您的智能小卫士",
                    "uid": "13212133313"
                }
            ]
        }
    ]
}
```
|params | 类型 | 描述 |
| - | -| -|
|uid | String | 该订单主键 |
|amount | String | 该订单中包含的商品个数 |
|cost | String | 该订单总共需要付款数 |
|imageUrl | String | 该商品缩略图url |
|detailUrl | String | 跳转到该商品详情页的web url|
|description | String | 对商品的简单描述 |
|deliveryType|String|送货上门、快递服务|
|actions|action数组|用户描述对该订单的操作，对应界面上的每条订单上的“删除订单、去付款、取消订单”等按钮|
|action|对象|category订单状态的分类uid，表达对应的action操作所需要的参数的值。比如待付款状态要取消这条订单，则在订单的update接口中，category参数传这个uid|
|type|String|针对订单的操作总共5种，其type值如下：1.去付款--pay；2.取消订单（cancel）；3.确认收货（receipt）；4.查看物流（logistics）；5.删除订单（delete）|
|type|String|如果是点击“查看物流”，则需要用到这个字段，跳转网页|

#### 查询一条订单（订单详情）
> 查询一条https://xxx.com/ec/v1/orders/{订单uid}

###### 请求头

```
GET /ec/v1/orders/34634634
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
无

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
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
    "data": {
        "uid": "43433331313",
        "deliveryType": "送货上门",
        "amount": "4",
        "category": "待付款",
        "orderNumber": "64614646464",
        "time": "2018-03-12 15:33:24",
        "note": "带一包烟上来",
        "pay": {
            "cost": "10",
            "discount": "10",
            "price": "10",
            "currency": "¥"
        },
        "actions": [
            {
                "action": "取消订单",
                "type": "cancel",
                "category": "2353552352",
                "link": ""
            },
            {
                "action": "去付款",
                "category": "2353343535",
                "type": "pay",
                "link": ""
            },
            {
                "action": "删除订单",
                "category": "",
                "type": "delete",
                "link": ""
            }
        ],
        "shop": {
            "name": "克拉家园店",
            "type": "shop",
            "cartUid": "235353552352",
            "uid": "54545454545"
        },
        "delivery": {
            "name": "黄沙",
            "gender": "male",
            "phoneNumber": "13888888888",
            "location": "凯宾斯基",
            "address": "C栋801",
            "longitude": "85.66",
            "latitude": "36.33"
        },
        "products": [
            {
                "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                "detailUrl": "https://item.jd.com/4264502.html",
                "title": "优乐美奶茶",
                "description": "wifi/电话双网 您的智能小卫士",
                "uid": "13212133313",
                "amount": "2",
                "pay": {
                    "cost": "4.125",
                    "discount": "7.5",
                    "price": "5.5",
                    "currency": "¥"
                }
            },
            {
                "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                "detailUrl": "https://item.jd.com/4264502.html",
                "title": "优乐美奶茶",
                "description": "wifi/电话双网 您的智能小卫士",
                "uid": "13212133313",
                "amount": "2",
                "pay": {
                    "cost": "4.125",
                    "discount": "7.5",
                    "price": "5.5",
                    "currency": "¥"
                }
            }
        ],
        "express": {
            "name": "小明同志",
            "phoneNumber": "13811112222",
            "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
            "identity": "派送员"
        }
    }
}
```
|params | 类型 | 描述 |
| - | -| -|
|uid | String | 该订单主键 |
|amount | String | 该订单中包含的商品个数 |
|cost | String | 该订单总共需要付款数 |
|imageUrl | String | 该商品缩略图url |
|detailUrl | String | 跳转到该商品详情页的web url|
|description | String | 对商品的简单描述 |
|deliveryType|String|送货上门、快递服务|
|note|String|下单时的备注|
|time|String|下单时间|
|orderNumber|String|订单号|
|actions|action数组|用户描述对该订单的操作，对应界面上的每条订单上的“删除订单、去付款、取消订单”等按钮|
|action|对象|category订单状态的分类uid，表达对应的action操作所需要的参数的值。比如待付款状态要取消这条订单，则在订单的update接口中，category参数传这个uid|
|type|String|针对订单的操作总共5种，其type值如下：1.去付款--pay；2.取消订单（cancel）；3.确认收货（receipt）；4.查看物流（logistics）；5.删除订单（delete）|
|type|String|如果是点击“查看物流”，则需要用到这个字段，跳转网页|

### 分类部分
由于这个分类接口是一个抽象的获取字符串的接口，因此我把商品的分类和订单的状态分类都归于这个接口。分类部分接口只需要“增删改查”中的“查”，所以只有GET，其他的我就不设计了。

#### 查询分类
> 地址：https://xxx.com/ec/v1/categories/{分类uid}

###### 请求头

```
GET /ec/v1/categories
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
不传参数则默认是返回一级分类。
>?params={type:orders,uid:1313113,keyword:方便面,order:des}

|params | 类型 | 描述 |
| - | -| - |
|uid | String | 某分类主键 |
|type | String |orders表示查询的订单状态分类，products表示查询商品分类|
|keyword | String | 查询的关键字 |
|sort | String | 排序参数，不传或者default-->综合排序，des-->降序，asc-->升序，priceDes-->价格从高到低，priceAsc-->价格从低到高，costDes-->总价从高到低，costAsc-->总价从低到高|

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
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
            "parentUid": "25235235235235",
            "category": "智能主机",
            "uid": "516165654656",
            "children": [
                {
                    "category": "智能门锁",
                    "uid": "516165654656"
                },
                {
                    "category": "智能门锁",
                    "uid": "516165654656"
                }
            ]
        },
        {
            "parentUid": "25235235235235",
            "category": "智能主机",
            "uid": "516165654656",
            "children": [
                {
                    "category": "智能门锁",
                    "uid": "516165654656"
                },
                {
                    "category": "智能门锁",
                    "uid": "516165654656"
                }
            ]
        }
    ]
}
```
|key | 类型 | 描述 |
| - | -| -|
|category | String | 种类名称 |
|uid | String | 种类主键，**外层json中的uid表示其父分类的uid，如果是一级分类，则其父分类uid可以为空字符串** |
|parentUid | String | 如果没有父分类则可以为空 |
|children | String数组 | 如果没有子分类则返回为空|

### 商品（产品）部分
商品部分接口只需要“增删改查”中的“查”，所以只有GET
> 例如：https://xxx.com/ec/v1/products/{商品uid}

#### 商品列表
> 例如：https://xxx.com/ec/v1/products
###### 请求头

```
GET /ec/v1/products
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
不传参数则默认返回的是根据系统推荐的产品，对应的业务是“为您推荐”。
>?params={category:1313113}

|params | 类型 | 描述 |
| - | - | - |
|category | String | 某分类主键 |
|sort | String | 排序参数，不传或者default-->综合排序，des-->降序，asc-->升序，priceDes-->价格从高到低，priceAsc-->价格从低到高，costDes-->总价从高到低，costAsc-->总价从低到高|

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
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
            "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
            "link": "https://item.jd.com/4264502.html",
            "title": "安防小卫士",
            "description": "wifi/电话双网 您的智能小卫士",
            "uid": "13212133313",
            "type": "smarthome",
            "price": "589.0",
            "currency": "¥"
        },
        {
            "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
            "link": "https://item.jd.com/4264502.html",
            "title": "安防小卫士",
            "description": "wifi/电话双网 您的智能小卫士",
            "uid": "13212133313",
            "type": "smarthome",
            "price": "589.0",
            "currency": "¥"
        },
        {
            "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
            "link": "https://item.jd.com/4264502.html",
            "title": "安防小卫士",
            "description": "wifi/电话双网 您的智能小卫士",
            "uid": "13212133313",
            "type": "smarthome",
            "price": "589.0",
            "currency": "¥"
        }
    ]
}
```
|key | 类型 | 描述 |
| - | -| -|
|imageUrl | String | 该商品缩略图url |
|link | String | 跳转到该商品详情页的web url|
|description | String | 对商品的简单描述 |
|uid | String | 该商品唯一识别id |
|type | String | 表示当前商品的类型：智能家居smarthome、便利店/超市/商场shop |
|price | String | 价格 |
|currency | String | 标识币种，可以是符号，也可以是文字，看前后端的需求，也可以再立一个字段表示 |

#### 商品详情
> 例如：https://xxx.com/ec/v1/products/65464131
###### 请求头

```
GET /ec/v1/products
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
无

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
```

###### 响应
```
{
    "message": "居然被你查询成功了",
    "code": 200,
    "data": {
        "images": [
            {
                "image": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                "discription": "500g/包"
            },
            {
                "image": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                "discription": "500g/包"
            },
            {
                "image": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                "discription": "500g/包"
            }
        ],
        "title": "巧克力豆",
        "uid": "45645646545454",
        "description": "500g/包",
        "share": "https://item.jd.com/4264502.html",
        "pay": {
            "cost": "4.125",
            "discount": "7.5",
            "price": "5.5",
            "currency": "¥"
        },
        "shop": {
            "name": "克拉家园店",
            "type": "shop",
            "cartUid": "235353552352",
            "uid": "54545454545"
        },
        "detail": {
            "url": "https://item.jd.com/4264502.html",
            "images": [
                "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg"
            ]
        },
        "attributes": [
            {
                "attribute": "领券",
                "uid": "356465464",
                "name": "满140元减10元券",
                "type": {
                    "uid": "6564656565656",
                    "name": "店铺优惠券"
                }
            },
            {
                "attribute": "服务",
                "uid": "356465464",
                "name": "7天无理由",
                "type": {
                    "uid": "6564656565656",
                    "name": "服务类型"
                }
            },
            {
                "attribute": "参数",
                "uid": "356465464",
                "name": "净含量12g，包装方式...",
                "type": {
                    "uid": "6564656565656",
                    "name": "参数类型是这个"
                }
            },
            {
                "attribute": "促销",
                "uid": "356465464",
                "name": "满1元可享受10倍积分",
                "type": {
                    "uid": "6564656565656",
                    "name": "积分"
                }
            }
        ]
    }
}
```
|key | 类型 | 描述 |
| - | -| -|
|images | String数组 | 该商品顶部缩略图url，有可能是轮播图，所以是个数组 |
|share | String | 跳转到该商品详情页的web url|
|title | String | 该商品的名称|
|description | String | 对商品的简单描述 |
|uid | String | 最外层为该商品主键，shop内的uid为店铺uid，categories中uid为分类主键 |
|pay | 对象 | 描述在该商品的价格展示，cost是最终价格，price是打折前的价格，discount是折扣，currency是币种 |
|cost | String | 最终价格 |
|price | String | 打折前的价格 |
|discount | String | 折扣 |
|currency | String | 标识币种，可以是符号，也可以是文字，看前后端的需求，也可以再立一个字段表示 |
|shop | 对象 | 该商品所属的商店信息，不过多解释 |
|detail | 对象 | 其中的images是详情图片列表 |
|attributes | 数组 | 包含该商品的一系列附加属性，比如服务，优惠等 |

## 店铺部分
店铺接口只需要“增删改查”中的“查”，所以只有GET
> 例如：https://xxx.com/ec/v1/shop/{店铺uid}

###### 请求头

```
GET /ec/v1/shop
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
* 不传任何参数则返回全部店铺。
* 对于接口https://xxx.com/ec/v1/shop/734237982375则不需要传参数，返回店铺详情。

```
?params={longitude:25.53,latitude:65.36}
```

|params | 类型 | 描述 |
| - | -| -|
|longitude | String | 以当前经纬度为中心搜索周边店铺 |
|latitude | String | 以当前经纬度为中心搜索周边店铺 |
|sort | String | 排序参数，不传或者default-->综合排序，des-->降序，asc-->升序，priceDes-->价格从高到低，priceAsc-->价格从低到高，costDes-->总价从高到低，costAsc-->总价从低到高|

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
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
            "uid": "56565665656",
            "cartUid": "235353552352",
            "name": "云山凯宾斯基店",
            "address": "惠州市云山西路2号凯宾斯基C座301-302",
            "time": "09:00-24:00",
            "url": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
            "longitude": "85.66",
            "latitude": "36.33"
        },
        {
            "uid": "56565665656",
            "cartUid": "235353552352",
            "name": "云山凯宾斯基店",
            "address": "惠州市云山西路2号凯宾斯基C座301-302",
            "time": "09:00-24:00",
            "url": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
            "longitude": "85.66",
            "latitude": "36.33"
        },
        {
            "uid": "56565665656",
            "cartUid": "235353552352",
            "name": "云山凯宾斯基店",
            "address": "惠州市云山西路2号凯宾斯基C座301-302",
            "time": "09:00-24:00",
            "url": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
            "longitude": "85.66",
            "latitude": "36.33"
        }
    ]
}
```
|key | 类型 | 描述 |
| - | -| -|
|uid | String | 该店铺的主键 |
|name | String | 店铺的名称 |
|address | String | 该店铺地址 |
|time | String | 营业时间 |
|url | String | icon的url |
|longitude | String | 该店铺的经度 |
|latitude | String | 该店铺的纬度 |

### 购物车部分
购物车接口原本是不需要的，可以通过缓存来实现，但是考虑到一个端上买了东西放到购物车，其他端也可以看得到，并继续完成购物流程，所以还是得有这部分接口，有“增删改查”四部分，**注意：这里说的“增删改查”是指针对购物车里的商品的增删改查，不是针对购物车的增删改查**。针对一般电商而已，从App的角度来看，每个用户只有一个购物车，所以不需要知道购物车的uid，根据token就可以查询该用户名下购物车存放的商品信息，但是我们的业务特殊一点，包含便利店外卖成分，所以是一个用户对应多个购物车，**并且是一个店铺一个购物车，一一对应**，至于智能家居则是一个特殊店铺而已，默认每个用户对每个店铺就配有一个购物车，所以购物车是不需要增删改查操作的，**但是这个购物车的uid不知道怎么获得，由于一个店铺就对应一个购物车，因此暂定购物车的uid就是店铺的uid，后期再议**。

#### 新增一个或者多个商品到购物车
> 地址：https://xxx.com/ec/v1/carts/{cartUID}/products

###### 请求头

```
POST /ec/v1/carts/656464646/products/32434
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
```
{
    "message": "上传这几个商品到购物车",
    "data": [
        {
            "uid": "13212133313",
            "amount": "5"
        },
        {
            "uid": "13212133313",
            "amount": "5"
        }
    ]
}
```


|params | 类型 | 描述 |
| - | -| -|
|uid | String | 商品主键 |
|amount | String | 商品个数 |

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
```

###### 响应
```
{
    "message": "居然被你保存成功了",
    "code": 200,
    "data": [
        {
            "uid": "13212133313"
        },
        {
            "uid": "13212133313"
        }
    ]
}
```
|key | 类型 | 描述 |
| - | -| -|
|uid | String | 保存到购物车的商品主键 |

#### 删除一个购物车中的商品
>删除一个购物车中的商品：https://xxx.com/ec/v1/carts/{cartUID}/products/{productUID}
>删除多个购物车中的商品：https://xxx.com/ec/v1/carts/{cartUID}/products

###### 请求头

```
DELETE /ec/v1/carts/6564646464/products/13434123
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
清空该购物车商品则不需要参数，删除购物车中的多个商品则需要以下参数。如果是只删除一个，则不需要参数，因为uid已经在url上了。
```
{
    "message": "删除这几个商品",
    "data": [
        {
            "uid": "13212133313"
        },
        {
            "uid": "13212133313"
        }
    ]
}
```

|params | 类型 | 描述 |
| - | -| -|
|uid | String | 表示某条商品信息的主键 |

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
```

###### 响应
```
{
    "message": "居然被你删除成功了",
    "code": 200,
    "data": [
        {
            "uid": "13212133313"
        },
        {
            "uid": "13212133313"
        }
    ]
}
```
|params | 类型 | 描述 |
| - | -| -|
|uid | String | 表示已删除的商品主键 |

#### 修改购物车一个或者多个商品
> 修改购物车一个商品：https://xxx.com/ec/v1/carts/{cartUID}/products/{productUID}
> 修改购物车多个商品：https://xxx.com/ec/v1/carts/{cartUID}/products
###### 请求头

```
PUT /ec/v1/carts/656464646/products/25234524
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
如果是修改一个，则不需要穿数组，因为uid已经在url上，此时已经限定了只操作这个uid的商品。
```
{
    "message": "修改这几个商品",
    "data": [
        {
            "title": "巧克力豆",
            "uid": "45645646545454",
            "description": "500g/包",
            "share": "https://item.jd.com/4264502.html",
            "pay": {
                "cost": "4.125",
                "discount": "7.5",
                "price": "5.5",
                "currency": "¥"
            },
            "shop": {
                "name": "克拉家园店",
                "type": "shop",
                "uid": "54545454545"
            }
        }
    ]
}
```
|key | 类型 | 描述 |
| - | -| -|
|share | String | 跳转到该商品详情页的web url|
|title | String | 该商品的名称|
|description | String | 对商品的简单描述 |
|uid | String | 最外层为该商品主键，shop内的uid为店铺uid |
|pay | 对象 | 描述在该商品的价格展示，cost是最终价格，price是打折前的价格，discount是折扣，currency是币种 |
|cost | String | 最终价格 |
|price | String | 打折前的价格 |
|discount | String | 折扣 |
|currency | String | 标识币种，可以是符号，也可以是文字，看前后端的需求，也可以再立一个字段表示 |
|shop | 对象 | 该商品所属的商店信息，不过多解释 |

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
```

###### 响应
```
{
    "message": "居然被你修改成功了",
    "code": 200,
    "data": [
        {
            "uid": "1333644113313131"
        }
    ]
}
```
|key | 类型 | 描述 |
| - | -| -|
|uid | String | 表示已修改商品的主键 |

#### 查询购物车商品信息
> 地址：https://xxx.com/ec/v1/carts/{cartUID}/products

###### 请求头

```
GET /ec/v1/carts/356466/products
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
>?params={shopUid=2562352352,sort:des}

如果是查询某一条递送时，由于uid已经在url上，所以返回一条指定递送数据。

|params | 类型 | 描述 |
| - | -| -|
|shopUid | String |通过shopUid来筛选出每一个店铺对应的购物车中的产品列表|
|sort | String | 排序参数，不传或者default-->综合排序，des-->降序，asc-->升序，priceDes-->价格从高到低，priceAsc-->价格从低到高，costDes-->总价从高到低，costAsc-->总价从低到高|

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
```

###### 响应
```
{
    "code": 200,
    "data": [
        {
            "products": [
                {
                    "count": 2,
                    "description": "500g/包",
                    "icon": "https://www.audi.cn/content/dam/nemo/cn/model/a7/rs7_sportback/2015/1680x1050/3q/rs7_sportback_performance_gallery_10.jpg.resize.maxWidth=1180.jpg",
                    "pay": {
                        "cost": "4.125",
                        "currency": "¥",
                        "discount": "7.5",
                        "price": "5.5"
                    },
                    "share": "https://item.jd.com/4264502.html",
                    "title": "巧克力豆",
                    "uid": "45645646545454"
                },
                {
                    "count": 5,
                    "description": "500g/包",
                    "icon": "https://www.audi.cn/content/dam/nemo/cn/model/a7/rs7_sportback/2015/1680x1050/3q/rs7_sportback_performance_gallery_10.jpg.resize.maxWidth=1180.jpg",
                    "pay": {
                        "cost": "4.125",
                        "currency": "¥",
                        "discount": "7.5",
                        "price": "5.5"
                    },
                    "share": "https://item.jd.com/4264502.html",
                    "title": "大力糖",
                    "uid": "45645646545454"
                }
            ],
            "shop": {
                "name": "克拉家园店",
                "serviceType": "快递服务",
                "type": "shop",
                "uid": "54545454545"
            }
        },
        {
            "products": [
                {
                    "count": 2,
                    "description": "500g/包",
                    "icon": "https://www.audi.cn/content/dam/nemo/cn/model/a7/rs7_sportback/2015/1680x1050/3q/rs7_sportback_performance_gallery_10.jpg.resize.maxWidth=1180.jpg",
                    "pay": {
                        "cost": "4.125",
                        "currency": "¥",
                        "discount": "7.5",
                        "price": "5.5"
                    },
                    "share": "https://item.jd.com/4264502.html",
                    "title": "巧克力豆",
                    "uid": "45645646545454"
                },
                {
                    "count": 5,
                    "description": "500g/包",
                    "icon": "https://www.audi.cn/content/dam/nemo/cn/model/a7/rs7_sportback/2015/1680x1050/3q/rs7_sportback_performance_gallery_10.jpg.resize.maxWidth=1180.jpg",
                    "pay": {
                        "cost": "4.125",
                        "currency": "¥",
                        "discount": "7.5",
                        "price": "5.5"
                    },
                    "share": "https://item.jd.com/4264502.html",
                    "title": "大力糖",
                    "uid": "45645646545454"
                }
            ],
            "shop": {
                "name": "威斯丹利店",
                "serviceType": "送货上门",
                "type": "shop",
                "uid": "123412312"
            }
        }
    ],
    "first": "https://...",
    "last": "https://...",
    "message": "居然被你查询成功了",
    "next": "https://...",
    "page": 0,
    "pageSize": 20,
    "previous": "https://..."
}
```
|key | 类型 | 描述 |
| - | -| -|
|share | String | 跳转到该商品详情页的web url|
|title | String | 该商品的名称|
|description | String | 对商品的简单描述 |
|uid | String | 最外层为该商品主键，shop内的uid为店铺uid |
|pay | 对象 | 描述在该商品的价格展示，cost是最终价格，price是打折前的价格，discount是折扣，currency是币种 |
|cost | String | 最终价格 |
|price | String | 打折前的价格 |
|discount | String | 折扣 |
|currency | String | 标识币种，可以是符号，也可以是文字，看前后端的需求，也可以再立一个字段表示 |
|shop | 对象 | 该商品所属的商店信息，不过多解释 |

### skus部分（最小库存单元集合）
sku为最小库存单元，这部分概念需要大家自行补习一下。反正要想唯一确定一个库存里的商品，就需要一系列属性来确定，这里的属性包括分类属性+产品属性+其他属性。在详情页中点击放入购物车或购买时，请求该接口来获取商品的详细规格来选择商品。

#### 查询一条或者多条sku
> 地址：https://xxx.com/ec/v1/skus/{sku的uid}

###### 请求头

```
GET /ec/v1/skus/566646464
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
无

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
```

###### 响应

```
{
    "code": "200",
    "message": "居然被你查询成功了",
    "data": {
        "attributes": [
            {
                "attribute": "性别",
                "values": [
                    {
                        "stockQuantity": 30,
                        "value": "男"
                    },
                    {
                        "stockQuantity": 10,
                        "value": "女"
                    }
                ]
            },
            {
                "attribute": "尺码",
                "values": [
                    {
                        "stockQuantity": 30,
                        "value": "红色"
                    },
                    {
                        "stockQuantity": 10,
                        "value": "黄色"
                    },
                    {
                        "stockQuantity": 0,
                        "value": "蓝色"
                    }
                ]
            },
            {
                "attribute": "尺码",
                "values": [
                    {
                        "stockQuantity": 30,
                        "value": "X码"
                    },
                    {
                        "stockQuantity": 10,
                        "value": "L码"
                    },
                    {
                        "stockQuantity": 0,
                        "value": "M码"
                    }
                ]
            }
        ],
        "skus": [
            {
                "sku": "男+红色+X码",
                "stockQuantity": 30,
                "price": 50,
                "currency": "¥",
                "note": "备注：该商品XXX",
                "uid": "34646464464",
                "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg"
            },
            {
                "sku": "男+黄色+X码",
                "stockQuantity": 30,
                "price": 50,
                "currency": "¥",
                "note": "备注：该商品XXX",
                "uid": "34646464464",
                "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg"
            },
            {
                "sku": "男+黄色+M码",
                "stockQuantity": 30,
                "price": 50,
                "currency": "¥",
                "note": "备注：该商品XXX",
                "uid": "34646464464",
                "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg"
            }
        ]
    }
}
```
|params | 类型 | 描述 |
| - | -| -|
|attributes | 数组 | 该产品的属性集合 |
|attribute | String | 属性类型，对应着一系列值values |
|values | 数组 | 对应的属性值的集合 |
|stockQuantity | 整型 | 库存数量 |
|shopType | String | 店铺类型 |
|price | String | 价格，精确到小数点后两位，单元为元 |
|currency | String | 标识币种，可以是符号，也可以是文字，看前后端的需求，也可以再立一个字段表示 |

### 首页部分
理论上来说是不需要这个接口的，根据已有的接口是可以拼凑出现在业务需求的数据的，但这样的话需要同时访问的接口数目过多，考虑到性能问题，则合并出一个home接口来。

#### 查询
> 地址：https://xxx.com/ec/v1/home

###### 请求头

```
GET /ec/v1/home
Accept: application/json
Content-Type: application/json;charset=UTF-8
```

###### 参数
>?params={type=smarthome}

|params | 类型 | 描述 |
| - | -| - |
|type | String |表示当前商品的类型：智能家居smarthome、便利店/超市/商场shop|

###### 响应头

```
Content-Type:application/json; charset=utf-8
Status:200 OK
```

###### 响应

```
{
    "message": "居然被你查询成功了",
    "code": 200,
    "page": 0,
    "pageSize": 20,
    "first": "",
    "next": "",
    "previous": "",
    "last": "",
    "data": {
        "ads": [
            {
                "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                "title": "安防小卫士",
                "description": "wifi/电话双网 您的智能小卫士",
                "uid": "13212133313",
                "type": "web",
                "price": "589.0",
                "currency": "¥",
                "link": "https://www.baidu.com/"
            },
            {
                "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                "title": "安防小卫士",
                "description": "wifi/电话双网 您的智能小卫士",
                "uid": "13212133313",
                "type": "web",
                "price": "589.0",
                "currency": "¥",
                "link": "https://www.baidu.com/"
            },
            {
                "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                "title": "安防小卫士",
                "description": "wifi/电话双网 您的智能小卫士",
                "uid": "13212133313",
                "type": "web",
                "price": "589.0",
                "currency": "¥",
                "link": "https://www.baidu.com/"
            }
        ],
        "recommendations": [
            {
                "category": {
                    "category": "智能主机",
                    "uid": "516165654656"
                },
                "products": [
                    {
                        "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                        "title": "安防小卫士",
                        "description": "wifi/电话双网 您的智能小卫士",
                        "uid": "13212133313",
                        "type": "product",
                        "price": "589.0",
                        "currency": "¥",
                        "link": "https://www.baidu.com/"
                    },
                    {
                        "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                        "title": "安防小卫士",
                        "description": "wifi/电话双网 您的智能小卫士",
                        "uid": "13212133313",
                        "type": "product",
                        "price": "589.0",
                        "currency": "¥",
                        "link": "https://www.baidu.com/"
                    },
                    {
                        "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                        "title": "安防小卫士",
                        "description": "wifi/电话双网 您的智能小卫士",
                        "uid": "13212133313",
                        "type": "product",
                        "price": "589.0",
                        "currency": "¥",
                        "link": "https://www.baidu.com/"
                    }
                ]
            },
            {
                "category": {
                    "category": "智能门锁",
                    "uid": "516165654656"
                },
                "products": [
                    {
                        "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                        "title": "安防小卫士",
                        "description": "wifi/电话双网 您的智能小卫士",
                        "uid": "13212133313",
                        "type": "product",
                        "price": "589.0",
                        "currency": "¥",
                        "link": "https://www.baidu.com/"
                    },
                    {
                        "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                        "title": "安防小卫士",
                        "description": "wifi/电话双网 您的智能小卫士",
                        "uid": "13212133313",
                        "type": "product",
                        "price": "589.0",
                        "currency": "¥",
                        "link": "https://www.baidu.com/"
                    },
                    {
                        "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                        "title": "安防小卫士",
                        "description": "wifi/电话双网 您的智能小卫士",
                        "uid": "13212133313",
                        "type": "product",
                        "price": "589.0",
                        "currency": "¥",
                        "link": "https://www.baidu.com/"
                    }
                ]
            },
            {
                "category": {
                    "category": "智能配件",
                    "uid": "516165654656"
                },
                "products": [
                    {
                        "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                        "title": "安防小卫士",
                        "description": "wifi/电话双网 您的智能小卫士",
                        "uid": "13212133313",
                        "type": "product",
                        "price": "589.0",
                        "currency": "¥",
                        "link": "https://www.baidu.com/"
                    },
                    {
                        "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                        "title": "安防小卫士",
                        "description": "wifi/电话双网 您的智能小卫士",
                        "uid": "13212133313",
                        "type": "product",
                        "price": "589.0",
                        "currency": "¥",
                        "link": "https://www.baidu.com/"
                    },
                    {
                        "imageUrl": "http://ww3.sinaimg.cn/large/0060lm7Tly1fo6vt0p500j30af0ad758.jpg",
                        "title": "安防小卫士",
                        "description": "wifi/电话双网 您的智能小卫士",
                        "uid": "13212133313",
                        "type": "product",
                        "price": "589.0",
                        "currency": "¥",
                        "link": "https://www.baidu.com/"
                    }
                ]
            }
        ]
    }
}
```
|key | 类型 | 描述 |
| - | -| -|
|imageUrl | String | 该商品缩略图url |
|link | String | 跳转到web的url|
|description | String | 对商品的简单描述 |
|uid | String | 该商品唯一识别id |
|type | String | 表示当前商品的类型：智能家居smarthome、便利店/超市/商场shop |
|price | String | 价格 |
|currency | String | 标识币种，可以是符号，也可以是文字，看前后端的需求，也可以再立一个字段表示 |
|type | String | product表示跳转到原生详情页，web表示跳转到web页面 |

## MQTT部分

### 使用场景
所有需要 **双向通讯（确切的说是需要后台主动推送给前端的情景）** 的部分都尽量使用MQTT协议

* 消息提示：比如底部导航栏的tab勋章提示、未读消息的提示。
* 广告弹框：比如后台发送一个广告信息，界面会弹出一个广告弹框。
* 消息中心：私信、IM等

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
个人倾向于将版本号放在HTTP/MQTT头信息中，虽然不如放入URL中更直观，但是不方便我们统一管理，因为在前端URL是拼出来的String，请求头是统一个对象去设置，除非有特殊情况，某一个接口需1.0版本，某一个接口需2.0版本，这就另当别论，到时候统一商量，在拼这个URL的时候，放到固定目录（位置），如：smarthome.aghl.com:8080<u>**/版本（一般用v1、v2）/**</u>user  统一放在一级目录，这样的前端在拼接的时候，统一放到某个位置，也就方便管理了。

## URL失效
随着系统发展，总有一些API失效或者迁移，对失效的API，返回404 not found 或 410 gone；对迁移的API，返回 301 重定向。

## 对于后台文档的要求
![文档要求](https://i.imgur.com/aQ3qb4f.png)
文档要求描述详尽，尽可能的引导接口使用者理解接口设计，这样才能减少接口的改动，又能适应多变的业务。

## 安全
待定

## 统计、历史记录等埋点业务
待定

### 参考
* http://www.ruanyifeng.com/blog/2011/09/restful.html
* http://www.ruanyifeng.com/blog/2014/05/restful_api.html
* https://demo.openhab.org:8443/doc/index.html
* https://api.github.com/
* https://developer.github.com/v3/search/#search-users
* http://open.taobao.com/docs/api_list.htm?spm=a219a.7629140.0.0.Z2srrA
* https://api.github.com/
* https://developer.github.com/v3/search/#search-users
* http://open.taobao.com/docs/api_list.htm?spm=a219a.7629140.0.0.Z2srrA
* http://www.infoq.com/cn/articles/webber-rest-workflow/
* https://www.zhihu.com/question/27785028
* http://www.infoq.com/cn/articles/webber-rest-workflow/
* https://www.jianshu.com/p/0ede793d41cc
* http://wiki.jikexueyuan.com/project/github-developer-guides/getting-started.html
* https://www.zhihu.com/question/28557115
* https://www.zhihu.com/question/35210451
* https://leancloud.cn/dashboard/apionline/index.html
* https://github.com/Microsoft/api-guidelines
首页（这个其实可以用，因为真的如果是需要首页展示的话，也应该是拼凑出来的，当我们完成了分类这个模块的内容的话，其实就只要根据分类模块获取分类列表，然后根据这个列表返回的路径，然后挨个的获取每一个分类的里的商品，就可以拼凑出一个首页的数据了。）
product还是得包含店铺信息，因为订单可以点击到店铺，但是这个也可以不需要包含，因为生成的订单那个数据是来自详情的，详情可以有店铺信息
SPU概念和SKU概念，其中SPU概念在我们平台用不到。

























