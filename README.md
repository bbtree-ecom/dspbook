# 智慧树ADX接入规范



1.技术要求
==========

1.1通信方式及编码
-----------------

通信协议采用HTTP协议，使用POST方法发送Bid
Request消息，开启keep-alive,数据使用json格式。

不出价可以返回HTTP 状态码204 (No Content)。

字段中所有中文必须使用UTF-8编码。

1.2 HTTP请求头
--------------

Content-Type: application/json。

2．竞价信息
===========

竞价协议基于OpenRTB规范，与OpenRTB兼容，最新版本参见<https://www.iab.com/wp-content/uploads/2016/03/OpenRTB-API-Specification-Version-2-5-FINAL.pdf>，做了些扩展。与ADX相关的交互过程如下：

![](http://i1.bbtree.com/park/1533193857993.png)

ADX与DSP的服务协议包括三个部分：

* 询价请求：ADX向DSP发送竞价请求（Bid Request）

* 应答：DSP向ADX返回竞价广告信息（Bid Response）

* 获胜通知：ADX向DSP发送竞价结果（Win
Notice）。DSP也可以将自己的获胜通知监测地址添加在曝光监测中，由客户端上报，但竞价成功到展示之间有路径损耗，需要DSP自行把控

2.1 ADX向DSP发送的广告询价请求(Bid Request)
-------------------------------------------

接口信息结构如下图：

![](http://i1.bbtree.com/park/1533193912035.png)

#### 1）竞价请求接口信息（BidRequest）

| 字段    | 类型         | 默认值 | 必填 | 备注                                                                 |
|---------|--------------|--------|------|----------------------------------------------------------------------|
| bidid   | string       |        | 是   | 生成的唯一竞价ID，32个字符组成的字符串                               |
| app     | object       |        | 是   | 应用信息                                                             |
| device  | object       |        | 是   | Device对象。设备信息                                                 |
| geo     | obiect       |        | 否   | Geo对象。地理位置信息                                                |
| imp     | object array |        | 是   | Imp对象。描述正在拍卖的广告展示位置。一个出价请求可以包含多个Imp对象 |
| pmp     | object       |        | 否   | PMP对象。约定PMP交易信息                                             |
| user    | object       |        | 否   | User对象。用户信息                                                   |
| version | int32        |        | 是   | 当前协议版本号                                                       |
| bcat    | string array |        | 否   | 禁投的广告主行业，参见iab中的5.1节-广告主行业列表                    |
| ext     | object       |        | 否   | BidRequest的扩展                                                     |

##### 2）app对象（BidRequest.app）

| 字段   | 类型            | 默认值 | 必填 | 备注                                           |
|--------|-----------------|--------|------|------------------------------------------------|
| id     | string          |        | 是   | 应用ID                                         |
| name   | string          |        | 是   | 应用名称, 例：“智慧树家长版”                   |
| bundle | string          |        | 否   | android 设备为package name；ios应用为bundle id |
| ver    | string          |        | 否   | 应用版本                                       |
| cat    | array of string |        | 否   | 应用类型                                       |

##### 3）device对象（BidRequest.Device）

| 字段        | 类型   | 默认值 | 必填 | 备注                                                                                                |
|-------------|--------|--------|------|-----------------------------------------------------------------------------------------------------|
| os          | string |        | 是   | 只能是"iOS"，"Android"或"wp"（windows phone）。如果是ios设备，返回所有资源请使用https访问           |
| ua          | string |        | 是   | 设备user agent                                                                                      |
| language    | string |        | 否   | 浏览器语言，参见ISO-639-1-alpha-2                                                                   |
| dnt         | bool   | false  | 否   | 禁止跟踪用户的标志                                                                                  |
| osversion   | string |        | 是   | 操作系统版本,例：“9.0.1”                                                                            |
| make        | string |        | 否   | 生产厂商, 例：“Apple”                                                                               |
| model       | string |        | 否   | 设备型号, 例：“iPhone”                                                                              |
| ip          | string |        | 是   | 设备ipv4地址                                                                                        |
| hwv         | string |        | 否   | 设备硬件版本号, 例：“6S”是iPhone 6S的版本号                                                         |
| w           | int32  |        | 否   | 以像素为单位的设备屏幕宽度                                                                          |
| h           | int32  |        | 否   | 以像素为单位的设备屏幕高度                                                                          |
| ppi         | int32  |        | 否   | 设备屏幕像素密度，每英寸像素个数                                                                    |
| conntype    | Int32  |        | 否   | 网络连接类型，1：2G网络，2：3G网络，3：4G网络,4：wifi,5:未知                                        |
| devicetype  | 枚举   |        | 否   | 设备类型，1：移动设备，4：手机，5：平板                                                             |
| imei        | string |        | 否   | imei码明文。                                                                                        |
| mac         | string |        | 否   | mac地址明文                                                                                         |
| android_id  | string |        | 否   | Android Id 明文                                                                                     |
| adid        | string |        | 否   | iOS ADID(也叫IDFA)或Android ADID(国内手机一般没有）,明文                                            |
| didsha1     | string |        | 否   | Android为IMEI SHA1；iOS无此字段，(cdma手机传meid码)                                                 |
| dpidsha1    | string |        | 是   | Android为ANDROID ID SHA1；iOS为ADID(也叫IDFA) SHA1， 例："8a319e9fdf05dd8f571b6e0dc2dc2a8263a6974b" |
| macsha1     | string |        | 否   | mac地址 SHA1；iOS无此字段， android也只是部分机器能拿到                                             |
| plmn        | string |        | 否   | 国家运营商编号, 例:"46000"                                                                          |
| orientation | string |        | 否   | 设备屏幕方向：1: 竖向，2: 横向                                                                      |

##### 4）geo对象（BidRequest. Geo）

| 字段          | 类型   | 默认值 | 必填 | 备注                                                                                                    |
|---------------|--------|--------|------|---------------------------------------------------------------------------------------------------------|
| lat           | double |        | 否   | 纬度,例：39.9167，是WGS84坐标                                                                           |
| lng           | double |        | 否   | 经度,例：116.3833，是WGS84坐标                                                                          |
| country       | string |        | 否   | 三位字母缩写的国家代码，请参见[ISO-3166-1 Alpha-3](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3)    |
| region        | string |        | 否   | 国内是省名，美国是州的2个字母缩写，其他国家请参见[ISO-3166-2](https://en.wikipedia.org/wiki/ISO_3166-2) |
| city          | string |        | 否   | 城市名称, 例：“北京”；使用IP条件下，IP库为广协库并同步更新                                              |
| location_type | 枚举   |        | 否   | 位置来源，1：根据gps位置，2：根据IP                                                                     |
| accuracy      | int32  | 0      | 否   | 精度，单位为米                                                                                          |
| street        | string |        | 否   | 街道名称， 例：“北下关街道”                                                                             |

##### 5）imp对象（BidRequest.Imp）

| 字段           | 类型           | 默认值         | 必填 | 备注                                                                                                                           |
|----------------|----------------|----------------|------|--------------------------------------------------------------------------------------------------------------------------------|
| id             | string         |                | 是   | imp.id用于标识一次广告请求下的多个广告位发出的请求，即一个request下可能有多个imp对象，可以是不同广告形式的组和                 |
| bidfloor       | double         |                | 是   | 底价，单位是分                                                                                                                 |
| bidfloorcur    | string         | "CNY"          | 否   | 报价货币单位                                                                                                                   |
| ad_type        | int            |                | 是   | 1：banner，2：开屏，3：原生                                                                                                    |
| banner         | object         |                | 否   | Banner对象，适用于banner、开屏，用于描述广告位信息                                                                             |
| video          | object         |                | 否   | video对象                                                                                                                      |
| native         | object         |                | 否   | Native对象。video、native在一个imp下有且只有一个                                                                               |
| tagid          | string         |                | 是   | 广告位id                                                                                                                       |
| mimes          | arry of string | [“image/jpeg”] | 是   | 支持的素材类型数组                                                                                                             |
| support_aciton | 枚举           | [1]            | 否   | 广告动作类型， 1: 在app内webview打开目标链接， 2： 在系统浏览器打开目标链接, 3：打开地图，4： 拨打电话，5：播放视频, 6:App下载 |

###### 横幅信息（BidRequest.Impression.Banner）

| 字段 | 类型  | 默认值 | 必填 | 备注                                                      |
|------|-------|--------|------|-----------------------------------------------------------|
| w    | int32 |        | 是   | 广告位宽度                                                |
| h    | int32 |        | 是   | 广告位高度                                                |
| pos  | 枚举  | 0      | 否   | 广告位位置，0：未知，4：头部，5：底部，6：侧边栏，7：全屏 |

###### 视频（BidRequest.Impression.Video）

| 字段        | 类型            | 默认值 | 必填 | 备注                                                                                                                   |
|-------------|-----------------|--------|------|------------------------------------------------------------------------------------------------------------------------|
| mimes       | array of string |        | 是   | 支持的视频类型                                                                                                         |
| protocols   | array of string |        | 是   | 支持的视频响应协议，目前使用的是VAST 3.0                                                                               |
| minduration | int             |        | 否   | 最短时间，单位：秒                                                                                                     |
| maxduration | int             |        | 否   | 最长时间，单位：秒                                                                                                     |
| w           | int             |        | 是   | 广告位宽度                                                                                                             |
| h           | int             |        | 是   | 广告位高度                                                                                                             |
| pos         | int             | 0      | 否   | 广告位位置，0：未知，1:流媒体内部（如前贴、中贴、后贴），2：banner位置；3：文章段落内；4.信息流；5：不滚动出视图的位置 |

###### 原生（BidRequest.Impression.native）

| 字段   | 类型  | 默认值 | 必填 | 备注                         |
|--------|-------|--------|------|------------------------------|
| layout | Int32 | 1      | 是   | 原生样式，目前仅有 1：信息流 |
| assets | array |        | 是   | 定义的元素数组列表           |

###### 原生assets（BidRequest.Impression.native.assets）

| 字段     | 类型 | 默认值 | 必填 | 备注                                      |
|----------|------|--------|------|-------------------------------------------|
| id       | int  |        | 是   | 元素id                                    |
| required | bool | false  | 否   | 广告元素是否必须，true：必须，flase：可选 |
| img      | 对象 |        | 否   | image元素                                 |
| title    | 对象 |        | 否   | title元素                                 |
| video    | 对象 |        | 否   | Video元素                                 |
| data     | 对象 |        | 否   | Data对象                                  |

原生广告image（BidRequest.Impression.native.assets.image）

| 字段名称 | 类型 | 默认值 | 必须 | 描述                                                          |
|----------|------|--------|------|---------------------------------------------------------------|
| type     | int  |        | 否   | image元素的类型，1：应用icon，2:品牌或者企业logo，3：广告图片 |
| w        | int  |        | 否   | 宽度                                                          |
| h        | int  |        | 否   | 高度                                                          |

原生广告title（BidRequest.Impression.native.assets.title）

| 字段名称 | 类型 | 默认值 | 必须 | 描述                  |
|----------|------|--------|------|-----------------------|
| len      | int  |        | 否   | title元素最大文字长度 |

原生广告video（BidRequest.Impression.native.assets.video）

| 字段名称    | 类型 | 默认值 | 必须 | 描述               |
|-------------|------|--------|------|--------------------|
| maxduration | int  |        | 否   | 最长时间，单位：秒 |

原生广告data（BidRequest.Impression.native.assets.data）

| 字段名称 | 类型 | 默认值 | 必须 | 描述                                    |
|----------|------|--------|------|-----------------------------------------|
| type     | int  |        | 否   | 数据类型1: 描述,2：点赞个数；3.评论个数 |
| len      | int  |        | 否   | data元素最大长度                        |

##### 6）pmp对象（BidRequest.Pmp）

| 字段     | 类型   | 默认值 | 必填 | 备注                                                                                                  |
|----------|--------|--------|------|-------------------------------------------------------------------------------------------------------|
| is_pmp   | bool   |        | 是   | 当pmp对象存在，该值为true                                                                             |
| dealid   | string |        | 是   | deal唯一标识，deal约定需要通过我方平台申请，合约达成后才会体现在流量中                                |
| bidfloor | double |        | 是   | 双方商定的交易价格                                                                                    |
| cur      | string | “CNY”  | 否   | 交易货币单位，以分为计量单位，以展示计费，参见ISO-4217 alpha                                          |
| at       | int    | 1      | 否   | 交易价格结算方式，1：第一价格，2：第二价格。默认为1，多个购买对象竞价时，获胜者按照deal约定的价格成交 |

##### 7）用户信息（BidRequest.User）

| 字段   | 类型   | 默认值 | 必填 | 备注                                                        |
|--------|--------|--------|------|-------------------------------------------------------------|
| yob    | int32  |        | 否   | 生日年份，例：1995                                          |
| gender | string |        | 否   | 男：”M”, 女：”F”, 其他：”O"；该值返回的是可能性为最大的性别 |
| data[] | object |        | 否   | Data对象，用户的扩展信息                                    |

###### 用户扩展信息（BidRequest.User.Data）

| 字段    | 类型     | 默认值 | 必填 | 备注                      |
|---------|----------|--------|------|---------------------------|
| segment | 对象数组 |        | 否   | Segment对象，用户人群属性 |

###### 用户人群属性信息（BidRequest.User.Data.Segment）

| 字段  | 类型   | 默认值 | 必填 | 备注                                |
|-------|--------|--------|------|-------------------------------------|
| id    | string |        | 否   | 属性id                              |
| name  | string |        | 否   | 属性值名称。我方ADX会再后续给出list |
| value | string |        | 否   | 属性值                              |

2.2 DSP向 ADX返回出价结果及广告代码(Bid Response)
-------------------------------------------------

#### 1）接口信息（BidResponse）

| 字段    | 类型         | 默认值 | 必填 | 备注                                                                                     |
|---------|--------------|--------|------|------------------------------------------------------------------------------------------|
| bidid   | string       |        | 是   | 在BidRequest中传入的bidid                                                                |
| seatbid | object array |        | 否   | DSP的出价信息，若提出竞价则需提供一个，并且只接受一个                                    |
| nbr     | int          |        | 否   | 未竞价原因，0：未知错误，1：技术错误， 2：可疑的伪造流量，3：被屏蔽媒体，4：不匹配的用户 |

#### 2）SeatBid信息（BidResponse.SeatBid）

| 字段  | 类型         | 默认值 | 必填 | 备注                                                                                                                                      |
|-------|--------------|--------|------|-------------------------------------------------------------------------------------------------------------------------------------------|
| bid   | object array |        | 否   | Bid对象，可接受一个或多个返回                                                                                                             |
| seat  | string       |        | 否   | 参与此次竞价的代理商/广告主id。为了避免问题广告对DSP的整体影响，请尽量返回该字段                                                          |
| group | int32        | 0      | 否   | 该次竞价响应对请求的响应机制。0表示该次响应可以针对单个imp做竞价判决；1表示该次响应必须作为整体参与竞价（要么所有竞价成功，所有竞价失败） |
| ext   | object       |        | 否   | 拓展信息                                                                                                                                  |

#### 3）Bid信息（BidResponse.SeatBid.Bid）

| 字段           | 类型         | 默认值         | 必填 | 备注                                                                                                                                                                                |
|----------------|--------------|----------------|------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id             | string       |                | 是   | 由DSP提供的竞价id                                                                                                                                                                   |
| impid          | string       |                | 是   | 曝光id，需要与请求传入的imp.id对应一致                                                                                                                                              |
| price          | double       |                | 否   | 出价，单位为分，不能低于曝光最低价格，否则会被当做无效应答。目前只支持人民币；仅当PMP交易时，该字段为空                                                                             |
| dealid         | string       |                | 否   | 如果是PMP交易，请在该字段返回请求中的deal id                                                                                                                                        |
| adid           | string       |                | 是   | 物料ID，由DSP提供                                                                                                                                                                   |
| nurl           | string       |                | 否   | win notice url, GET方法调用。可以使用[宏](https://www.zybuluo.com/mdeditor#BID_MACRO)。DSP也可以使用[曝光监测链接](https://www.zybuluo.com/mdeditor#BID_WIN_NOTICE)来获取获胜通知。 |
| bundle         | string       |                | 否   | 推广应用标识。android 设备为package name；ios应用为app id。在ios中，如果deeplink唤醒失效，应用会先尝试通过 app id在应用内打开下载App Store下载界面                                  |
| iurl           | string       |                | 否   | 广告素材的图片URL。banner广告时非html素材必填                                                                                                                                       |
| w              | int32        |                | 否   | 素材宽度，该字段缺省，客户端会默认响应尺寸与请求一致，如与请求不符，务必返回该值。                                                                                                  |
| h              | int32        |                | 否   | 素材高度，该字段缺省，客户端会默认响应尺寸与请求一致，如与请求不符，务必返回该值                                                                                                    |
| cat            | string arry  |                | 否   | 广告类别，详见[IAB §6.1](http://www.iab.net/media/file/OpenRTB_API_Specification_Version2.0_FINAL.PDF)                                                                              |
| adm            | string       |                | 否   | 视频广告物料必填。 视频素材必须符合VAST 3.0规范，请参看[VAST 3.0 标准](http://www.iab.com/wp-content/uploads/2015/06/VASTv3_0.pdf)                                                  |
| admnative      | object       |                | 否   | 当请求为原生广告时，必须使用该对象响应                                                                                                                                              |
| clickurl       | string       |                | 否   | 广告点击跳转地址，允许使用[宏](https://www.zybuluo.com/mdeditor#BID_MACRO)，例[http://www..cn/ad/](http://www.zplay.cn/ad/){AUCTION_BID_ID}                                         |
| deeplink       | string       |                | 否   | 广告点击deeplink链接，在deeplink和clickurl同时存在的况下，会先访问deplink，如果唤起失败，则打开clickurl                                                                             |
| imptrackers    | string array |                | 否   | 曝光追踪地址，允许有多个追踪地址，允许使用[宏](https://www.zybuluo.com/mdeditor#BID_MACRO)                                                                                          |
| clktrackers    | string array |                | 否   | 点击追踪地址，允许有多个追踪地址，允许使用[宏](https://www.zybuluo.com/mdeditor#BID_MACRO)                                                                                          |
| html_snippet   | string array |                | 否   | html广告代码                                                                                                                                                                        |
| inventory_type | string array | [“image/jpeg”] | 否   | 返回的素材广告类型，必须与请求在请求中素材类型以内                                                                                                                                  |
| title          | string       |                | 否   | 图文广告中的标题。                                                                                                                                                                  |
| description    | string       |                | 否   | 图文广告中的描述                                                                                                                                                                    |
| action         | int          | 1              | 否   | 广告动作类型， 1: 在app内webview打开目标链接， 2： 在系统浏览器打开目标链接, 3：打开地图，4： 拨打电话，5：播放视频, 6:App下载                                                      |

###### 原生（Bidresponse.admnative）

| 字段  | 类型   | 默认值 | 必填 | 备注                               |
|-------|--------|--------|------|------------------------------------|
| id    | int    |        | 是   | 元素id，该元素需要与请求中的id一致 |
| img   | object |        | 否   | image元素                          |
| title | object |        | 否   | title元素                          |
| video | object |        | 否   | video对象                          |
| data  | object |        | 否   | data对象                           |

原生广告image（Bidresponse.admnative image）

| 字段名称 | 类型   | 默认值 | 必须 | 描述               |
|----------|--------|--------|------|--------------------|
| url      | string |        | 是   | image元素的URL地址 |
| w        | int    |        | 否   | 宽度               |
| h        | int    |        | 否   | 高度               |

原生广告title（Bidresponse.admnative .title）

| 字段名称 | 类型   | 默认值 | 必须 | 描述                |
|----------|--------|--------|------|---------------------|
| title    | string |        | 是   | title元素的内容文字 |

原生广告video（Bidresponse.admnative.video）

| 字段名称 | 类型   | 默认值 | 必须 | 描述         |
|----------|--------|--------|------|--------------|
| video    | string |        | 是   | 原生视频内容 |

原生广告data（Bidresponse.admnative.data）

| 字段名称 | 类型   | 默认值 | 必须 | 描述                                    |
|----------|--------|--------|------|-----------------------------------------|
| type     | int    |        | 否   | 数据类型1: 描述,2：点赞个数；3.评论个数 |
| label    | string |        | 否   | 数据显示的名称                          |
| value    | string |        | 是   | 数据的内容文字                          |

2.3竞价结果通知(Win Notice)
---------------------------

通过对展示监测链接中特定参数的宏替换，将广告的计费价格发送给赢得竞价的DSP平台，DSP
获取到的结算价格，是经过加密后的结算价格，每个DSP
有一个唯一的token，请联系ADX团队获取，并妥善保管。

DSP平台可根据自己的需要在链接中加入宏字段。

目前支持的宏如下：

| 字段                 | 含义                         |
|----------------------|------------------------------|
| {AUCTION_BID_ID}     | 竞价 id                      |
| {AUCTION_BID_PRICE}  | 加密成交价格                 |
| {AUCTION_IMP_ID}     | 曝光id                       |
| {AUCTION_PRICE }     | 明文成交价格                 |
| {AUCTION_SEAT \_ID } | 参与竞价的DSP方bidder id     |
| {AUCTION_TIMESTAMP}  | GMT unix timestamp, 单位为秒 |

##### 价格加密解密算法说明：

价格加密采用AES加密算法，算法的key为DSP的token，算法的iv请联系ADX获取。

2.4 示例
--------

##### 1）Banner 示例

请求banner广告，响应时将广告素材图片的URL填充到Bid的iurl字段，宽、高填充到Bid的w、h字段。暂不支持响应为html格式。

BidRequest：

``` json

{ "bidid": "shdihdkashdka11001", "app": { "id": "zhihuishu", "name": "智慧树家长端", "bundle": "package name", "ver": "P_Build_6.6.5", "cat": [ "Education", "Family & Parenting" ] }, "device": { "os": "Android", "ua": "Mozilla/5.0 (iPhone; CPU iPhone OS 11_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Mobile/15E302 bbtree_P/6.6.5 /sa-sdk-ios/sensors-verify/shence-api-uniq.bbtree.com?default ", "language": "zh-ch", "dnt": false, "osversion": "iOS 11.3.1", "make": "Apple", "model": "apple", "ip": "", "hwv": "iPhone9,1", "w": 750, "h": 1334, "ppi": 0, "conntype": 4, "devicetype": 4, "imei": "", "mac": "", "android_id": "", "adid": "9F8C4844-80DB-4946-A78B-241017CAD2A9", "didsha1": "", "dpidsha1": "", "macsha1": "", "plmn": "46000", "orientation": "1" }, "geo": { "lat": 0, "lng": 0, "country": "china", "region": "", "city": "", "location_type": 2, "accuracy": 0, "street": "" }, "imp": [ // 广告展示，可以有多个，以id 区分，暂时只有一个 { "id": "1", "bidfloor": 16, "bidfloorcur": "CNY", "ad_type": 1, "banner": { // 要求banner广告 "w": 750, "h": 1334, "pos": 0 }, "video": null, "native": null, "tagid": "11001", "mimes": [ "image/jpeg" ], "support_action": 1 } ], "pmp": null, "user": null, "version": 11, "bcat": null, "ext": null } 

``` 

BidResponse：

``` json

 { "bidid": "shdihdkashdka11001", "seatbid": [ { "bid": [ // 针对imp id 为1 的出价， 注意impid 字段 { "id": "e4d7b46d-db50-43f4-b4ea-c114c81d0064", "impid": "1", // 跟request 里的imp id 相对应。 "price": 258, "deadid": "", "adid": "640855a7-e907-479b-ab29-4a1ad5802674", "nurl": "https://www.baidu.com/s?bidid={AUCTION_BID_ID}&bidprice={AUCTION_BID_PRICE}&impid={AUCTION_IMP_ID}&price={AUCTION_PRICE}&seatid={AUCTION_SEAT_ID}&time={AUCTION_TIMESTAMP}", "bundle": "", "iurl": "https://ad5.bbtree.com/ad-test/zTb9gMqecj7_1530848062503.jpg", "w": 0, "h": 0, "cat": [ "Education", "Family & Parenting" ], "adm": "", "admnative": [ ], "clickurl": "https://www.baidu.com/s?bidid={AUCTION_BID_ID}&bidprice={AUCTION_BID_PRICE}&price={AUCTION_PRICE}", "deeplink": "", "imptrackers": [ "https://www.baidu.com/s?wd=pv" ], "clktrackers": [ "https://www.baidu.com/s?wd=click" ], "html_snippet": [ ], "inventory_type": [ "image/jpeg" ], "title": "dspmock bid responce", "description": "impid :1, bidid :e4d7b46d-db50-43f4-b4ea-c114c81d0064, adid: 640855a7-e907-479b-ab29-4a1ad5802674", "action": 2 } ], "seat": "68504472-3f2e-469d-819c-1d1eee726cec", "group": 0, "ext": null } ], "nbr": 0 }
 
``` 


##### 2）native 示例

BidRequest：

```  json

{ "bidid": "shdihdkashdka11001", "app": { "id": "zhihuishu", "name": "智慧树家长端", "bundle": "package name", "ver": "P_Build_6.6.5", "cat": [ "Education", "Family \\u0026 Parenting" ] }, "device": { "os": "Android", "ua": "Mozilla/5.0 (iPhone; CPU iPhone OS 11_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Mobile/15E302 bbtree_P/6.6.5 /sa-sdk-ios/sensors-verify/shence-api-uniq.bbtree.com?default ", "language": "zh-ch", "dnt": false, "osversion": "iOS 11.3.1", "make": "Apple", "model": "apple", "ip": "", "hwv": "iPhone9,1", "w": 750, "h": 1334, "ppi": 0, "conntype": 4, "devicetype": 4, "imei": "", "mac": "", "android_id": "", "adid": "9F8C4844-80DB-4946-A78B-241017CAD2A9", "didsha1": "", "dpidsha1": "", "macsha1": "", "plmn": "46000", "orientation": "1" }, "geo": { "lat": 0, "lng": 0, "country": "china", "region": "", "city": "", "location_type": 2, "accuracy": 0, "street": "" }, "imp": [ { "id": "1", "bidfloor": 0, "bidfloorcur": "CNY", "ad_type": 3, "banner": null, "video": null, "native": { // native请求 "layout": 1, "assets": [ { "id": 1, // 资源1 "required": true, "img": { "type": 3, "w": 400, "h": 300 }, "title": null, "video": null, "data": null }, { "id": 2, // 资源2 "required": true, "img": null, "title": { "len": 15 }, "video": null, "data": null }, { "id": 3, // 资源3 "required": true, "img": null, "title": null, "video": { "maxduration": 300 }, "data": null }, { "id": 4, // 资源4 "required": true, "img": null, "title": null, "video": null, "data": { "type": 1, "len": 1 } } ] }, "tagid": "11001", "mimes": [ "image/jpeg" ], "support_action": 1 } ], "pmp": null, "user": null, "version": 11, "bcat": null, "ext": null }
``` 


Bidresponse:

```json

 { "bidid": "shdihdkashdka11001", "seatbid": [ { "bid": [ // 针对imp id 为1的出价，注意impid字段 { "id": "8ac27959-1e85-4492-8685-6930f74b6892", "impid": "1", // 跟request里的imp id 相对应 "price": 1075, "deadid": "", "adid": "625f22a5-5def-4552-840d-1e2c1c49e56a", "nurl": "https://www.baidu.com/s?bidid={AUCTION_BID_ID}&bidprice={AUCTION_BID_PRICE}&impid={AUCTION_IMP_ID}&price={AUCTION_PRICE}&seatid={AUCTION_SEAT_ID}&time={AUCTION_TIMESTAMP}", "bundle": "", "iurl": "", "w": 0, "h": 0, "cat": [ "Education", "Family & Parenting" ], "adm": "", "admnative": [ { "id": 1, // 资源1，与request一一对应 "img": { "url": "https://ad5.bbtree.com/ad-test/zTb9gMqecj7_1530848062503.jpg", "w": 400, "h": 300 }, "title": null, "video": null, "data": null }, { "id": 2, // 资源2，与request一一对应 "img": null, "title": { "title": "this is a dspmock title" }, "video": null, "data": null }, { "id": 3, // 资源3，与request一一对应 "img": null, "title": null, "video": { "video": "https://filesystem1.bbtree.com/mp/PNmDms62MA1489571339168.mp4" }, "data": null }, { "id": 4, // 资源4，与request一一对应 "img": null, "title": null, "video": null, "data": { "type": 1, "label": "description", "value": "description" } } ], "clickurl": "https://www.baidu.com/s?bidid={AUCTION_BID_ID}&bidprice={AUCTION_BID_PRICE}&price={AUCTION_PRICE}", "deeplink": "", "imptrackers": [ "https://www.baidu.com/s?wd=pv" ], "clktrackers": [ "https://www.baidu.com/s?wd=click" ], "html_snippet": [ ], "inventory_type": [ "image/jpeg" ], "title": "dspmock bid responce", "description": "impid :1, bidid :8ac27959-1e85-4492-8685-6930f74b6892, adid: 625f22a5-5def-4552-840d-1e2c1c49e56a", "action": 2 } ], "seat": "b47dce30-5d9c-48a1-ba2b-58332d65aef4", "group": 0, "ext": null } ], "nbr": 0 }
``` 
 
2.5 沙盒测试
------------

智慧树提供了用于接入方测试自己dsp接口的链接，地址为：http://dsp.bbtree.com

页面上“竞价地址”一项填写接入方dsp接口url。



2.6 价格加解密算法
---------------------------

加解密使用AES-256-GCM算法，IV值在管理后台中显示


golang示例：

```golang

import (
	"crypto/aes"
	"crypto/cipher"
	"crypto/rand"
	"encoding/hex"
	"fmt"
	"net/url"
	"strconv"
)


func AESGCM_encrypt(src, stKey, stNonce string) (string, error) {
	key := []byte(stKey)
	plaintext := []byte(src)

	block, err := aes.NewCipher(key)
	if err != nil {
		return "", err
	}
	nonce, _ := hex.DecodeString(stNonce)
	aesgcm, err := cipher.NewGCM(block)
	if err != nil {
		return "", err
	}
	ciphertext := aesgcm.Seal(nil, nonce, plaintext, nil)
	return fmt.Sprintf("%x", ciphertext), nil
}

func AESGCM_decrypt(src, stKey, stNonce string) (string, error) {
	key := []byte(stKey)
	ciphertext, _ := hex.DecodeString(src)

	nonce, _ := hex.DecodeString(stNonce)
	block, err := aes.NewCipher(key)
	if err != nil {
		return "", err
	}

	aesgcm, err := cipher.NewGCM(block)
	if err != nil {
		return "", err
	}

	plaintext, err := aesgcm.Open(nil, nonce, ciphertext, nil)
	if err != nil {
		panic(err.Error())
	}

	return string(plaintext), nil
}


```

java代码示例：
```java

import org.apache.commons.codec.binary.Hex;

import javax.crypto.Cipher;
import javax.crypto.spec.GCMParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;


public class AESGCMHelper {

    public static String encrypt(String key, String sIv, String data) throws Exception {
        Cipher cipher = initCipher(Cipher.ENCRYPT_MODE, key, sIv);
        byte[] encrypt = cipher.doFinal(data.getBytes());
        return Hex.encodeHexString(encrypt);
    }

    public static String decrypt(String key, String sIv, String data) throws Exception {
        Cipher cipher = initCipher(Cipher.DECRYPT_MODE, key, sIv);
        byte[] os = Hex.decodeHex(data.toCharArray());
        return new String(cipher.doFinal(os), StandardCharsets.UTF_8);
    }

    private static Cipher initCipher(int mode, String key, String iv) throws Exception {
        Cipher cipher =  Cipher.getInstance("AES/GCM/NoPadding", "SunJCE");
        cipher.init(mode, new SecretKeySpec(key.getBytes(), "AES"), new GCMParameterSpec(128, (byte[]) new Hex().decode(iv)));
        return cipher;
    }

}


```



python代码示例：

```python

from cryptography.hazmat.primitives.ciphers.aead import AESGCM


class AESGCMHelper:
    
    def __init__(self, key, iv):
        self.key = key
        self.sIv = iv
        self.iv = bytes(bytearray.fromhex(self.sIv))
        self.aesgcm = AESGCM(key)

    def encrypt(self, text):
        ct = self.aesgcm.encrypt(self.iv, text, None)
        return ct.encode("hex")

    def decrypt(self, text):
        return self.aesgcm.decrypt(self.iv, text.decode("hex"), None)

```


c代码示例：
```c


/**
  AES encryption/decryption demo program using OpenSSL EVP apis

  gcc -Wall openssl_aes.c -lcrypto
**/

#include <openssl/aes.h>
#include <openssl/err.h>
#include <openssl/evp.h>
#include <string.h>

void handleErrors(void) {
  unsigned long errCode;

  printf("An error occurred\n");
  while (errCode = ERR_get_error()) {
    char *err = ERR_error_string(errCode, NULL);
    printf("%s\n", err);
  }
  abort();
}

int encrypt(const unsigned char *plaintext, int plaintext_len, const unsigned char *aad,
            int aad_len, const unsigned char *key, const unsigned char *iv, int iv_len,
            unsigned char *ciphertext, unsigned char *tag) {
  EVP_CIPHER_CTX *ctx;

  int len;
  int ciphertext_len;

  /* Create and initialise the context */
  if (!(ctx = EVP_CIPHER_CTX_new())) handleErrors();

  /* Initialise the encryption operation. */
  if (1 != EVP_EncryptInit_ex(ctx, EVP_aes_256_gcm(), NULL, NULL, NULL))
    handleErrors();

  /* Set IV length if default 12 bytes (96 bits) is not appropriate */
  if (1 != EVP_CIPHER_CTX_ctrl(ctx, EVP_CTRL_GCM_SET_IVLEN, iv_len, NULL))
    handleErrors();

  /* Initialise key and IV */
  if (1 != EVP_EncryptInit_ex(ctx, NULL, NULL, key, iv)) handleErrors();

  /* Provide any AAD data. This can be called zero or more times as
   * required
   */
  if (1 != EVP_EncryptUpdate(ctx, NULL, &len, aad, aad_len)) handleErrors();

  /* Provide the message to be encrypted, and obtain the encrypted output.
   * EVP_EncryptUpdate can be called multiple times if necessary
   */
  if (1 != EVP_EncryptUpdate(ctx, ciphertext, &len, plaintext, plaintext_len))
    handleErrors();
  ciphertext_len = len;

  /* Finalise the encryption. Normally ciphertext bytes may be written at
   * this stage, but this does not occur in GCM mode
   */
  if (1 != EVP_EncryptFinal_ex(ctx, ciphertext + len, &len)) handleErrors();
  ciphertext_len += len;

  /* Get the tag */
  if (1 != EVP_CIPHER_CTX_ctrl(ctx, EVP_CTRL_GCM_GET_TAG, 16, tag))
    handleErrors();

  /* Clean up */
  EVP_CIPHER_CTX_free(ctx);

  return ciphertext_len;
}

int decrypt(unsigned char *ciphertext, int ciphertext_len, const unsigned char *aad,
            int aad_len, unsigned char *tag, const unsigned char *key,
            const unsigned char *iv, int iv_len, unsigned char *plaintext) {
  EVP_CIPHER_CTX *ctx;
  int len;
  int plaintext_len;
  int ret;

  /* Create and initialise the context */
  if (!(ctx = EVP_CIPHER_CTX_new())) handleErrors();

  /* Initialise the decryption operation. */
  if (!EVP_DecryptInit_ex(ctx, EVP_aes_256_gcm(), NULL, NULL, NULL))
    handleErrors();

  /* Set IV length. Not necessary if this is 12 bytes (96 bits) */
  if (!EVP_CIPHER_CTX_ctrl(ctx, EVP_CTRL_GCM_SET_IVLEN, iv_len, NULL))
    handleErrors();

  /* Initialise key and IV */
  if (!EVP_DecryptInit_ex(ctx, NULL, NULL, key, iv)) handleErrors();

  /* Provide any AAD data. This can be called zero or more times as
   * required
   */
  if (!EVP_DecryptUpdate(ctx, NULL, &len, aad, aad_len)) handleErrors();

  /* Provide the message to be decrypted, and obtain the plaintext output.
   * EVP_DecryptUpdate can be called multiple times if necessary
   */
  if (!EVP_DecryptUpdate(ctx, plaintext, &len, ciphertext, ciphertext_len))
    handleErrors();
  plaintext_len = len;

  /* Set expected tag value. Works in OpenSSL 1.0.1d and later */
  if (!EVP_CIPHER_CTX_ctrl(ctx, EVP_CTRL_GCM_SET_TAG, 16, tag)) handleErrors();

  /* Finalise the decryption. A positive return value indicates success,
   * anything else is a failure - the plaintext is not trustworthy.
   */
  ret = EVP_DecryptFinal_ex(ctx, plaintext + len, &len);

  /* Clean up */
  EVP_CIPHER_CTX_free(ctx);

  if (ret > 0) {
    /* Success */
    plaintext_len += len;
    return plaintext_len;
  } else {
    /* Verify failed */
    return -1;
  }
}

int main(int arc, char *argv[]) {
  OpenSSL_add_all_algorithms();
  ERR_load_crypto_strings();

  /* Set up the key and iv. Do I need to say to not hard code these in a real
   * application? :-) */

  /* A 256 bit key */
  static const unsigned char key[] = "01234567890123456789012345678901";

  /* A 128 bit IV */
  static const unsigned char iv[] = "0123456789012345";

  /* Message to be encrypted */
  unsigned char plaintext[] = "In theory, theory and practice are the same. In practice, they’re not";

  /* Some additional data to be authenticated */
  static const unsigned char aad[] = "Some AAD data";

  /* Buffer for ciphertext. Ensure the buffer is long enough for the
   * ciphertext which may be longer than the plaintext, dependant on the
   * algorithm and mode
   */
  unsigned char ciphertext[128];

  /* Buffer for the decrypted text */
  unsigned char decryptedtext[128];

  /* Buffer for the tag */
  unsigned char tag[16];

  int decryptedtext_len = 0, ciphertext_len = 0;

  printf("origin plaintext:\n%s\n", plaintext);
  /* Encrypt the plaintext */
  ciphertext_len = encrypt(plaintext, strlen(plaintext), aad, strlen(aad), key,
                           iv, strlen(iv), ciphertext, tag);

  /* Do something useful with the ciphertext here */
  printf("Ciphertext is:\n");
  BIO_dump_fp(stdout, ciphertext, ciphertext_len);
  printf("Tag is:\n");
  BIO_dump_fp(stdout, tag, 14);

  /* Mess with stuff */
  /* ciphertext[0] ^= 1; */
  /* tag[0] ^= 1; */

  /* Decrypt the ciphertext */
  decryptedtext_len = decrypt(ciphertext, ciphertext_len, aad, strlen(aad), tag,
                              key, iv, strlen(iv), decryptedtext);

  if (decryptedtext_len < 0) {
    /* Verify error */
    printf("Decrypted text failed to verify\n");
  } else {
    /* Add a NULL terminator. We are expecting printable text */
    decryptedtext[decryptedtext_len] = '\0';

    /* Show the decrypted text */
    printf("Decrypted text is:\n");
    printf("%s\n", decryptedtext);
  }

  /* Remove error strings */
  ERR_free_strings();

  return 0;
}


```



php代码示例：
```php
待补充

```
