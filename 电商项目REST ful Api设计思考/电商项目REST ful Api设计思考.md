# 电商项目(EC)协议定制

>主要依赖HTTP+MQTT完成整个电商项目的前后端通讯和协作。当然只是抛砖引玉，不一定完全正确，需要大家一起商议后决定，由于不知道后台具体的表的设计如何，因此只能猜测着用url定义资源，因此url设计的路径部分会有不合理的地方，所以最好商讨、抽象出一套合理的资源路径，要注意这一点,可参考** [github接口](https://developer.github.com/v3/ "github接口") **和** [淘宝Api](http://open.taobao.com/doc2/apiList.htm "淘宝Api") **的设计。

---

https://api.github.com/
https://developer.github.com/v3/search/#search-users
http://open.taobao.com/docs/api_list.htm?spm=a219a.7629140.0.0.Z2srrA
http://www.infoq.com/cn/articles/webber-rest-workflow/
https://www.zhihu.com/question/27785028
http://www.infoq.com/cn/articles/webber-rest-workflow/
https://www.jianshu.com/p/0ede793d41cc
http://wiki.jikexueyuan.com/project/github-developer-guides/getting-started.html
https://www.zhihu.com/question/28557115
https://www.zhihu.com/question/35210451

首页（这个其实可以用，因为真的如果是需要首页展示的话，也应该是拼凑出来的，当我们完成了分类这个模块的内容的话，其实就只要根据分类模块获取分类列表，然后根据这个列表返回的路径，然后挨个的获取每一个分类的里的商品，就可以拼凑出一个首页的数据了。）


对于get的列表，比如shop、product，如没有id路径，则返回的是整个表中的数据，只是需要分页而已，每一次返回20--100条。

---


## 要求
### 目标
* 接口“粒度”争取设计得足够小，争取在业务发生变化后，后台接口不需要增减，只需要前端组合接口仍然能满足新的业务需求。
* 任何一个接口都可以获取到数据，哪怕没传参数。

### 命名规范
一名二姓三风水，四积阴德五读书，名不正则言不顺，言不顺则事难成。软件开发其实就是门命名的艺术，所以首先定义一些规范，提出一些硬性要求，大家在命名的时候尽量多花点心思，多参考优秀的命名风格。
* 强烈推荐参考：[参考阿里巴巴Java开发手册](http://www.baidu.com "阿里巴巴Java开发手册")，[iOS开发手册](https://www.baidu.com/s?ie=utf-8&f=3&rsv_bp=1&ch=3&tn=98010089_dg&wd=iOS%E5%BC%80%E5%8F%91%E7%BC%96%E7%A0%81%E5%8F%8A%E5%91%BD%E5%90%8D%E8%A7%84%E8%8C%83&oq=ios%25E5%25BC%2580%25E5%258F%2591%25E5%2591%25BD%25E5%2590%258D%25E8%25A7%2584%25E8%258C%2583&rsv_pq=861604950004ebc0&rsv_t=58c07RTbI99q5XYs9eJyxgRtHHUk%2FLDHkwLo0Z86N1kK3GeDd1ktZqvYuIIsSazuXxg&rqlang=cn&rsv_enter=1&inputT=3905&rsv_sug3=17&rsv_sug1=16&rsv_sug7=100&rsv_n=2&bs=ios%E5%BC%80%E5%8F%91%E5%91%BD%E5%90%8D%E8%A7%84%E8%8C%83 "iOS开发编码及命名规范")。
* 一个单词尽量选择5--7个字母的，这样才最优美。
* 首字母缩写的单词尽量每个字母都用大写，例如ID。用个小写，人家还以为是一个单词。当然uri、url、urn这种除外，因为大家都知道这个是什么。
* 规范我们公司的基础包名与项目的关系，看需不需要前后端统一一下，现在的亿社区是：com.aglhz.yicommunity，智能家居还是用美伦安保为蓝本：com.meilun.security.smart。还有很多细节需要完善啊，现在不改等到后面引用的地方越来越多，比如地图，支付、社会化分享等，这些虽然改不改无所谓，但是会让后来人引起不必要的误会，尤其是像我这种有强迫症的人，完全难以忍受。
* 前后端的某些名称概念要统一用某一个单词，比如支付的统一订单，支付宝用的是order，微信用的是unifiedorder，那我们统一对订单这个概念用order这个词。再比如主机：后台用gateway，现在我们统一用host。这单词不统一很容易分裂。

### URI规范

URI 表示资源，资源一般对应服务器端领域模型中的实体类。

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
* 不要使用缩写
* 统一用驼峰命名法
* 不要使用_或者-
* 用名词复数表示集合类型
* 为了方便以后的扩展兼容，如果返回的是数组，强烈建议用一个包含如items属性的对象进行包裹。如：{"items":[{},{}]}
* 建议对每个字段设置默认值（数组型可设置为[],字符串型可设置为””，数值可设置为0，对象可设置为{}）,这一条是为了方便前端/客户端进行判断字段存不存在操作。
* 建议资源使用UUID最为唯一标识。同时建议命名为id或者uid。
* 采用UTF-8编码。
* 数据应该拿来就能用，不应该还要进行转换操作。

## 业务简介
本项目是我司的一次电商重新探索，之前的宅宜购项目的试探未成功，此次在其基础上重新探索，希望做成嵌入式的商城，嵌入到其他项目或者能独立运行，承载着我司的电商渠道，为销售我司智能硬件和外包打下基础。

### 业务模块
总体参考了主流电商的功能后划分出7大部分，如下图所示：
![模块划分](https://i.imgur.com/ids8hGR.png)

## Http部分
### 使用场景
* App的初始化数据尽量都用http协议获取。
* 页面的初始化数据尽量都用http协议获取。

### 公共参数
公共参数是指每一个接口应该传的参数，同时后天要指定公共参数的默认值，且要保证没有传公共参数不会报错。

|params | 类型 | 描述 |
| - | -| -|
|token | String | 检测权限、标识登录状态 |

### 个性参数（该方案待定）
个性参数就是除了公共参数之外的，看能否考虑统一用json浓缩成一个参数，把想要表达的参数通过json中的key-value形式传递。
> 例如：https://xxx.com/ec/v1/search/product?params={keyword:方便面,order:des}
这种方式我暂时还不确定是否合理，或者考虑与业务相关的参数就用json形式包装，而与业务无关的个性参数就还是用传统的方式另立一个参数。
> 例如：https://xxx.com/ec/v1/search/product?limit=10&offset=10&params={keyword:方便面,order:des}

### 公共请求头
通过Content-Type指定请求与返回的数据格式有json和xml,暂时我们只管json的。其中请求数据还要指定Accept。
>Accept: application/json
>Content-Type: application/json;charset=UTF-8

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
常用的http状态码及使用场景：

|状态码 | 使用场景|
| - | - | 
|200 | 表示正常结果 |
|400 | bad request	常用在参数校验 |
|401 | unauthorized	未经验证的用户，常见于未登录。如果经过验证后依然没权限，应该 403（即 authentication 和 authorization 的区别）。|
|403 | forbidden	无权限|
|404 | not found	资源不存在|
|500 | internal server error	非业务类异常|
|503 | service unavaliable	由容器抛出，自己的代码不要抛这个异常|

### 电商项目url预览
![url预览](https://i.imgur.com/F9xSGFK.png)

站在数据的角度，若想满足7大模块的功能需求，可以将接口分成如下几类：

* base url：https://xxx.com/ec/v1/
* 注册登录（暂不考虑）
* 搜索search
地址：https://xxx.com/ec/v1/search/{搜索分类：如product}
参数：key、order等
* 首页home
地址：https://xxx.com/ec/v1/home
参数：key、category等
* 地址address
地址：https://xxx.com/ec/v1/address/{地址id}
参数：key、order、name、address、phone等
* 分类（或者叫“更多”）category
地址：https://xxx.com/ec/v1/category/{种类名称或者id，包括具体的种类和子种类}
参数：key、order等
* 订单order
地址：https://xxx.com/ec/v1/order/{orderID}
参数：key、sort、order、productID等
* 商品推荐recommend
地址：https://xxx.com/ec/v1/recommend
参数：key、order等
* 商品product
地址：https://xxx.com/ec/v1/product/{具体商品id}
参数：key、category、order等
* 商铺shop
地址：https://xxx.com/ec/v1/shop/{商铺id}
参数：key、order等

#### 登录注册
暂不考虑

#### 搜索
搜索只需要“增删改查”中的“查”，所以只有GET
> 例如：https://xxx.com/ec/v1/search/product?params={keyword:方便面,order:des}

1. 请求头
> GET /search/product

2. 参数
>?params={keyword:方便面,order:des}

|params | 类型 | 描述 |
| - | -| -|
|keyword | String | 查询的关键字 |
|order | String | 默认值为des，值为des表示降序，值为asc表示升序|



3. 响应头

4. 响应


## MQTT部分

### 使用场景
所有需要**双向通讯（确切的说是需要后台主动推送给前端的情景）**的部分都尽量使用MQTT协议

* 消息提示：比如底部导航栏的tab勋章提示、未读消息的提示。
* 广告弹框：比如后台发送一个广告信息，界面会弹出一个广告弹框。
* 消息中心：私信、IM等


## 错误/异常处理

* 不要发生了错误但给2xx响应，客户端可能会缓存成功的http请求；
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
    "message": "特么的又错了",
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

## 安全
待定

### 参考
* http://www.ruanyifeng.com/blog/2011/09/restful.html
* http://www.ruanyifeng.com/blog/2014/05/restful_api.html
* https://demo.openhab.org:8443/doc/index.html
* https://api.github.com/
* https://developer.github.com/v3/search/#search-users
* http://open.taobao.com/docs/api_list.htm?spm=a219a.7629140.0.0.Z2srrA































