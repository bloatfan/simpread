> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [tech.meituan.com](https://tech.meituan.com/2015/10/19/mock-server-in-action.html)

> 美团 EP 团队开发的 Mock Server，是用来模拟被测系统外部依赖模块行为的通用服务。本文介绍了 Mock Server 的整体结构及在美团使用的典型案例。

背景
--

在美团服务端测试中，被测服务通常依赖于一系列的外部模块，被测服务与外部模块间通过 REST API 或是 Thrift 调用来进行通信。要对被测服务进行系统测试，一般做法是，部署好所有外部依赖模块，由被测服务直接调用。然而有时被调用模块尚未开发完成，或者调用返回不好构造，这将影响被测系统的测试进度。为此我们需要开发桩模块，用来模拟被调用模块的行为。最简单的方式是，对于每个外部模块依赖，都创建一套桩模块。然而这样的话，桩模块服务将非常零散，不便于管理。Mock Server 为解决这些问题而生，其提供配置 request 及相应 response 方式来实现通用桩服务。本文将专门针对 REST API 来进行介绍 Mock Server 的整体结构及应用案例。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/35221602.png)

"mock server 背景"

名词解释
----

*   Mock 规则：定义 REST API 请求及相应模拟响应的一份描述。
*   Mock 环境：根据请求来源 IP 来区分的 Mock 规则分组。Mock Server 可以定义多套 Mock 环境，各套环境间相互隔离。同一个 IP 只能对应一个 Mock 环境，不同的 IP 可以对应同一个 Mock 环境。

整体结构
----

Mock Server 由 web 配置页面 Mock Admin 及通用 Mock Stub 组成：Mock Admin 提供了 web UI 配置页面，可以增加 / 删除请求来源 IP 到 Mock 环境的映射，可以对各套环境中的 Mock 规则进行 CRUD 操作；Mock Stub 提供通用桩服务，对被测系统的各类 REST API 请求调用，返回预先定义好的模拟响应。为了提高桩服务的通吐，使得桩服务能在被测系统压力测试中得到好的表现，我们开启了 5 个桩服务，通过 Nginx 做负载均衡。Mock Server 的整体结构如下图所示。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/acd7db25.png)

"mock server 整体结构"

### 数据存储

*   将请求来源 IP 到 Mock 环境名的映射存储到 mock-env.conf 中，mock-env.conf 的每一行定义了一条映射，如： 192.168.3.68 闫帅的测试机环境 这条映射表明来源是`192.168.3.68`的请求，使用 Mock 环境名为`闫帅的测试机环境`的 Mock 规则。
*   将配置的 Mock 规则存放到 <对应 Mock 环境名>.xml 中，下面部分展示了 Mock 规则的存储格式。

```
<configuration>
...
  <mock >
    <request>
      <uri>/api/test/.*</uri>
      <method>GET|POST|PUT|DELETE</method>
      <parameters>
        <parameter />
        ...
      </parameters>
      <headers>
        <header 1E[0-9a-zA-Z]+"/>
        ...
      </headers>
    </request>
    <response delay="1000" real="false">
      <statusCode>200</statusCode>
      <format>application/json;charset=UTF-8</format>
      <customHeaders>
        ...
      </customHeaders>
      <body>{"name":"闫帅"}</body>
    </response>
  </mock>
...
</configuration>


```

### Mock Stub

当请求发送到 Mock Stub 时，Mock Stub 会根据请求的来源 IP 找到对应的独立环境名，然后根据独立环境名获取所有预定义的 Mock 规则，遍历这些 Mock 规则，如果找到一条规则与接受到的请求匹配，那么返回预定义的模拟响应。如果找不到规则匹配，那么返回 404 错误。其中，规则匹配是根据请求中的 uri/method/headers/parameters/body 是否与 Mock 规则中定义的对应字段正则匹配来定的。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/5d307678.png)

"mock stub 工作原理"

### Mock Admin

*   打开 Mock Admin 配置页面，如果尚未映射来源 IP 地址到环境，则点击环境列表导航链接，进入环境列表页面，点击添加，输入源 IP 及环境名，点击确定按钮，实现源 IP 到所设环境的映射。
    
    ![](https://raw.githubusercontent.com/bloatfan/PicGo/master/95a0dd7a.png)
    
    "mock admin 环境列表"
    

* 点击规则列表，规则列表页面将默认罗列出 default 环境的所有 Mock 规则（如 “语音登录 code 获取” 规则）。重新选择环境，可以罗列出所选环境中的 Mock 规则。每个 Mock 规则都处于详细信息展开的状态。点击 “全部折叠” 按钮，将把所有的规则详细信息给隐藏；点击 “全部展开” 按钮，将把所有的规则详细信息给展开。点击 “只显示本人创建的规则”，将过滤得到 mis 账户用户创建的规则。点击“按创建时间排序” 的开关按钮，将实现 Mock 规则的升序 / 降序显示。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/9ca83c82.png)

"mock admin 规则列表"

* 点击导航栏的 “新建规则” 选项，可以创建一个 Mock 规则，需要填写规则名称、请求及响应，并选中环境。对于请求，需要填写 URL 及勾选 Method，如果要求对于符合某种规则的请求才被 Mock，则填写对应的 Headers/Parameters/Body Like，这些数据都是正则匹配的形式。对于响应，如果勾选了 “返回真实响应”，则只需要关注延时（延时是指返回请求需要的 sleep 时间，单位是毫秒）。此时需要将请求的 URL 地址给写完整了，需要包含 host(IP) 及 port，不能只是 path。如果不勾选“返回真实响应”，则将返回模拟响应。Status Code 填写返回码，比如 200，404；Format 选择返回数据的格式，比如 json，html 等；还可以返回自定义的 Headers 及响应的 Body。点击新建按钮以后，如果创建成功，则提示成功创建，并跳转到规则列表页面。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/31add35b.png)

"mock admin 新建规则"

* 对于一个创建好的 Mock 规则，可以点击 Action 下拉菜单，进行操作。如果点击 “克隆”，则跳转到“新建规则” 页面，并将克隆的 Mock 规则信息给填充进去；点击“编辑”，则跳转到更新页面，更新页面填充了要编辑的 Mock 规则信息；点击“删除”，则弹出确认删除对话框，点击确定按钮，将删除此规则；点击取消按钮，则取消删除操作。如果此 Mock 规则处于详细信息展开状态，则可点击折叠来隐藏详细信息；如果处于详细信息折叠状态，则可点击展开选项，将显示详细信息。上移选项，可以将 Mock 规则上移一位；下移选项，可以将 Mock 规则下移一位。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/5bfc3190.png)

"mock admin 规则操作"

使用方式
----

1.  修改被测服务的 HTTP 依赖，将依赖的 IP 和端口分别设置为 mock.ep.sankuai.com 和 80，并重启被测服务；
2.  创建 Mock 规则；
3.  调用被测服务的 API，被测服务将调用 Mock 服务；
4.  删除 Mock 规则（可选）。

### 编程使用

创建 / 删除 Mock 规则，除了可通过 Mock Admin 页面配置外，Mock Server 还提供了 SDK 方式，用户可以通过编码来使用 Mock Server。

* 在 Maven 工程 pom.xml 中添加 mock-client 依赖

```
<dependency>
  <groupId>com.sankuai.meituan.ep.mockserver</groupId>
  <artifactId>mock-client</artifactId>
  <version>1.0.6</version>
</dependency>


```

*   创建 / 删除 Mock 规则

```
MockRule rule = new MockRule();
rule.setMockName("test-" + System.currentTimeMillis()); 
rule.setAuthor("yanshuai"); 
MockRequest mockRequest = new MockRequest();
mockRequest.setUri("/api/test/" + System.currentTimeMillis()); 

mockRequest.setMethod("POST|GET");

List<MockRequestHeader> mockRequestHeaders = new ArrayList<MockRequestHeader>();
mockRequestHeaders.add(new MockRequestHeader("device", "android2.3"));
mockRequest.setHeaders(mockRequestHeaders);

List<MockRequestParameter> mockRequestParameters = new ArrayList<MockRequestParameter>();
mockRequestParameters.add(new MockRequestParameter("wd", "123.*"));
mockRequestParameters.add(new MockRequestParameter("version", "v1"));
mockRequest.setParameters(mockRequestParameters);
rule.setMockRequest(mockRequest);
MockResponse mockResponse = new MockResponse();
mockResponse.setDelay(1000L); 
mockResponse.setStatusCode(200); 

mockResponse.setFormat("application/json;charset=UTF-8");
List<MockResponseHeader> mockResponseHeaders = new ArrayList<MockResponseHeader>(); 
mockResponseHeaders.add(new MockResponseHeader("customHeaderName", "customHeaderValue"));
mockResponse.setMockResponseHeaders(mockResponseHeaders);
mockResponse.setBody("{\"code\":200}"); 
rule.setMockResponse(mockResponse);
 

final MockClient client = new MockClient();
String id = client.addRule("default", rule); 
 


 

client.removeRule("default", id); 


```

典型案例
----

1.  相同 Mock 环境，同一接口，不同参数，可以有不同的 Mock 结果 按照下图，依次创建这两条规则（顺序相关），然后在 default 环境对应的机器上，访问 [http://mock.ep.sankuai.com/user/v1/info?token=fake](http://mock.ep.sankuai.com/user/v1/info?token=fake)，返回 {“code”:401,“type”:“sys_err_auth_fail”,“message”:“invalid token”}；访问 [http://mock.ep.sankuai.com/user/v1/info?token=other](http://mock.ep.sankuai.com/user/v1/info?token=other)，返回 {“user”: {“id”: 29008301,“mobile”: “15001245907”,“isBindedMobile”: 1}}。
    
    ![](https://raw.githubusercontent.com/bloatfan/PicGo/master/28866c9f.png)
    
    "案例 1_1"
    

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/d6c393e1.png)

"案例 1_2"