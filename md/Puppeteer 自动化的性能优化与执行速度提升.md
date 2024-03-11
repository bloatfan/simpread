> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [github.com](https://github.com/biaochenxuying/blog/issues/69)

![](https://raw.githubusercontent.com/lslz627/PicGo/master/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d346331323931636430306539333434362e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

前言
--

最近随着复杂的自动化任务的增加，robot 项目出现了很多问题，经常要人工智能，在上次清远漂流的时候，就是经常报警，而且基本都是我人工智能解决的，厉害吧 🤩。

![](https://raw.githubusercontent.com/lslz627/PicGo/master/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d646266613637366232316264663561332e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

这些问题包括：**经常卡住，运行慢、卡，浏览器关不掉，CPU 和 内存 经常是满载运行的，特别是 CPU ，经常是 99% 的使用率。**

Chromium 消耗最多的资源是 CPU，一是渲染需要大量计算，二是 Dom 的解析与渲染在不同的进程，进程间切换会给 CPU 造成压力（进程多了之后特别明显）。

其次消耗最多的是内存，Chromium 是以多进程的方式运行，一个页面会生成一个进程，一个进程占用 30M 左右的内存，大致估算 1000 个请求占用 30G 内存，在并发高的时候内存瓶颈最先显现。

优化最终会落在内存和 CPU 上（所有软件的优化最终都要落到这里），通常来说因为并发造成的瓶颈需要优化内存，计算速度慢的问题要优化 CPU。

所以这篇文章，我们谈谈如何优化 Puppeteer 的性能优化与执行速度。

> `Headless Chrome` ，无头模式，浏览器的无界面形态，可以在不打开浏览器的前提下，在命令行中运行测试脚本，能够完全像真实浏览器一样完成用户所有操作，不用担心运行测试脚本时浏览器受到外界的干扰，也不需要借助任何显示设备，使自动化测试更稳定。

优化点
---

### 优化 Chromium 启动项

1.  如果将 Dom 解析和渲染放到同一进程，肯定能提升时间（进程上下文切换的时间）。对应的配置是 `single-process`
2.  部分功能 disable 掉，比如 GPU、Sandbox、插件等，减少内存的使用和相关计算。

代码如下：

```
const browser = await puppeteer.launch(
{
	headless:true,
	args: [
  ‘–disable-gpu’, // GPU硬件加速
  ‘–disable-dev-shm-usage’, // 创建临时文件共享内存
  ‘–disable-setuid-sandbox’, // uid沙盒
  ‘–no-first-run’, // 没有设置首页。在启动的时候，就会打开一个空白页面。
  ‘–no-sandbox’, // 沙盒模式
  ‘–no-zygote’,
  ‘–single-process’ // 单进程运行
	]
});
```

### 优化 Chromium 执行流程

接下来我们再单独优化 Chromium 对应的页面。

每次请求都启动 Chromium，再打开 tab 页，请求结束后再关闭 tab 页与浏览器。

流程大致如下：

**请求到达 -> 启动 Chromium -> 打开 tab 页 -> 运行代码 -> 关闭 tab 页 -> 关闭 Chromium -> 返回数据**

真正运行代码的只是 tab 页面，理论上启动一个 Chromium 程序能运行成千上万的 tab 页，可不可以复用 Chromium 只打开一个 tab 页然后关闭呢？

当然是可以的，Puppeteer 提供了 puppeteer.connect() 方法，可以连接到当前打开的浏览器。

流程如下：

**请求到达 -> 连接 Chromium -> 打开 tab 页 -> 运行代码 -> 关闭 tab 页 -> 返回数据**

代码如下：

```
const MAX_WSE = 4;  //启动几个浏览器 
let WSE_LIST = []; //存储browserWSEndpoint列表
init();
app.get('/', function (req, res) {
    let tmp = Math.floor(Math.random()* MAX_WSE);
    (async () => {
        let browserWSEndpoint = WSE_LIST[tmp];
        const browser = await puppeteer.connect({browserWSEndpoint});
        const page = await browser.newPage();
        await page.goto('file://code/screen/index.html');
        await page.setViewport({
            width: 600,
            height: 400
        });     
        await page.screenshot({path: 'example.png'});
        await page.close();
        res.send('Hello World!');
    })();
});
function init(){
    (async () => {
        for(var i=0;i<MAX_WSE;i++){
            const browser = await puppeteer.launch({headless:true,
                args: [
                '--disable-gpu',
                '--disable-dev-shm-usage',
                '--disable-setuid-sandbox',
                '--no-first-run',
                '--no-sandbox',
                '--no-zygote',
                '--single-process'
            ]});
            browserWSEndpoint = await browser.wsEndpoint();
            WSE_LIST[i] = browserWSEndpoint;
        }
        console.log(WSE_LIST);
    })();   
}
```

程序启动时（使用 Express 提供 Web 接口），初始化一定数量的无头浏览器，并保存 `WSEndpoint` 列表，当收到请求时，通过随机数做简单的负载均衡（利用多核特性）。

使用 tab 方式渲染后请求速度提升了 200ms 左右，一个 tab 进程使用内存降到 20M 以内，带来的收益也非常可观。

不过这里要注意，官方并不建议这样做，**因为一个 tab 页阻塞或者内存泄露会导致整个浏览器阻塞并 Crash**。万全的解决办法是定期重启程序，**当请求 1000 次或者内存超过限制后重启对应的进程**。

其实这个方法并不适用于我们的 robot 项目，因为 代理、浏览器指纹 等信息，很难在一个浏览器里面做到完全隔离，如果要隔离，要写很多的代码来删除缓存、配置等 来区分环境才行。

之所以讲出来，如果后面有项目是专门做爬虫来采集数据、信息的，可能可以用得上。

### 页面优化

浏览器打开的页面数量越多，占用的内存就越多，和我们平时使用浏览器是一样的原理的。

但是 robot 项目里面有几个任务是打开多个 标签页面 来做任务的，比如 绑定货币、检查组合。

1.  tab 页多必然会卡，所以必须有效控制 tab 页个数。
    
2.  浏览器打开时会默认有一个 page 页面，直接利用该页面能减少 1/3 左右的内存消耗。
    
3.  如果要打开多个页面来执行任务时，打开的页面执行完任务之后，最好把其关闭，减少内存的占用。
    

### 场景及数据分析

因为 FB 变化多端，有很多的场景，而且有时候场景的元素重叠了，导致程序跑错流程，所以代码写了很多种场景的判断，而且有时候看到都觉得无语。

![](https://raw.githubusercontent.com/lslz627/PicGo/master/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d353465353261313166326133303136662e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

而且有些场景都不用了，我们也不知道，旧场景的代码还留在程序里面了。

所以我觉得要做个**场景的统计才行，每过一两个月看下数据，把不用的场景的代码删掉**。

刚好我们上传日志的 **Kibana** 也即是 elk 那个平台就有这个功能，可以搞很多的报表分析，代码也不用修改，只分析一下那个日志就行。

还可以 **分析 FB 的新版与旧版的灰度量**，决定处理异常的优先度。

以此类推，其他项目结合具体的场景，应该也可以采用这个方法，比如 web 项目有些场景的日志。

Kibana 功能其实很强大的，之前都不知道，往后还是要学习一下这个产品才行。

![](https://raw.githubusercontent.com/lslz627/PicGo/master/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d353530346561326332396466663737622e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

### 植入 javascript 代码

iframe 较多时，浏览器经常卡到无法运行，所以可以考虑在代码里加了删除无用 iframe 的脚本。

不过，这各情况，在 robot 项目里面遇到的不多。

```
(async () => {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    await page.goto('https://webmail.vip.188.com');
    //注册一个 Node.js 函数，在浏览器里运行
    await page.exposeFunction('md5', text =>
        crypto.createHash('md5').update(text).digest('hex')
    );
    //通过 page.evaluate 在浏览器里执行删除无用的 iframe 代码
    await page.evaluate(async () =>  {
        let iframes = document.getElementsByTagName('iframe');
        for(let i = 3; i <  iframes.length - 1; i++){
            let iframe = iframes[i];
            if(iframe.name.includes("frameBody")){
                iframe.src = 'about:blank';
                try{
                    iframe.contentWindow.document.write('');
                    iframe.contentWindow.document.clear();
                }catch(e){}
                //把iframe从页面移除
                iframe.parentNode.removeChild(iframe);
            }
        }
        //在页面中调用 Node.js 环境中的函数
        const myHash = await window.md5('PUPPETEER');
        console.log(`md5 of ${myString} is ${myHash}`);
    });
    await page.close();
    await browser.close();
})();
```

### 优化静态文件加载

我们在爬取网站的时候, 一般比较关心网站的加载速度, 而限制加载速度的大多数是静态文件, 包括 css, font, image。

为了优化爬虫性能, 我们需要阻止浏览器加载这些不必要的文件, 这可以通过对请求进行拦截来实现。

而且做到 **随机拦截** 更好一点。

```
await page.setRequestInterception(true);
page.on('request',  req => {
  if(['image', 'stylesheet', 'font'].includes(req.resourceType())) {
    return request.abort();
  }
  return request.continue();
});
```

### 开发调试

*   puppeteer.launch(options)

devtools: true // 是否为每个选项卡自动打开 DevTools 面板，这个选项只有当 headless 设置为 false 的时候有效 *

开发时，可以通过 **环境变量** 来设置自动打开控制台，不用每次手动打开，减少操作时间。

_![](https://raw.githubusercontent.com/lslz627/PicGo/master/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d613065646465366331613362616165392e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)_

### 开启缓存

```
--user-data-dir=${this.userDataDir}
```

设置用户数据目录，用户数据目录（User Data Directory）是 Chrome / Chromium 用来存放户插件、书签和 cookie 等信息的文件夹。

现在已经开发了这个功能了的，线上任务机都还没用上，只有开发任务机和本地开发上用到而已。

不过开启这个功能会耗费磁盘内存，要加个功能：**缓存达到 80% 左右，就自动删除本地的缓存**。

### 配置优化

现在线上的任务机已经有 32 台了，而且任务机会越来越多。

如果某天要加一个环境变量什么的，我就要手动修改 32 次，如果增加到 100 台任务机，就更恐怖了。

所以觉得有必要把一些配置放在 admin 里面来配置，并且统一管理。

觉得现在有必要加到 admin 配置有：

1.  所有的环境变量：由统一的一个文件或者接口管理。
    
2.  进程数量的配置也由接口控制。
    

还有一点就是：现在 robot 发版要 8 分钟左右了，之前是 2 分钟左右就能发完的，所以任务机的维护也要重视了。

### 浏览器关不掉

最近关不掉的浏览器都是这个情况的：

![](https://raw.githubusercontent.com/lslz627/PicGo/master/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d613136623565353738623664393039322e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

原因：911 代理的 ip 相同，用的端口不同，就会出现 **This site can’t be reached** 没网络，还扣钱。

解决方法：用新的代理方案出来之后，应该就不会出现了。或者定时调用脚本重启 robot 程序 (执行任务超过 1000 条，或者没有执行任务的时候)。

### 911 没代理

获取 911 代理的余额、没有代理时，暂停拉取任务，15 分钟检查一次，还是没有代理就进行报警。

### 平滑重启 robot

robot 启动超过 24 小时，且再没有拉取到任务时，robot 调用脚本来重启 robot。

之前想通过 定时检查 CPU 和 内存，过高的次数超过一定值或者 robot 开启的状态超过一定时间，就重启 robot 的，但是发现 CPU 和 内存经常是很高的，所以不太可行。

想要优化的点
------

### 场景的重现

robot 最耗时的就是场景的重现，往往都是要找到特定的号，去到特定的页面位置，才能补好场景的。

之前想过，robot 出现未知错误时，就保存 html、js、css 等文件，特定的元素是保留下来了，但是因为特定的账号没有登录，一打开 html 文件时，是重现不了特定的场景的，补不了场景。

这个暂时没解决方法。

最后
--

之所以分享这个内容，最近人工智能的操作真的有点多，哈哈哈。

所以想集思广益，得到更多、更好的优化方法，提升 robot 项目的开发与维护工作。

所以大佬们，如果有更好的建议，请提出来，哈哈哈 😉。

[![](https://raw.githubusercontent.com/lslz627/PicGo/master/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d663165663439663361616166623737622e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/3713f51c46ef939c309ac94d5586a1cdeefcabe180f1dcadec5ce3f25c21a89b/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d663165663439663361616166623737622e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

 ![](https://raw.githubusercontent.com/lslz627/PicGo/master/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d656132366263373761613563653932322e6769663f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970) [ ![](https://raw.githubusercontent.com/lslz627/PicGo/master/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d656132366263373761613563653932322e6769663f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970) ](https://camo.githubusercontent.com/fcf804d653752581bb6c6c984f4a0d5da6326c7f0595ae3ce68e10dcc2fe24cf/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d656132366263373761613563653932322e6769663f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970)   [](https://camo.githubusercontent.com/fcf804d653752581bb6c6c984f4a0d5da6326c7f0595ae3ce68e10dcc2fe24cf/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d656132366263373761613563653932322e6769663f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970) 

**参考文章与好文章**

*   [Puppeteer 性能优化与执行速度提升](https://blog.it2048.cn/article-puppeteer-speed-up/)
*   [无头浏览器性能对比与 Puppeteer 的优化文档](https://blog.it2048.cn/article-headless-puppeteer/)
*   [playwright](https://github.com/microsoft/playwright)
*   [跨平台的浏览器自动化工具 Playwright 简析](https://yrq110.me/post/front-end/dive-into-playwright/)
*   [使用 generic-pool 优化 puppeteer 并发问题](https://blog.guowenfh.com/2019/06/16/2019/puppeteer-pool/)
*   [结合项目来谈谈 Puppeteer](https://juejin.im/post/5d4059305188255d38489a8c)
*   [Puppeteer 爬虫性能优化](https://github.com/nfwyst/Blog/issues/14)
*   [京喜前端自动化测试之路](https://aotu.io/notes/2020/05/06/jingxi-automated-testing/index.html)
*   [利用 cluster 优化 Puppeteer](https://www.yuque.com/luqixiuzichiji/nodejs/ces)
*   [可爱的 Puppeteer 使用小技巧](https://yrq110.me/post/front-end/some-tips-of-using-puppetter/)
*   [puppeteer 优化小技巧](https://juejin.im/post/5db97541e51d4529de39f72d)
*   [使用 Puppeteer 搭建统一海报渲染服务](https://www.infoq.cn/article/dcSBL_9AzCwVPsaQ70dh)

 

和人工智能有啥关系...

 

您用什么代理，有什么稳定又好用的代理推荐吗

 

Nothing to preview