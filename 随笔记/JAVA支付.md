# JAVA支付

## 支付宝

签约功能列表：

![二维码支付.png](https://gitee.com/kongyin/picture_bed/raw/master/wx_picture/174516_1f8bc13b_87650.png)

- 电脑支付：https://docs.open.alipay.com/270
- 扫码支付：https://docs.open.alipay.com/194
- 手机支付：https://docs.open.alipay.com/203
- APP支付 : https://docs.open.alipay.com/54/106370/
- 沙箱环境：https://docs.open.alipay.com/200/105311/
- 支付宝公钥参数：https://openclub.alipay.com/read.php?tid=2190&fid=69
- RSA(SHA1)升级为RSA(SHA256)：https://opensupport.alipay.com/support/knowledge/20069/201602242782
- 参数zfbinfo.properties

```
支付宝网关名、partnerId和appId
open_api_domain = https://openapi.alipay.com/gateway.do
mcloud_api_domain = http://mcloudmonitor.com/gateway.do
此处请填写你的PID
pid =XXXXXXXXXXXXXX
此处请填写你当面付的APPID 
appid =XXXXXXXXXXXXXX

RSA私钥、公钥和支付宝公钥
private_key = XXXXXXXXXXXXXX
public_key = XXXXXXXXXXXXXX
alipay_public_key = XXXXXXXXXXXXXX

当面付最大查询次数和查询间隔（毫秒）
max_query_retry = 5
query_duration = 5000

当面付最大撤销次数和撤销间隔（毫秒）
max_cancel_retry = 3
cancel_duration = 2000

交易保障线程第一次调度延迟和调度间隔（秒）
heartbeat_delay = 5
heartbeat_duration = 900
```

支付宝的SDK-alipay-sdk-java这里下载： https://docs.open.alipay.com/54/103419/

大家比较好奇的alipay-trade-sdk从这里下载的TradePayDemo项目中的额lib下面，不过是16年的，目前来说还是可以使用的： https://docs.open.alipay.com/54/104506/

## 微信

- H5支付：https://pay.weixin.qq.com/wiki/doc/api/H5.php?chapter=15_1
- 公众号支付：https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_1
- 扫码支付模式一：https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=6_4
- 扫码支付模式二：https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=6_5
- 微信退款说明：https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=4_3
- 网络设置指引：https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=23_2
- HTTPS服务器配置:https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=10_4
- 参数wxinfo.properties
- 微信网页授权部分，向微信申请测试号：http://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421137522

```
服务号的应用ID
APP_ID = XXXXXXXXXXXXXX
服务号的应用密钥
APP_SECRET = XXXXXXXXXXXXXX
服务号的配置token
TOKEN = XXXXXXXXXXXXXX
商户号
MCH_ID = XXXXXXXXXXXXXX
API密钥
API_KEY = XXXXXXXXXXXXXX
签名加密方式
SIGN_TYPE = MD5
微信支付证书名称
CERT_PATH = apiclient_cert.p12
```

## 银联

- 开放平台：https://open.unionpay.com/
- 商家中心：https://merchant.unionpay.com/join/
- 测试账号：https://blog.52itstyle.vip/archives/326/
- 证书问题(QA)：https://open.unionpay.com/ajweb/help/faq/list?id=174&level=0&from=0