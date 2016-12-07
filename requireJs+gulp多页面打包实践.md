### requireJs+gulp多页面打包实践
[requirejs单公有包多页面包方案项目demo](https://github.com/iscarecrow/require-mutientry-hugin-demo)


#### 一. requireJS构建项目分成两部分
- 开发环境 requireJs请求对应的依赖加载
- 正式环境 almond替换require打包

#### 二. 开发及生产依赖方式。
采用一个base包依赖，每个页面都有自己的page.js

#### 三. 依赖打包流程如下

1.代码依赖方式如下

##### 入口 main-a.js (有很多入口)
关键点是这个js，先依赖main的配置，再依赖base,最后引入page/a.js。

```
requirejs([ "main" ], function () {
  requirejs([ "base"], function () {
    requirejs([ "page/a"], function () {

    });
  });
});
```

##### 公共级别 base.js（1个）
```
define([
    "jquery"
], function($) {
  console.log($);
  $('.a_color').css({'color':'red'});
});
```

##### 页面级别 a.js（有很多page级别）
```
define(['jquery'], function($) {
  console.log('a page');
  $('.a_size').css({'font-size':'40px'});
});
```

##### 配置main.js（第三方组件依赖都靠这个）
```

requirejs.config({
  baseUrl: '../src/demo1/js',
  // 开发环境加时间戳，清理缓存
  urlArgs: "bust=" + (new Date()).getTime(),
  paths: {
    almond:'../../../node_modules/almond/almond',
    jquery: "../../../node_modules/jquery/dist/jquery.min"
  },
  shim : {
    jquery: {
      exports: '$'
    }
  }
});
```

2.使用rjs，gulp任务如下：
- `mainConfigFile`写入模块的依赖路径，包括jquery等第三方组件
- `modules`include表示需要引入，exclude表示排除。所以page级别的包。全部排除base。

```
gulp.task('jsmin', function(cb) {
  rjs.optimize({
    mainConfigFile: "src/demo1/js/main.js",
    baseUrl: 'src/demo1/js',
    removeCombined: false,
    findNestedDependencies: false,
    optimize: "none",
    optimizeCss: "none",
    preserveLicenseComments: "false",
    paths: {
      machina: "empty:"
    },
    dir: './static/demo1/js',
    generateSourceMaps: false,
    modules: [{
      "name": "base",
      "include":["almond"]
    },{
      "name":"main-a",
      "include": ["page/a"],
      "exclude": ["base"] 
    },{
      "name":"main-b",
      "include": ["page/b"],
      "exclude": ["base"]
    },]
  }, function(buildResponse) {
    // console.log('build response', buildResponse);
    cb();
  }, cb);
});
```

3.开发环境和生产环境

开发环境
```
<script data-main="../src/demo1/js/main-a.js" src="../node_modules/requirejs/require.js"></script>
```

请求的内容
- require.js （amd加载）
- main-a.js
- main.js
- base.js
- jquery
- a.js

正式环境
此时用almond替换了requirejs,目的是为了减少包的大小。同时不存在异步加载需求

```
<script type="text/javascript" src="../static/demo1/js/base.js"></script>
<script type="text/javascript" src="../static/demo1/js/main-a.js"></script>

base.js
----------------
almond.js
jquery.js
base.js

main-a.js
----------------
main.js
main-a.js
page/a.js
```
请求内容
- base.js
- main-a.js



#### 结论
目的是获取单个公有包，各个页面有自己的js包。多个main入口是此方法的关键。存在的问题是rjs打包速度非常慢！！如果页面不断增加，后期很难打包。
如果做单页应用,backbone+requirejs+jquery的方案，就变成了1个base1个app。