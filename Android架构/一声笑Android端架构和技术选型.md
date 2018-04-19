![logo](https://i.imgur.com/DyZZ5vE.png)

## 摘要

* [前言](#前言)
* [开发环境](#开发环境)
	* [IDE](#IDE)
* [代码样式规范](#4-代码样式规范)
* [资源文件规范](#5-资源文件规范)
* [版本统一规范](#6-版本统一规范)
* [第三方库规范](#7-第三方库规范)
* [注释规范](#8-注释规范)
* [测试规范](#9-测试规范)
* [ 其他的一些规范](#10-其他的一些规范)


# 一声笑Android端架构和技术选型
这份文档可看做我司Android部分项目简介和技术简介。由于时间紧凑，若有遗漏可通过QQ/微信666233联系我询问。

### 前言
由于我司处于项目探索阶段，会有较多的项目和迭代，因此快速开发是主旋律，可维护是基调，这就要求代码高可复用、灵活拆分、便于调试，基于这些原因我将根据经验对Android开发架构和技术选型进行梳理。

## 开发环境

### IDE
Android Studio在3.0这一版本改动比较大，功能上增强了不少，起码可以原生使用lambda，也可以使用Stream流操作集合，这些在数据处理上都是很好的。

### 插件
工欲善其事必先利其器


## 规范
无规矩不成方圆，为了有利于项目维护、增强代码可读性、提升 Code Review 效率以及规范团队协作开发，应当遵守一定的规范，具体参考

## Android部分技术栈

### 规范

配合阿里巴巴开源的代码规范检测插件


### Gradle
对Gradle的配置要求：
* 对于编译版本和依赖版本等进行统一管理。
* 尽量分为debug版本和release版本。
* 能自动处理版本号。
* 配置多渠道打包。
* 自定义输出各类Apk文件名称。
* 尽量通过配置Gradle的形式配置第三方Key。

### 组件化
由于技术积累时间较短，自己的组件较少，目前只开源并维护了几个，在项目的依赖中有注释可查看到。

### 模块化
整体结构关系应该是如下图所示：

![架构图](https://i.imgur.com/IwjarzT.png)

整体架构从上依赖至下，下层为上层提供服务。架构的设计主要是为了应对我们公司自身业务探索时快速开发，在不断的开发过程中汲取可用部分抽象整理成独立的模块以方便后续使用。同时模块的划分也根据不同的功能和业务划分到不同的层级中。理论上来说可以有一个集成代码库，然后从这一套代码中通过差异化的构建，编译出不同的App，缺点是代码库会越来越大，组件越来越多，模块也越来越多。如果单独根据项目来集成又会出现多套相同代码需要维护的问题，这个得根据业务需求和是否需要长期维护等因素适当取舍才行。根据Android项目的目录特点，每一个项目在创建出来会有一个App module，这个module我们当做一个空壳，单独依赖main层，只根据需求写Gradle配置而已。模块之间的通讯通过ARouter进行，或者可以自己在ACommon层定义接口的方式进行通讯。

### MVP模式
MVC/MVP/MVVM模式有很多，这里所说的也不是纯粹的MVP，应该是MVP和MVVM混合体的改造版，目的还是为了充分解耦和方便调试。同时为了方便调试，各层之间完全依赖接口。

![MVP模式](https://i.imgur.com/UgFvZJg.jpg)

## 前后端分离

### REST ful接口

### UI框架
整体框架以多Activity+多Fragment的形式搭建，Activity只做Fragment的容器，Fragment做为View层。使用了Fragmentation这个框架来操作Fragment。

### 依赖
我们公司依赖的第三方库都写在ABse module和main module中的build.gradle中，可自行到github上查看学习。需要注意的是：
- 其中由于摄像头SDK中依赖了RxJava1，与我们原本依赖的RxJava2冲突了，所以也回退了版本。
- 某些我们自己写的库则也可以根据源代码来学习，github上也有响应的介绍。
- 最头疼的是依赖的部分硬件商的SDK，这个实在头疼。

### 代码
都是用git管理，使用的是oschina的码云。

### 构建
这一部分是根据业务需求来的，因为后期突然有一个渠道商的概念，导致一个项目需要有不同的渠道商App，他们之间只是App名称、图标、主题色和icon等的不同，所以后期调整，通过productFlavors构建不同的渠道商的App。因此会有很多Gradle代码要写。由于不同的App包名不同，因此使用的第三方SDK需要填写不同的appid或者key之类的，其中需要注意的是：
- LBS：我司用的是高德地图服务，不同的App，apikey不一样，```manifestPlaceholders = [amap_apikey: "717e984e84d635547f8bcca89ad03109"]```
- 支付：本来是用App原生支付的，后期由于渠道商多了，太麻烦，微信需要申请不同的商户，且能申请的商户个数的有限的，所以就使用改造版的H5支付，同时也是免SDK支付，这个是我们公司独创的，具体要看APayment的代码，技术点在后台会返回一个weixin：//的协议uri，自己解析得到也行，自己用webview跳转得到也行，只要拿到这个uri就能合法的调起微信支付界面。支付宝没这个问题，所以还是原生App支付。
- 推送：也是要自己配置Appid和key
```
yishequ {
            buildConfigField "String", "XIAOMI_APP_ID", "\"2882303761517621288\""//配置小米的AppId。
            buildConfigField "String", "XIAOMI_APP_KEY", "\"5861762181288\""//配置小米的AppKey。

            manifestPlaceholders = [application_id  : "com.aglhz.yicommunity",
                                    aliyun_appkey   : "24632152",
                                    aliyun_appsecret: "077b8ea013652e988304c0bc6b41fb3a",
                                    huawei_appid    : "100099829"]
        }
        meilun {
            buildConfigField "String", "XIAOMI_APP_ID", "\"2882303761517677829\""//配置小米的AppId。
            buildConfigField "String", "XIAOMI_APP_KEY", "\"5251767792829\""//配置小米的AppKey。

            manifestPlaceholders = [application_id  : "com.meilun.one",
                                    aliyun_appkey   : "24713561",
                                    aliyun_appsecret: "8d7429b0e82469d5b2ab909719f06d31",
                                    huawei_appid    : "100156589"]
        }
        vensi {
            buildConfigField "String", "XIAOMI_APP_ID", "\"2882303761517621288\""//配置小米的AppId。
            buildConfigField "String", "XIAOMI_APP_KEY", "\"5861762181288\""//配置小米的AppKey。

            manifestPlaceholders = [application_id  : "com.aglhz.yicommunity",
                                    aliyun_appkey   : "24632152",
                                    aliyun_appsecret: "077b8ea013652e988304c0bc6b41fb3a",
                                    huawei_appid    : "100099829"]
        }
```

### 不足之处
由于没有考虑到后期业务的变化，很多不足的地方和一些临时方案充斥在代码中，比如UserHelper这个类，用的是sp文件存储多个用户的数据，应该改用数据库来存储，使用适当的ORM框架来操作。比如推送，后期要求使用多维度的推送，集成多家厂商的推送，所以也有一些临时方案代码。需要后期慢慢优雅的实现一遍。

### 建议
还有很多的细节希望在后续版本中出现，让产品越来越优秀，具体链接如下：
> http://note.youdao.com/groupshare/?token=7197035EA599410BAD2499ED222FF7D9&gid=47332264
> http://note.youdao.com/noteshare?id=b198928e8715524a23ef9ed1c7a6543d&sub=303857AD7F6D41B684280EC19195C840

这些都是我平时点滴想到的，已经实现了的是划了横杠，没划横杠的就是需要考虑实现的。**我的观点不一定正确**，酌情处理才行。
- 尽量使用最新的技术。
- 现在Android 4.4以下的手机已经越来越少，几乎可以不用考虑，因此可以把minSdkVersion设置成20。
- 在使用icon的时候，尽量使用矢量图。
- 数据操作可以考虑使用RxJava或者Stream。
- 由于我们公司有渠道商这个概念，因此一个项目会变成很多个变种App，它们之间样式上的差别可以考虑通过插件化的换肤功能来实现。
- 考虑使用跨平台的方式实现，混合开发会好一点。
- 引入热更新和插件技术，否则有些需求实现起来很为难，但是这个要求也高，需要酌情处理。
- 添加适当的动画，比如转场动画和交互动画。































