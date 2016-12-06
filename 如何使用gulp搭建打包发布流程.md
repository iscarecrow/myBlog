### 如何使用gulp搭建打包发布流程

#### 开发环境
- requireJs
- browserify
- 原生代码
- es6 import

开发环境，使用构建工具。关注的点是代码的模块化方式，从源码编译成可运行代码的过程等。本章主要讲的问题是发布流程所需要做的工作。

#### 发布环境
代码发布会有几个流程

- 合并
- 压缩
- 静态加戳
- 模版戳替换
- 静态资源+模版提供使用


##### 1.合并

将模块化的代码，合并编译。
1.`gulp-usemin`
2.`requireJS`
3.`browserify`
4.`es6 import`

##### `gulp-usemin`，模版上写上合并的依赖

```
<!-- build:inlinecss ./static/bundle.css -->
<link rel="stylesheet" type="text/css" href="./static/a.css">
<link rel="stylesheet" type="text/css" href="./static/b.css">
<!-- endbuild -->

<!-- build:inlinejs ./static/bundle.js -->
<script src="./static/a.js"></script>
<script src="./static/b.js"></script>
<!-- endbuild -->

gulp.task('usemin', function () {
  return gulp.src('./a.html')
      .pipe(usemin())
      .pipe(gulp.dest('../build/'));
});

//合并出需要的js，css
```

##### `requireJS` [requireDemo]()
使用rj合并require方式依赖的代码
```
gulp.task('jsmin', function(cb) {
  rjs.optimize({
    mainConfigFile: "static/js/h5-main-dev.js",
    baseUrl: 'static/js',
    removeCombined: false,
    findNestedDependencies: false,
    optimize: "none",
    optimizeCss: "none",
    preserveLicenseComments: "false",
    paths: {
      machina: "empty:"
    },
    dir: './dist/js',
    generateSourceMaps: false,
    modules: h5modules
  }, function(buildResponse) {
    // console.log('build response', buildResponse);
    cb();
  }, cb);
})
```

##### `browserify` [browserifyDemo]()
处理browserify的方式
```
gulp.task('browserify', function() {
    var bundler = browserify({
        entries: ['./src/js/app.js'], // Only need initial file, browserify finds the deps
        transform: [reactify], // We want to convert JSX to normal javascript
        debug: false, // Gives us sourcemapping
        cache: {},
        packageCache: {},
        fullPaths: true // Requirement of watchify
    });
    var watcher = watchify(bundler);
    return watcher
        .on('update', function() { // When any files update
            var updateStart = Date.now();
            console.log('Updating!');
            watcher.bundle() // Create new bundle that uses the cache for high performance
            .pipe(source('main.js'))
                // This is where you add uglifying etc.
            .pipe(gulp.dest('./static/js/'));
            console.log('Updated!', (Date.now() - updateStart) + 'ms');
        })
        .bundle() // Create the initial bundle when starting the task
        .pipe(source('main.js'))
        .pipe(gulp.dest('./static/js/'));
});

```

##### `es6 import`
见： [webpack打包es6 demo](https://github.com/iscarecrow/webpack-hugin-demo)
 配置webpack,编译合并代码


##### 2.压缩
压缩的目的是减少包的大小，混淆代码。

```
// 对js进行jsuglify， 可是使用 gulp-uglify
gulp.task('jsuglify', function() {
  return gulp.src('./static/js/**/*.js')
    .pipe(uglify())
    .pipe(gulp.dest('./dist/js/'));
});
// 从static文件生成dist

// 对 css压缩，可以使用 gulp-minify-css
gulp.task('cssmin', function() {
  return gulp.src(
      './static/css/**/*.css'
    )
    .pipe(minifyCSS())
    .pipe(gulp.dest('./dist/css'));
});
// 从static文件生成dist

```

#### 3.静态加戳

为了解决浏览器缓存问题，文件变动时。主动改变js的戳。使浏览器能够获取最新的js。

```
// 静态资源，加戳后生成新的静态资源。可以使用 gulp-rev-all
gulp.task('rev', function() {
  var revAll = new RevAll();
  return gulp.src(['dist/**/*.js','dist/**/*.css'])
    .pipe(revAll.revision())
    .pipe(gulp.dest('cdn'))
    .pipe(revAll.versionFile())
    .pipe(gulp.dest('rev'))
    .pipe(revAll.manifestFile())
    .pipe(gulp.dest('rev'));
});

// 从dist目录（压缩后的）生成到cdn. a.js--> a.xxx.js
// rev目录生成 rev-manifest.json(md5的map) rev-version.json(版本)
```


####  4.模版戳替换

```
// 模版上的js和css链接要替换
gulp.task("revreplace", function(){
  var manifest = gulp.src('./rev/rev-manifest.json');
  return gulp.src("./view/**/*")
    .pipe(revReplace({
      replaceInExtensions: ['.css','.js','.html'],
      manifest: manifest
    }))
    .pipe(gulp.dest('./build/view'));
});

// 替换前后。html内部 引用的变化
<link rel="stylesheet" type="text/css" href="./static/css/demo2.css">(替换前)
<link rel="stylesheet" type="text/css" href="./static/css/demo2.3167210f.css">（替换后）

<script type="text/javascript" src="./static/js/demo1.js"></script>（替换前）

<script type="text/javascript" src="./static/js/demo1.e5ca275c.js"></script>（替换后）

```

#### 5.静态资源+模版提供使用

```
最后把静态资源和替换后的模版,copy到build目录。整个目录和后端对接，静态资源推送到远程cdn,模版资源推到后台应用目录下启动。
```

