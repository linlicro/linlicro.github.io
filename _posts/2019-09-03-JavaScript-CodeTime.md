---
layout:     post
title:      听JavaScript播客记录
subtitle:    "\"后端工程师跨界喽\""
date:       2019-09-03
author:     Lin
header-img: img/post-bg-js-version.jpg
catalog: true
tags:
    - javascript
---

*记录自「代码时间」第一期节目「JavaScript – 公子」。*

通过十一个小话题，了解下JS及周边 >>>

**什么是JavaScript, 它和Java的关系**

wiki:

> JavaScript (/ˈdʒɑːvəˌskrɪpt/),[8] often abbreviated as JS, is a high-level, interpreted scripting language that conforms to the ECMAScript specification. JavaScript has curly-bracket syntax, dynamic typing, prototype-based object-orientation, and first-class functions.

JavaScript是ECMAScript的协议在浏览器端的一个实现，也就是说JavaScript是基于ECMAScript协议实现的、在浏览器端的轻量级编程语言。

首先，要了解什么是ECMAscript，ECMAScript是由网景的Brendan Eich开发并提交给Ecma国际的一种脚本语言(JavaScript)的标准化规范。

ECMAscript的实现语言除了JavaScript，还有其他的，比如Adobe的ActionScript、PS里的脚本。

第二点，为什么说是JavaScript是轻量级的，因为它语法简单，不用编译，调试方便，结合大量的其他编程语言的语法，用比较少量的语法实现较多的功能，学习成本也较低。

至于说与Java的关系，这个关系只是当时网景为了推广蹭Java的热度，命名上有点关联，其实这两者是没有半毛钱的关系。

**为什么JavaScript现在如此火热**

TIOBE Index:
![](https://i.loli.net/2019/09/03/JabBARGx6yfcvlL.jpg)

JavaScript语言开始流行的标志事件应该是Google的Gmail的出现，单页实现。

*什么是单页面?*

比如普通的浏览网页，点击一个链接，会跳转到另一个网页，这个点击过程，会刷新整个页面，并重新加载资源。但是单页面的话，点击一个链接，不会发生跳转页面，而是触发ajax请求，在原页面的基础上更新一些内容，而不是重新加载页面，这样不用重新加载重复的资源，访问的流畅度大大提升。

**ECMAScript 5 和 ECMAScript 6 (ECMAScript 2015)**

ECMAScript是协议，JavaScript是协议的实现。ECMAScript 6 是2015年发布的新版本协议，现在的浏览器部分支持这类协议。同时也是因为浏览器没有完全支持ECMAScript 6协议，所有有人开发了Babel插件项目，Babel前身叫「6 to 5」，开发的时候写ECMAScript 6的语法，进过Babel转化 变成ES5的语法，这样浏览器就能兼容的展示。

**Babel 工具**

wiki:

> Babel is a free and open-source JavaScript compiler that is mainly used to convert ECMAScript 2015+ (ES6+) code into a backwards compatible version of JavaScript that can be run by older JavaScript engines.

Babel是一个广泛使用的转码器，可以将ES6代码转为ES5代码，从而在现有环境执行，通俗地来讲，就是一个降低软件。

**用ES5还是6来写代码**

推荐使用ES6来进行日常的开发工作，尤其方法有新版本支持时，用新版本写。

**Javascript模块化**

前端开发的复杂度提高，逐渐有团队分工协作、进度管理、单元测试等，Javascript模块化编程成为了不得不使用的一种软件工程。

早期的JS开发是使用script标签，随着项目越来越大越复杂，不可能一直加script标签、加script标签、加script标签...一方面性能上有影响，另一方面工程/项目管理非常麻烦，所以要使用模块化编程。通过模块加载来引用三方模块的加载、管理。

也就是说，工程项目越来越大，用到的三方插件越来越多，你很难确认引用了插件A时，它的依赖插件B是不是已经加载进来了，作为开发人员，手动控制这些很难。

**webpack, gulp, grunt 工具**

gulp，grunt本质上是任务管理的工具， 而webpack属于模块打包工具。

通过gulp建立一些任务，当我们对相应文件进行修改之后，会自动触发这些任务，自动的对文件进行了处理。不用我们手动的、反复的去处理这些文件，大大提高了我们的工作效率。

Grunt 可以帮助我们减少很多的工作量，比如：检查每个 JS 文件语法、合并两个 JS 文件、将合并后的 JS 文件压缩、将 SCSS 文件编译等，包括我们上面提到的试用gulp将.less文件转换为.css文件。

webpack 本质上是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

使用这类工具，可以让你的项目代码越来越方便管理，越来越方便多人协作，线上的代码越来越符合浏览器快速加载、易于交互。

**NodeJS 和 NPM**

NodeJS的出现是JavaScript的一个标识里程碑。

Node.js是一个基于Chrome V8引擎的JavaScript运行环境，简单的说 Node.js 就是运行在服务端的 JavaScript。Chrome V8引擎的出现 让JavaScript的解释变得非常快速、高效，NodeJS就是把这个引擎包进来了，同时NodeJS不仅仅可以以Chrome V8为基本架构，同时微软发布的Chakra引擎也可以嵌入到NodeJS。

npm全称为Node Package Manager，就是包管理。npm大大解决了NodeJS包依赖的管理，npm对于包发布、管理、安装都非常便利方便，大大促进了开发者的提交，促进了NodeJS的成功。

简而言之，NodeJS帮助JS前端程序员开始写后端代码，而不仅仅是只专注与浏览器端的开发，所谓现在的JavaScript全栈开发工程师。

**微软开发的TypeScript和它的作用**

wiki:

> TypeScript is an open-source programming language developed and maintained by Microsoft. It is a strict syntactical superset of JavaScript, and adds optional static typing to the language. 

我的理解是，TypeScript 较JavaScript更加接近传统的编程语言，包括Java、C语言的开发，有了一个编译的过程，在编译期间就能发现很多错误，能够纠错。

**热门的开发框架AngularJS 和 ReactJS**

框架的出现主要是为了方便工程化的项目开发管理，AngularJS 和 ReactJS的主要区别在于数据流上: Angular为双向绑定; React为单向数据流。语法和代码风控上，React 以 JavaScript 为中心，Angular 以 HTML 为中心。React 将 “HTML” 嵌入到 JavaScript 中，Angular将 “JavaScript” 嵌入到 HTML 中。

**如何学习JavaScript**

ES6入门推荐开源书[《ECMAScript 6入门》](https://github.com/ruanyf/es6tutorial)。

练题/练手推荐[codewars练题网站](https://www.codewars.com/)。

另外推荐**awesome-javascript** [JavaScript 资源大全](https://github.com/jobbole/awesome-javascript-cn)，是 sorrycc 发起维护的 JS 资源列表，内容包括：包管理器、加载器、测试框架、运行器、QA、MVC框架和库、模板引擎、数据可视化、时间轴、编辑器等等。


#### 参考

* 代码时间/<http://codetimecn.com/>
* ECMAScript 6入门/<https://github.com/ruanyf/es6tutorial>
* awesome-javascript/<https://github.com/jobbole/awesome-javascript-cn>