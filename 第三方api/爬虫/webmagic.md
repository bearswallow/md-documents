# webmagic

github：<https://github.com/code4craft/webmagic>

gitee：<https://gitee.com/flashsword20/webmagic>

文档：<http://webmagic.io/docs/zh/>

## 主要特色

- 完全模块化的设计，强大的可扩展性。
- 核心简单但是涵盖爬虫的全部流程，灵活而强大，也是学习爬虫入门的好材料。
- 提供丰富的抽取页面API。
- 无配置，但是可通过POJO+注解形式实现一个爬虫。
- 支持多线程。
- 支持分布式。
- 支持爬取js动态渲染的页面。
- 无框架依赖，可以灵活的嵌入到项目中去。

## 项目结构

- **webmagic-core**

  webmagic核心部分，只包含爬虫基本模块和基本抽取器。webmagic-core的目标是成为网页爬虫的一个教科书般的实现。

- **webmagic-extension**

  webmagic的扩展模块，提供一些更方便的编写爬虫的工具。包括注解格式定义爬虫、JSON、分布式等支持。

webmagic还包含两个可用的扩展包，因为这两个包都依赖了比较重量级的工具，所以从主要包中抽离出来，这些包需要下载源码后自己编译：：

- **webmagic-saxon**

  webmagic与Saxon结合的模块。Saxon是一个XPath、XSLT的解析工具，webmagic依赖Saxon来进行XPath2.0语法解析支持。

- **webmagic-selenium**

  webmagic与Selenium结合的模块。Selenium是一个模拟浏览器进行页面渲染的工具，webmagic依赖Selenium进行动态页面的抓取。