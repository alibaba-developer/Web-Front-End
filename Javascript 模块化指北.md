# Javascript 模块化指北.md

<b>前言</b>

随着 Web 技术的蓬勃发展和依赖的基础设施日益完善，前端领域逐渐从浏览器扩展至服务端（Node.js），桌面端（PC、Android、iOS），乃至于物联网设备（IoT），其中 JavaScript 承载着这些应用程序的核心部分，随着其规模化和复杂度的成倍增长，其软件工程体系也随之建立起来（协同开发、单元测试、需求和缺陷管理等），模块化编程的需求日益迫切。

JavaScript 对模块化编程的支持尚未形成规范，难以堪此重任；一时间，江湖侠士挺身而出，一路披荆斩棘，从刀耕火种过渡到面向未来的模块化方案；

<b>概念</b>

模块化编程就是通过组合一些__相对独立可复用的模块__来进行功能的实现，其最核心的两部分是__定义模块__和__引入模块__；

定义模块时，每个模块内部的执行逻辑是不被外部感知的，只是导出（暴露）出部分方法和数据；
引入模块时，同步 / 异步去加载待引入的代码，执行并获取到其暴露的方法和数据；
刀耕火种
尽管 JavaScript 语言层面并未提供模块化的解决方案，但利用其可__面向对象__的语言特性，外加__设计模式__加持，能够实现一些简单的模块化的架构；经典的一个案例是利用单例模式模式去实现模块化，可以对模块进行较好的封装，只暴露部分信息给需要使用模块的地方；

// Define a module
var moduleA = (function ($, doc) {
  var methodA = function() {};
  var dataA = {};
  return {
    methodA: methodA,
    dataA: dataA
  };
})(jQuery, document);

// Use a module
var result = moduleA.mehodA();
直观来看，通过立即执行函数（IIFE）来声明依赖以及导出数据，这与当下的模块化方案并无巨大的差异，可本质上却有千差万别，无法满足的一些重要的特性；

定义模块时，声明的依赖不是强制自动引入的，即在定义该模块之前，必须手动引入依赖的模块代码；
定义模块时，其代码就已经完成执行过程，无法实现按需加载；
跨文件使用模块时，需要将模块挂载到全局变量（window）上；
AMD & CMD 二分天下
题外话：由于年代久远，这两种模块化方案逐渐淡出历史舞台，具体特性不再细聊；

为了解决”刀耕火种”时代存留的需求，AMD 和 CMD 模块化规范问世，解决了在浏览器端的异步模块化编程的需求，__其最核心的原理是通过动态加载 script 和事件监听的方式来异步加载模块；__

AMD 和 CMD 最具代表的两个作品分别对应 require.js 和 sea.js；其主要区别在于依赖声明和依赖加载的时机，其中 require.js 默认在声明时执行， sea.js 推崇懒加载和按需使用；另外值得一提的是，CMD 规范的写法和 CommonJS 极为相近，只需稍作修改，就能在 CommonJS 中使用。参考下面的 Case 更有助于理解；

// AMD
define(['./a','./b'], function (moduleA, moduleB) {
  // 依赖前置
  moduleA.mehodA();
  console.log(moduleB.dataB);
  // 导出数据
  return {};
});
 
// CMD
define(function (requie, exports, module) {
  // 依赖就近
  var moduleA = require('./a');
  moduleA.mehodA();     

  // 按需加载
  if (needModuleB) {
    var moduleB = requie('./b');
    moduleB.methodB();
  }
  // 导出数据
  exports = {};
});
CommonJS
2009 年 ry 发布 Node.js 的第一个版本，CommonJS 作为其中最核心的特性之一，适用于服务端下的场景；历年来的考察和时间的洗礼，以及前端工程化对其的充分支持，CommonJS 被广泛运用于 Node.js 和浏览器；

// Core Module
const cp = require('child_process');
// Npm Module
const axios = require('axios');
// Custom Module
const foo = require('./foo');

module.exports = { axios };
exports.foo = foo;
规范
module (Object): 模块本身
exports (*): 模块的导出部分，即暴露出来的内容
require (Function): 加载模块的函数，获得目标模块的导出值（基础类型为复制，引用类型为浅拷贝），可以加载内置模块、npm 模块和自定义模块
实现
1、模块定义

默认任意 .node .js .json 文件都是符合规范的模块；

2、引入模块

首先从缓存（require.cache）优先读取模块，如果未命中缓存，则进行路径分析，然后按照不同类型的模块处理：

内置模块，直接从内存加载；
外部模块，首先进行文件寻址定位，然后进行编译和执行，最终得到对应的导出值；
其中在编译的过程中，Node对获取的JavaScript文件内容进行了头尾包装，结果如下：

(function (exports, require, module, __filename, __dirname) {
    var circle = require('./circle.js');
    console.log('The area of a circle of radius 4 is ' + circle.area(4));
});
特性总结
同步执行模块声明和引入逻辑，分析一些复杂的依赖引用（如循环依赖）时需注意；
缓存机制，性能更优，同时限制了内存占用；
Module 模块可供改造的灵活度高，可以实现一些定制需求（如热更新、任意文件类型模块支持）；
ES Module（推荐使用）
ES Module 是语言层面的模块化方案，由 ES 2015 提出，其规范与 CommonJS 比之 ，导出的值都可以看成是一个具备多个属性或者方法的对象，可以实现互相兼容；但写法上 ES Module 更简洁，与 Python 接近；

import fs from 'fs';
import color from 'color';
import service, { getArticles } from '../service'; 

export default service;
export const getArticles = getArticles;
主要差异在于：

ES Module 会对静态代码分析，即在代码编译时进行模块的加载，在运行时之前就已经确定了依赖关系（可解决循环引用的问题）；
ES Module 关键字：import export 以及独有的 default 关键字，确定默认的导出值；
ES Module 中导出的值是一个 只读的值的引用 ，无论基础类型和复杂类型，而在 CommonJS 中 require 的是值的拷贝，其中复杂类型是值的浅拷贝；
// a.js
export let a = 1;
export function caculate() {
  a++;
};

// b.js
import { a, caculate } from 'a.js';

console.log(a); // 1
caculate();
console.log(a); // 2

a = 2; // Syntax Error: "a" is read-only
UMD
通过一层自执行函数来兼容各种模块化规范的写法，兼容 AMD / CMD / CommonJS 等模块化规范，贴上代码胜过千言万语，需要特别注意的是 ES Module 由于会对静态代码进行分析，故这种运行时的方案无法使用，此时通过 CommonJS 进行兼容；

(function (global, factory) {
  if (typeof exports === 'object') {   
    module.exports = factory();
  } else if (typeof define === 'function' && define.amd) {
    define(factory);
  } else {
    this.eventUtil = factory();
  }
})(this, function (exports) {
 ​ // Define Module
  Object.defineProperty(exports, "__esModule", {
    value: true
  });
  exports.default = 42;
});
构建工具中的实现
为了在浏览器环境中运行模块化的代码，需要借助一些模块化打包的工具进行打包（ 以 webpack 为例），定义了项目入口之后，会先快速地进行依赖的分析，然后将所有依赖的模块转换成浏览器兼容的对应模块化规范的实现；

模块化的基础
从上面的介绍中，我们已经对其规范和实现有了一定的了解；在浏览器中，要实现 CommonJS 规范，只需要实现 module / exports / require / global 这几个属性，由于浏览器中是无法访问文件系统的，因此 require 过程中的文件定位需要改造为加载对应的 JS 片段（webpack 采用的方式为通过函数传参实现依赖的引入）。具体实现可以参考：tiny-browser-require。

webpack 打包出来的代码快照如下，注意看注释中的时序；

(function (modules) {
  // The module cache
  var installedModules = {};
  // The require function
  function __webpack_require__(moduleId) {}
  return __webpack_require__(0); // ---> 0
})
({
  0: function (module, exports, __webpack_require__) {
    // Define module A
    var moduleB = __webpack_require__(1); // ---> 1
  },
  1: function (module, exports, __webpack_require__) {
    // Define module B
    exports = {}; // ---> 2
  }
});
实际上，ES Module 的处理同 CommonJS 相差无几，只是在定义模块和引入模块时会去处理 __esModule 标识，从而兼容其在语法上的差异。

异步和扩展
1、浏览器环境下，网络资源受到较大的限制，因此打包出来的文件如果体积巨大，对页面性能的损耗极大，因此需要对构建的目标文件进行拆分，同时模块也需要支持动态加载；

webpack 提供了两个方法 require.ensure() 和 import() （推荐使用）进行模块的动态加载，至于其中的原理，跟上面提及的 AMD & CMD 所见略同，import() 执行后返回一个 Promise 对象，其中所做的工作无非也是动态新增 script 标签，然后通过 onload / onerror 事件进一步处理。

2、由于 require 函数是完全自定义的，我们可以在模块化中实现更多的特性，比如通过修改 require.resolve 或 Module._extensions 扩展支持的文件类型，使得 css / .jsx / .vue / 图片等文件也能为模块化所使用；

附录1：特性一览表
模块化规范	加载方式	加载时机	运行环境	备注
AMD	异步	运行时	浏览器	
CMD	异步	运行时	浏览器	依赖基于静态分析，require 时已经 module ready
CommonJS	同步/异步	运行时	浏览器 / Node	
ES Module	同步/异步	编译阶段	浏览器 / Node	通过 import() 实现异步加载
