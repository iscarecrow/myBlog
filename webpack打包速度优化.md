###  webpack打包速度优化

##### 开发环境

最佳方案是采用`webpack-dev-server`, 本地开发调试。

- 开发与正式环境区分
- webpack-dev-server，hmr配置
- api跨域代理

等问题可参考此demo：[webpack-hugin-demo](https://github.com/iscarecrow/webpack-hugin-demo)

##### 正式环境发布或者开发情况下需要打包

- 抽出共用包，使用 `new webpack.optimize.CommonsChunkPlugin`,在watch情况下，大多数情况只编译业务包。速度4s左右
- 如果打包同时做uglify, `new webpack.optimize.UglifyJsPlugin`组件打包变慢，边打包边uglify。看流程是否需要开启
- `devtool`的选择, source-map > eval > cheap-module-eval-source-map。如果没有需求，选快的
- `externals`, 去除react或者react-dom可以提高打包速度。（待思考）


#### 控制变量测试（可自行测试）
|编号| 包数量      |    是否压缩 | devTool | 包大小 | build时间| watch 时间,改动一行代码|
| ------------- |:-------------:|:-------------:| :-------------:|:-------------:| :-------------:|   -----:|
|1| 1个包  | 不压缩 |  soure-map    | 3M  3.52(map) | 45s| 45s|
|2| 1个包  | 压缩 |   soure-map   | 1.14M 7.23M  (map) |65s| 68s|
|3| 2个包   | 不压缩| sourc-map  |common(1.13M 1.16M)react全家桶 app(1.93M 1.36M)业务代码| 38.1s|5.5s左右|
|4| 2个包   | 不压缩|eval |common(1.21M)react全家桶 app(2.1M)业务代码| 32.7s|4s左右|
|5|2个包|不压缩|cheap-module-eval-source-map|3.01M 4.21M| 35s| 2.4s左右|


#### 1.抽出共有包
表格1，3比较 分两个包第一次build时间更快，代码变动build更快

#### 2.压缩方案
表格1，2打包时是否引入代码uglify，引入变慢

#### 3.devtool选择 [官网devtool](https://webpack.github.io/docs/configuration.html#devtool)
表格3，4，5 cheap-module-eval-source-map>eval>soure-map (first build time),实际上根据官网描述 eval最优？？我的选择
- 如果用于开发 `cheap-module-eval-source-map`
- 如果用于打包 `eval`, 42.5s>50s



结论：如果使用监听不断打包的方式开发，1.拆分共有包和业务包，2.不使用uglify，3.devtool根据自己的打断点调试需求选择。（关注增量编译的时间）但是最好的方案应该还是使用`webpack-dev-server`进行本地开发。而不是实时编译开发。而对于ci自动发布而做build打包的情况（关注build时间），1分钟左右还是可以接受的。


### 疑问点

#### 1.使用resolve alias是否能够优化速度
帮助模块引用指定路径，比如使用压缩包。理论上应该有减少webpack查找的成本，但是实际上测试下来，没有看出效果改变（不稳定）。所以此处保留疑问。 可以给大家提供的帮助。如果遇到`his seems to be a pre-built javascript file`问题，解决方法module.noParse配置此模块的忽略。
#### 2.使用externals，去除react的方案。
没有试过，看起来把react抽出共有包。再在页面引入是一件非常诡异的事情。