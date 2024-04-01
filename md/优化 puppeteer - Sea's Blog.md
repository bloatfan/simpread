> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [hailangya.com](https://hailangya.com/articles/2021/07/05/optimize-puppeteer/)

> 本文将讲述如何优化 puppeteer

```
import puppeteer from 'puppeteer';
import { htmlGenWaterMark } from './watermark';

class PuppeteerHelper {
  constructor() {
    this.instance = null;
    
    this.MAX_WSE = 4;
    
    this.WSE_LIST = [];
    
    this.PAGE_COUNT = 1000;
    
    this.PAGE_NUM = [];
    
    this.replaceTimer = [];
    
    this.p_config = {
      headless: true, 
      args: [
        '--disable-gpu', 
        '--disable-dev-shm-usage', 
        '--disable-setuid-sandbox', 
        '--no-first-run', 
        '--no-sandbox', 
        '--no-zygote',
        '--single-process', 
      ],
    };

    
    this._init();
  }

  static getInstance() {
    if (!this.instance) {
      this.instance = new PuppeteerHelper();
    }
    return this.instance;
  }

  
  





  _init() {
    (async () => {
      console.log('【PuppeteerHelper】puppeteer config:', this.p_config);
      for (let i = 0; i < this.MAX_WSE; i++) {
        await this._generateBrowser(i);
      }
      console.log('【PuppeteerHelper】WSE_LIST：', this.WSE_LIST);
    })();
  }

  



  async _generateBrowser(num) {
    
    const browser = await puppeteer.launch(this.p_config);
    
    this.WSE_LIST[num] = await browser.wsEndpoint();
    
    this.PAGE_NUM[num] = 1;

    return browser;
  }

  





  async _replaceBrowserInstance(browser, num, retries = 2) {
    clearTimeout(this.replaceTimer[num]);

    const pageNum = this.PAGE_NUM[num];

    
    const openPages = await browser.pages();
    const oneMinute = 60 * 1000;
    
    if (openPages && openPages.length > 1 && retries > 0) {
      const nextRetries = retries - 1;
      console.log(
        '【PuppeteerHelper】当前使用浏览器编号：%s，browser.pages：%s，retries',
        num,
        openPages.length,
        retries
      );
      this.replaceTimer[num] = setTimeout(() => this._replaceBrowserInstance(browser, num, nextRetries), oneMinute);
      
      return browser;
    }

    
    browser.close();

    
    const newBrowser = await this._generateBrowser(num);
    console.log(
      '【PuppeteerHelper】当前使用浏览器编号：%s 已打开页面总次数（%s）超过上限，创建新实例，新的wsEndpoint：%s',
      num,
      pageNum,
      this.WSE_LIST[num]
    );

    return newBrowser;
  }

  


  async _currentBrowser() {
    
    const tmp = Math.floor(Math.random() * this.MAX_WSE);
    const browserWSEndpoint = this.WSE_LIST[tmp];
    const pageNum = this.PAGE_NUM[tmp];
    console.log(
      '【PuppeteerHelper】当前使用浏览器编号：%s ，wsEndpoint：%s，过去已打开页面总次数 %s',
      tmp,
      browserWSEndpoint,
      pageNum
    );

    let browser;
    try {
      
      browser = await puppeteer.connect({ browserWSEndpoint });

      
      if (this.PAGE_NUM[tmp] > this.PAGE_COUNT) {
        browser = this._replaceBrowserInstance(browser, tmp);
      }
    } catch (err) {
      
      browser = await this._generateBrowser(tmp);
      console.log(
        '【PuppeteerHelper】当前使用浏览器编号：%s 连接失败，创建新实例，新的wsEndpoint：%s',
        tmp,
        this.WSE_LIST[tmp]
      );
      console.log('【PuppeteerHelper】WSE_LIST：', this.WSE_LIST);
    }

    
    this.PAGE_NUM[tmp]++;

    return browser;
  }

  




  async _waitRender(page, timeout) {
    console.log('【_waitRender】开启自定义等待,自定义等待时长：%s ms,（默认30s）', timeout);
    
    
    const renderDoneHandle = await page.waitForFunction('window._renderDone', {
      polling: 120,
      timeout: timeout,
    });

    const renderDone = await renderDoneHandle.jsonValue();
    if (typeof renderDone === 'object') {
      console.log(`【_waitRender】加载页面失败： -- ${renderDone.msg}`);
      await page.close();

      throw new Error(`客户端请求重试： -- ${renderDone.msg}`);
    } else {
      console.log('【_waitRender】页面加载成功');
    }
  }

  






  async _watermark(page, text) {
    
    await page.addScriptTag({ content: htmlGenWaterMark.toString() });
    await page.evaluate(
      (options) => {
        window.htmlGenWaterMark(options);
      },
      { text: text }
    );
  }

  














  async screenshot({
    url,
    filePath = './example.png',
    width = 800,
    height = 600,
    screenshotType = 'default',
    selector,
    altitudeCompensation = 0,
    headers,
    openWait,
    waitTimeout,
    openWatermark,
    watermarkText = '水印',
  }) {
    console.log('【PuppeteerHelper】开始截图');
    
    const browser = await this._currentBrowser();
    
    const page = await browser.newPage();
    try {
      
      await page.setViewport({ width, height });
      
      headers && (await page.setExtraHTTPHeaders(headers));
      
      await page.goto(url, {
        
        waitUtil: 'networkidle0',
      });
      
      
      const documentSize = await page.evaluate(() => {
        return {
          width: document.documentElement.clientWidth,
          
          height: document.body.clientHeight,
        };
      });
      
      openWait && (await this._waitRender(page, waitTimeout));
      
      openWatermark && (await this._watermark(page, watermarkText));

      
      
      

      
      const picture = await this._capture(page, { screenshotType, filePath, selector, altitudeCompensation });

      return picture;
    } finally {
      console.log('【PuppeteerHelper】结束截图，关闭当前页面');
      
      await page.close();
    }
  }

  









  async _capture(page, { screenshotType, filePath, selector, altitudeCompensation = 0 }) {
    switch (screenshotType) {
      case 'selector': {
        const element = await page.$(selector);
        const boundingBox = await element.boundingBox();
        const picture = await element.screenshot({ path: filePath, clip: boundingBox });
        return picture;
      }
      case 'scrollBody': {
        
        const { scrollHeight, isBody, width } = await page.evaluate(() => {
          const clientHeight = document.documentElement.clientHeight;
          const clientWidth = document.documentElement.clientWidth;
          const divs = [...document.querySelectorAll('div')];
          const len = divs.length;
          let isBody = false;
          let boxEl = null;
          let i = 0;
          for (; i < len; i++) {
            const div = divs[i];
            if (div.scrollHeight > clientHeight) {
              boxEl = div;
              break;
            }
          }
          if (!boxEl && i === len) {
            boxEl = document.querySelector('body');
            isBody = true;
          }
          return { scrollHeight: boxEl.scrollHeight, isBody: isBody, width: clientWidth };
        });
        
        !isBody && (await page.setViewport({ height: scrollHeight + altitudeCompensation, width: width }));
        
        const picture = await page.screenshot({
          path: filePath,
          fullPage: true,
        });
        return picture;
      }
      case 'default':
      default: {
        const picture = await page.screenshot({
          
          path: filePath,
          fullPage: true,
          
          
          
          
          
          
        });
        return picture;
      }
    }
  }
}

export default PuppeteerHelper.getInstance();
```