### webpack打包经常遇到的问题

讨论的问题包括

- 依赖挂载到全局，如jquery
- jquery plugin引入
- 如何打共有包
- handlebar如何引入
- hmr + webpack-dev-server 如何做依赖环境
- alias 使用报错


#### 1.依赖挂载到全局，如jquery
方案一 
[Expose jQuery to real Window object with Webpack](http://stackoverflow.com/questions/29080148/expose-jquery-to-real-window-object-with-webpack)
```
使用expose-loader

loaders: [
    { test: require.resolve('jquery'), loader: 'expose?jQuery!expose?$' }
]
```
方案二
```
window.$ = window.JQuery = JQuery;
入口的js,手工挂载
```

方案三
```
plugins: [
  new webpack.ProvidePlugin({
     $: "jquery",
     jQuery: "jquery",
     "window.jQuery": "jquery"
  })
]
```


#### 2.jquery plugin引入 

把jquery挂载到window能解决很多问题

- [Managing Jquery plugin dependency in webpack](http://stackoverflow.com/questions/28969861/managing-jquery-plugin-dependency-in-webpack)
- [shimming modules](https://github.com/webpack/docs/wiki/shimming-modules)
- [https://github.com/webpack/webpack/issues/717/](jQuery plugins available for all modules? )


#### 3.如何打共有包
- CommonsChunkPlugin

```
plugins: [
    new webpack.optimize.CommonsChunkPlugin(/* chunkName= */"common", /* filename= */"common.js")
]
```

#### 4.handlebar如何引入
[Using Handlebars with webpack warning require.extentions not supported](https://github.com/wycats/handlebars.js/issues/953)
```
{
  resolve: {
    modulesDirectories: ['node_modules', 'src'],
    fallback: path.join(__dirname, 'node_modules'),
    alias: {
      'handlebars': 'handlebars/runtime.js'
    }
  },
  resolveLoader: {
    fallback: path.join(__dirname, 'node_modules'),
    alias: {
      'hbs': 'handlebars-loader'
    }
  }
}
```

#### 5.hmr + webpack-dev-server 如何做依赖环境       [webpack-hugin-demo](https://github.com/iscarecrow/webpack-hugin-demo)
- HotModuleReplacementPlugin
- webpack-dev-server
```
  entry: [
    'webpack-dev-server/client?http://localhost:8080',
    'webpack/hot/dev-server',
    './src/buy/js/index.js'
  ],
  plugins: [new webpack.HotModuleReplacementPlugin()]
```

#### 6.alias引入组件报错

如果报 `this seems to be a pre-built javascript file` 的错误，可以使用noParse处理
```
resolve: {
    extensions: ["",".js",".jsx",".es6"],
    modulesDirectories: ['node_modules', 'src'],
    fallback: path.join(__dirname, 'node_modules'),
    alias: {
      'jquery': 'jquery/dist/jquery.min.js',
      'redux': 'redux/dist/redux.min.js',
      'react': 'react/dist/react.min.js',
      'react-dom': 'react-dom/dist/react-dom.min.js',
      'react-redux': 'react-redux/dist/react-redux.min.js'
    }
  },
  module: {
    noParse: [path.resolve(__dirname, './node_modules/react/dist/react.min.js')],
    loaders: [{
      test: /\.jsx?$/,
      exclude: /(node_modules|bower_components)/,
      loaders: ['babel?presets[]=react,presets[]=es2015'],
    }]
  }
```