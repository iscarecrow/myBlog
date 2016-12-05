### angular大型项目架构的目录结构

[angular项目结构demo](https://github.com/iscarecrow/angular-hugin-demo )

一般来说项目目录结构组织有两种
- by type 文件类型组织
- by feature 业务类型组织


#### by type
```
+-- controller
|   +-- aController.js
|   +-- bController.js
+-- service
|   +-- aService.js
|   +-- bService.js
+-- directive
|   +-- xxDirective.js
|   +-- yyDirective.js
+-- filter
|   +-- xxFilter.js
|   +-- yyFilter.js
+-- models
|   +-- aModels.js
|   +-- bModels.js
+-- template
|   +-- a.html
|   +-- b.html
+-- route.js
+-- app.js
+-- index.html
```

by type类型组织业务，
1.最麻烦的一个问题是，随着业务不断变得庞大。寻找和维护文件的成本变的非常高。比如在几百个controller了中找到需要的controller，再去寻找filter,service都需要非常高的成本。
2.同时还有一个问题，在于打包过程是否全部打包到app.js的业务代码中，这个问题也非常的麻烦。


#### by feature
``` 
+-- common
|   +-- service(处理共有逻辑)
|   +-- lib
|   +-- utils
|   +-- directive
+-- home
|   +-- app.js
|   +-- homeService.js
|   +-- homeAController.js
|   +-- homeBController.js
|   +-- home_a.html
|   +-- home_b.html
+-- login
|   +-- app.js
|   +-- loginService.js
|   +-- loginAController.js
|   +-- loginBController.js
|   +-- login_a.html
|   +-- login_b.html
+-- user
|   +-- ...
+-- template
|   +-- home.html
|   +-- login.html
|   +-- user.html
```

以业务方式组织代码结构。
1.查找，找到需要开发的业务模块，查找依赖，业务模块功能总会在一定的限度范围内，所以数量相对可控
2.打包，可以对应的业务整个打成对应的包

注意点，把一些抽象的业务，组件放到common

所以此种方法，可以提供大型项目的项目组织结构。详细的代码内容，请参考对应demo
