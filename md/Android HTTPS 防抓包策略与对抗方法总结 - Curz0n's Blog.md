> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [curz0n.github.io](https://curz0n.github.io/2020/08/15/android-ssl-and-intercept/#0x04-%E7%BB%93%E8%AF%AD)

> 在对移动 App 安全测试时，通常第一步是对 App 的网络请求报文进行抓包。

0x01 前言
-------

在对移动 App 安全测试时，通常第一步是对 App 的网络请求报文进行抓包。因为 App 客户端可以实现对 HTTPS 证书进行校验以防止抓包，为了对抗证书校验，一般会采用信任代理工具签发的证书、安装插件绕过证书检测等方法。本文将对常用的证书校验方式及相应的对抗方法做个简单总结。

0x02 了解 HTTPS 证书
----------------

在 HTTPS 握手建立链接时，会使用非对称算法协商通讯使用的加密密钥，非对称加密算法需要两个密钥: 公钥和私钥，通常我们所说的 HTTPS 证书就可以简单理解成公钥，为了确保客户端接受到的证书的真实性，于是诞生了 CA（Certification Authority）机构，CA 机构发布的证书称为 CA 证书，基于 CA 机构的权威性，用户可以无条件的信任该机构发布的证书，为了鉴别 CA 证书的真实性，防止 CA 证书被伪冒，同时避免套娃，因此在操作系统中内置了一个信任库，里面保存了可信任的 CA 证书集合，也称为系统根证书，用于校验服务端返回的证书的真实性。在 Android 系统中，可以点击`设置-系统安全-加密与凭据-信任的凭据`查看默认信任的 CA 证书。

0x03 证书校验实现 & 对抗
----------------

在使用 HTTPS 协议进行网络通讯时，对 HTTPS 证书校验有多种处理方式，包括忽略证书链校验、系统证书链校验、证书绑定（SSL Pinning，代码校验和配置文件绑定）、双向校验等方式，下面写个 Demo APP 实现每种校验方法，并介绍下相应的破解方法。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/0.png)

### 1. 忽略证书校验

在对 HTTPS 证书验证时，可以通过代码实现信任所有证书，以 OkHttp 网络框架为例，发起一个忽略 HTTPS 证书校验的请求实现如下:

```
/*
        * https协议
        * 忽略证书验证
         */
        mHttpsConnectTrustButton = findViewById(R.id.httpsConnectTrust);
        mHttpsConnectTrustButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                new Thread(new Runnable(){
                    @RequiresApi(api = Build.VERSION_CODES.KITKAT)
                    @Override
                    public void run() {
                        OkHttpClient mClient = client.newBuilder().sslSocketFactory(HttpsTrustAllCerts.createSSLSocketFactory(),new HttpsTrustAllCerts()).hostnameVerifier(new HttpsTrustAllCerts.TrustAllHostnameVerifier()).build();

                        Request request = new Request.Builder()
                                .url("https://www.baidu.com/?q=trustAllCerts")
                                .build();
                        Message message = new Message();
                        try (Response response = mClient.newCall(request).execute()) {
                            message.what = 4432001;
                            message.obj = "请求成功: " + response.code();
                            mHandler.sendMessage(message);
                        } catch (IOException e) {
                            message.what = 4434001;
                            message.obj = e.getLocalizedMessage();
                            mHandler.sendMessage(message);
                            e.printStackTrace();
                        }
                    }
                }).start();
            }
        });
```

HttpsTrustAllCerts 对象实现如下，重写 checkClientTrusted、checkServerTrusted、verify 等方法，使其验证逻辑为空，达到信任所有证书的效果:

```
public class HttpsTrustAllCerts implements X509TrustManager {


    @Override
    public void checkClientTrusted(X509Certificate[] chain, String authType) {
        //TODO
    }

    @Override
    public void checkServerTrusted(X509Certificate[] chain, String authType) {
        //TODO
    }

    @Override
    public X509Certificate[] getAcceptedIssuers() {
        return new X509Certificate[0]; //返回长度为0的数组，相当于return null
    }


    public static SSLSocketFactory createSSLSocketFactory() {
        SSLSocketFactory sSLSocketFactory = null;
        try {
            SSLContext sc = SSLContext.getInstance("TLS");
            sc.init(null, new TrustManager[]{new HttpsTrustAllCerts()},new SecureRandom());
            sSLSocketFactory = sc.getSocketFactory();
        } catch (Exception e) {
        }
        return sSLSocketFactory;
    }


    public static class TrustAllHostnameVerifier implements HostnameVerifier {
        @Override
        public boolean verify(String s, SSLSession sslSession) {
            return true;
        }
    }
}
```

忽略证书校验的 https 请求和使用 http 协议的请求报文一样，可以通过直接设置系统代理，抓取请求报文:

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1.png)

### 2. 系统证书校验

OkHttp 框架发起 https 请求时，默认就是使用的系统信任库证书链对服务端返回的证书进行验证，https 请求实现如下:

```
/*
        * https协议
        * 默认证书链校验，只信任系统CA(根证书)
        *
        * OKHTTP默认的https请求使用系统CA验证服务端证书（Android7.0以下还信任用户证书，Android7.0开始默认只信任系统证书）
         */
        mHttpsConnectButton = findViewById(R.id.httpsConnect);
        mHttpsConnectButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                new Thread(new Runnable(){
                    @RequiresApi(api = Build.VERSION_CODES.KITKAT)
                    @Override
                    public void run() {
                        Request request = new Request.Builder()
                                .url("https://www.baidu.com/?q=defaultCerts")
                                .build();
                        Message message = new Message();
                        try (Response response = client.newCall(request).execute()) {
                            message.what = 4432002;
                            message.obj = "请求成功: " + response.code();
                            mHandler.sendMessage(message);
                        } catch (IOException e) {
                            message.what = 4434002;
                            message.obj = e.getLocalizedMessage();
                            mHandler.sendMessage(message);
                            e.printStackTrace();
                        }
                    }
                }).start();
            }
        });
```

设置系统代理，抓取该数据报文，会发现 App 报错 _java.security.cert.CertPathValidatorException: Trust anchor for certification path not found._，抓包工具 burp 会提示 _certificate_unknown_，其原因在于 burp 拦截 https 流量时，会同时充当客户端和服务端。在面向 App 时，此时 burp 充当服务端，https 协议握手过程中，burp 会将自己的证书发送给 App，因为 App 使用系统信任库里面的证书进行校验，而 burp 的证书不在系统信任库中，所以导致 https 握手失败，报错 CertPathValidatorException。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2.png)

根据上述分析，对抗使用系统证书校验的方式，只需要将抓包工具的证书安装到系统信任库里面即可正常抓包。需要注意的是，在 Android 7.0 以前，应用默认会信任系统证书和用户证书，Android 7.0 开始，默认只信任系统证书，在 root 环境下，将证书导入系统目录的方法可参考 [Configuring Burp Suite With Android Nougat](https://blog.ropnop.com/configuring-burp-suite-with-android-nougat/)，将抓包工具的证书添加到手机系统证书以后，设置代理，抓包效果如下:

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/3.png)

### 3. SSL PINNING

[SSL PINNING](https://developer.android.com/training/articles/security-config) 是 Google 官方推荐的校验方式，原理是在客户端中预先设置好证书信息，握手时与服务端返回的证书进行比较，以确保服务端返回的证书的真实性，实现方式有两种，一种是在代码层实现，一种是通过 network_security_config.xml 配置文件完成。

**代码校验**

以 OkHttp 网络框架为例，校验证书公钥的示列代码如下

```
/*
        * SSL Pinning
        * 公钥绑定，验证证书公钥，还可以add验证其他要素
         */
        mSSLPinningButton = findViewById(R.id.sslPinning);
        mSSLPinningButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                new Thread(new Runnable(){
                    @RequiresApi(api = Build.VERSION_CODES.KITKAT)
                    @Override
                    public void run() {
                        final String CA_DOMAIN = "www.baidu.com";
                        //获取目标公钥: openssl s_client -connect www.baidu.com:443 -servername www.baidu.com | openssl x509 -pubkey -noout | openssl rsa -pubin -outform der | openssl dgst -sha256 -binary | openssl enc -base64
                        final String CA_PUBLIC_KEY = "sha256//558pd1Y5Vercv1ZoSqOrJWDsh9sTMEolM6T8csLucQ=";
                        //只校验公钥
                        CertificatePinner pinner = new CertificatePinner.Builder()
                                .add(CA_DOMAIN, CA_PUBLIC_KEY)
                                .build();
                        OkHttpClient pClient = client.newBuilder().certificatePinner(pinner).build();
                        Request request = new Request.Builder()
                                .url("https://www.baidu.com/?q=SSLPinningCode")
                                .build();
                        Message message = new Message();
                        try (Response response = pClient.newCall(request).execute()) {
                            message.what = 4432003;
                            message.obj = "请求成功: " + response.code();
                            mHandler.sendMessage(message);
                        } catch (IOException e) {
                            message.what = 4434003;
                            message.obj = e.getLocalizedMessage();
                            mHandler.sendMessage(message);
                            e.printStackTrace();
                        }
                    }
                }).start();
            }
        });
```

**配置文件**

通过`res/xml/network_security_config.xml`配置文件对证书进行校验是官方推荐使用的方法，配置方式可以细分为两种，使用证书校验和公钥校验，具体示列如下:

```
<!--证书校验-->
    <domain-config>
        <domain includeSubdomains="true">bing.com</domain>
        <trust-anchors>
            <!--获取证书: openssl s_client -connect bing.com:443 -servername bing.com | openssl x509 -out bing.pem-->
            <certificates src="@raw/bing"/>
        </trust-anchors>

    </domain-config>

    <!--公钥校验-->
    <domain-config>
        <domain includeSubdomains="true">so.com</domain>
        <!--so.com公钥校验
        获取公钥: openssl s_client -connect so.com:443 -servername so.com | openssl x509 -pubkey -noout | openssl rsa -pubin -outform der | openssl dgst -sha256 -binary | openssl enc -base64
        -->
        <pin-set expiration="2099-01-01">
            <pin digest="SHA-256">XCXZ8ud+eneT+di9eIMfC7Z2x+p2cr2KJ3e7TnIEOx4=</pin>
        </pin-set>
    </domain-config>
```

通过 SSL PINNING 绑定证书以后，设置系统代理，即使将抓包工具的证书导入到系统目录，抓包时依然会发现 App 报错 _javax.net.ssl.SSLPeerUnverifiedException: Certificate pinning failure!_ 或者 _javax.net.ssl.SSLHandshakeException: java.security.cert.CertPathValidatorException: Trust anchor for certification path not found._，报错的原因是 APP 预置的证书信息与服务端返回的证书信息校验不一致，导致握手失败。  
绕过 SSL PINNING 会稍微麻烦一点，需要使用到第三方插件，比如老牌的 JustTrustMe 模块，其绕过原理是通过 Hook 各网络框架的证书验证方法，替换其方法原有的逻辑，使校验失效。安装插件以后，设置代理，抓包效果如下:

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/4.png)

需要说明的是，安装三方插件以后，即使不安装代理工具的证书到系统信任库，依然可以抓到使用系统证书校验的数据报文。

### 4. 双向校验

无论是系统证书链校验还是 SSL PINNING，其本质都是客户端在校验服务端的证书，即单向校验。如果先保存一个证书在 APP 客户端中，HTTPS 协议握手时客户端把 APP 中保存的证书发送给服务端，这种客户端校验服务端证书，同时服务端也校验客户端证书的方式称为双向校验。以 OkHttp 框架为例，示列代码如下:

```
/*
        * 双向校验
        * 因该测试是自建服务器并自签名，所以需要先在res/xml/network_security_config中配置信任服务端证书
         */
        mTwoVerificationButton = findViewById(R.id.twoVerification);
        mTwoVerificationButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                new Thread(new Runnable(){
                    @RequiresApi(api = Build.VERSION_CODES.KITKAT)
                    @Override
                    public void run() {
                        X509TrustManager trustManager = null;
                        try {
                            TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());

                            trustManagerFactory.init((KeyStore) null);
                            TrustManager[] trustManagers = trustManagerFactory.getTrustManagers();
                            if (trustManagers.length != 1 || !(trustManagers[0] instanceof X509TrustManager)) {
                                throw new IllegalStateException("Unexpected default trust managers:" + Arrays.toString(trustManagers));
                            }
                            trustManager = (X509TrustManager) trustManagers[0];
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                        OkHttpClient mClient = client.newBuilder().sslSocketFactory(ClientSSLSocketFactory.getSocketFactory(MainActivity.this),trustManager).hostnameVerifier(new HostnameVerifier() {
                            @Override
                            public boolean verify(String hostname, SSLSession session) {
                                HostnameVerifier hv = HttpsURLConnection.getDefaultHostnameVerifier();
                                return hv.verify("www.test.com", session);
                            }
                        }).build();

                        Request request = new Request.Builder()
                                .url("https://www.test.com/?q=doubleVerif")
                                .build();
                        Message message = new Message();
                        try (Response response = mClient.newCall(request).execute()) {
                            message.what = 4432005;
                            Log.d("TestReq", response.body().string());
                            message.obj = "请求成功: " + response.code();
                            mHandler.sendMessage(message);
                        } catch (IOException e) {
                            message.what = 4434005;
                            message.obj = e.getLocalizedMessage();
                            mHandler.sendMessage(message);
                            e.printStackTrace();
                        }
                    }
                }).start();
            }
        });
```

示列中校验的是 test.com 域名，可以通过绑定 hosts 的方式在本地进行测试。ClientSSLSocketFactory 对象实现如下:

```
public class ClientSSLSocketFactory {
    private static final String KEY_STORE_PASSWORD = "Curz0n";//证书密码
    private static InputStream client_input;

    public static SSLSocketFactory getSocketFactory(Context context) {
        try {
            //客户端证书
            client_input = context.getResources().getAssets().open("client.p12");
            SSLContext sslContext = SSLContext.getInstance("TLS");
            KeyStore keyStore = KeyStore.getInstance("PKCS12");
            keyStore.load(client_input, KEY_STORE_PASSWORD.toCharArray());
            KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
            keyManagerFactory.init(keyStore, KEY_STORE_PASSWORD.toCharArray());
            sslContext.init(keyManagerFactory.getKeyManagers(), null, new SecureRandom());
            SSLSocketFactory factory = sslContext.getSocketFactory();
            return factory;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                client_input.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return null;
    }
}
```

服务端使用 Python 实现的，代码如下所示:

```
import os
import sys
import ssl
from http.server import HTTPServer, BaseHTTPRequestHandler

#服务端证书和私钥
serverCerts = "%s/certs/server/server-cert.cer" % os.getcwd()
serverKey = "%s/certs/server/server-key.key" % os.getcwd()
#客户端证书
clientCerts = "%s/certs/client/client-cert.cer" % os.getcwd()

class RequestHandler(BaseHTTPRequestHandler):
    def _writeheaders(self):
        self.send_response(200)
        self.send_header('Content-type','text/plain')
        self.end_headers()
    def do_GET(self):
        self._writeheaders()
        self.wfile.write("OK".encode("utf-8"))

def main():
    if (len(sys.argv) != 2):
        port = 443
    else:
        port = sys.argv[1]
    server_address = ("0.0.0.0", int(port))
    server = HTTPServer(server_address, RequestHandler)
    #双向校验
    server.socket = ssl.wrap_socket(server.socket, certfile = serverCerts, server_side = True,  
                               keyfile = serverKey,
                               cert_reqs = ssl.CERT_REQUIRED,
                               ca_certs = clientCerts
                               )
    print("Starting server, listen at: %s:%s" % server_address)
    server.serve_forever()

if __name__ == "__main__":
    main()
```

设置系统代理，访问目标抓取报文，发现代理工具 burp 报错`certificate_required`

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/5.png)

我们知道 burp 能抓包 HTTPS 协议报文的原理是: burp 在中间即充当客户端，又充当服务端。在 burp 冒充客户端时，因为服务端要求 APP 发送客户端证书进行验证，而 burp 没有客户端证书，所以导致握手失败。知道报错的原因，那解决问题就简单了，我们可以通过逆向 APP 获取到客户端证书和密码，然后将客户端证书导入 burp 中，这样 burp 又可以正常冒充客户端身份证了，如下所示，把客户端证书导入到 burp 中:

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/6.png)

抓包效果如下:

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/7.png)

### 5. WebView 证书校验

在使用 WebView 进行 HTTPS 请求时，同样可以对 HTTPS 证书进行校验，WebView 控件对请求逻辑进行了封装，所以进行证书校验更加方便，具体代码如下所示，handler.cancel() 相当于使用系统证书校验:

```
mWebview.setWebViewClient(new WebViewClient(){
            @Override
            public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
                //忽略证书校验
                if("trustAllCerts".equals(tag)){
                    handler.proceed();

                }else {
                    handler.cancel();
                }
            }
        });
```

如果想 SSL PINING，同样可以通过`res/xml/network_security_config.xml`配置文件对服务端证书进行校验。对抗证书校验的方法和 OkHttp 网络框架一样，具体参考上文，除了双向校验以外，抓包截图中都有 WebView 的请求报文。

### 6. 代理检测

除了校验 HTTPS 证书防止中间人抓包以外，常见的方法还有通过检测代理防止抓包，其原理是检测到设备开启系统代理后，APP 中通过代码实现禁用代理，以 OkHttp 框架为例，示列代码如下:

```
/*
        * 检测代理
        * 直接绕过代理，网络正常但抓包工具无法抓包
         */
        mCheckProxy = findViewById(R.id.checkProxy);
        mCheckProxy.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
                if (isChecked){
                    client = new OkHttpClient().newBuilder().proxy(Proxy.NO_PROXY).build();
                }else{
                    client = new OkHttpClient();
                }
            }
        });
```

对抗方法也比较简单，可以使用 iptables 对请求进行强制转发，ProxyDroid 全局代理工具就是通过 iptables 实现的，所以使用 ProxyDroid 开启代理，可以比较有效的绕过代理检测。

0x04 结语
-------

文章对 HTTPS 证书常见的校验及防抓包方法做了一个简单的总结，针对每种场景提供了具体的代码示列及对应的抓包方法。笔者水平有限，文章如有理解错误的地方，还请不吝赐教。

**References:**  
[Android+Nginx 一步步配置 https 单向 / 双向认证请求](https://blog.csdn.net/jjzhoujun2010/article/details/101442325)  
[为 Python Web Server 增加 HTTPs 双向验证](https://zeroyi.com/wei-python-web-serverzeng-jia-httpsshuang-xiang-yan-zheng/)  
[Network security configuration](https://developer.android.com/training/articles/security-config)

**版权声明：转载请注明出处，谢谢。[https://github.com/curz0n](https://github.com/curz0n)**