---
date: '2017-04-13 10:21:07'
draft: false
lastmod: '2017-11-30 16:59:49'
title: 阿里云DDNS客户端
url: aliyun-ddns-client
categories:
- 项目
sticky: 1
---

万网注册的域名有福了，背靠阿里这个大金主，最近推出了不少有用的功能，甩出新网之类的域名注册商不止一条街了。朋友之前一直使用DNSPod和花生壳来构筑VPN内网的域名解析，可TPLINK自带的路由器花生壳功能太不稳定了，DNSPod也拒绝加入那些免费的二级域名了，只能想看看有没有办法在他已有的域名上实现DDNS的二级域名解析了。凑巧看见万网和阿里云推出的云解析功能支持API调用，既然API可以实现域名管理的基本全部功能，那实现一个DDNS的客户端就可以完美替代花生壳的解决方案了。可惜没有找到现成的阿里云解析客户端，参考的实现也只有JAVA实现，虽然有完备的API开发文档，但对认证签名一块用Python实现起来还是有些含糊不清的地方。这里我详细给出了针对阿里云解析API(2015-01-09)版的认证这一块的代码，完整DDNS客户端请前往github搜索aliyun-ddns-client

另外要能够调用云解析的API，用户需要从阿里云获取Access ID和Access Key,步骤如下:

1. 登陆 阿里云官方网站
2. 进入页面上方菜单【用户中心】
3. 进入上方二级菜单【我的服务】
4. 进入左侧菜单【安全认证】
5. 页面所显示的Access Key ID和Access Key Secret则为所需ID和Key

完整云解析OpenAPI文档下载: 万网--帮助中心--域名--域名解析--OpenAPI文档下载

#### 签名参考Python代码

    import urllib
    import hashlib
    import hmac
    import uuid

    def getSignatureNonce(self):
        """
        Unique random value
        """
        return uuid.uuid4()

    def getTimeStamp(self):
        """
        ISO8601 standard: YYYY-MM-DDThh:mm:ssZ, e,g:2015-0109T12:00:00Z (UTC Timezone)
        @return string
        """
        return  datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%SZ')

    def getCommonParams(self):
        commonParams = {
            'Format': 'json',
            'Version': '2015-01-09',
             # 这里accessKeyId是阿里云获取的Access Key ID
            'AccessKeyId': self.accessKeyId,
            'TimeStamp': self.getTimeStamp(),
            'SignatureMethod': 'HMAC-SHA1',
            'SignatureNonce': self.getSignatureNonce(),
            'SignatureVersion': "1.0",
        }

        return commonParams

    def getSignature(self, httpMethod, params):
        """
        1. params = combine specific params and common params
        2. sortedParams = sort params by key
        3. queryString = urlencode sortedParams
        4. signString = HTTPMethod + "&amp;" + urlencode "/" + quote_plus queryString
        5. hashValue = sha1 with hashKey against signString
        6. base64HashValue = base64 encoding hashValue
        7. signature = urlencode base64HashValue
        @return string
        """
        # 获取API文档中的公共参数, 并与每个不同方法中的特殊参数合并
        params.update(self.getCommonParams())
        # 对应API文档中的1.11-1.a)步骤: 按参数Key字母排序
        sortedParams = sorted(params.items())
        # 对应API文档中的1.11-1.b)步骤: 对参数key和value进行URL编码
        canonString = urllib.urlencode(sortedParams)
        # 对应API文档中的1.11-2步骤: 构造签名所要使用的规范化字符串
        signString = httpMethod + "&amp;" + urllib.quote_plus("/") + "&amp;" + urllib.quote_plus(canonString)
        # 对应API文档中的1.11-3步骤: 对签名字符串计算HMAC值, 这里self.hashKey=阿里云获取的Access Key Secret + "&amp;"
        hashValue = hmac.new(self.hashKey, signString, hashlib.sha1).digest()
        # 对应API文档中的1.11-4步骤: 对HMAC值进行base64编码
        # 注意: 有些第三方http库比如requests会在发送请求时自动对所有参数再进行一次URL编码, 所以这里忽略1.11-5步骤
        signature=hashValue.encode("base64").rstrip('n')

        return signature
