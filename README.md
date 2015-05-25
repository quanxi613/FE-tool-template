# FE-tool-template

前端开发模板，提高前端开发效率

使用`gulp`作为自动化构建工具，`yeoman`生成项目文件代码结构，`bower`做包管理器

##准备工作##


###安装nodejs###

1. windows系统下安装nodejs及环境配置
	
   - 下载nodejs，官网[http://nodejs.org/download/](http://nodejs.org/download/)，选择适合你电脑配置的版本下载

   - 下载完成后双击无脑安装，装好后在cmd控制台输入：`node -v`，控制台打印出版本号，则安装成功。

   - npm安装  新版nodejs已经集成npm，在cmd控制台输入`npm -v`，打印出版本号，则安装成功。

   - 接下来便可以使用npm命令安装模块了
  
        安装：`npm install <module name>`

		全局安装：`npm install -g <Module Name>`

		卸载模块：`npm uninstall <Module Name>`

		显示当前目录下安装的模块：`npm list`

2. mac系统下安装nodejs

   - 下载nodejs，官网[http://nodejs.org/download/](http://nodejs.org/download/)，选择mac版本下载

   - 下载完成后，点击安装

顺便列两个常用命令行

`cd ` 定位到目录

`ls ` 列出文件列表 


###安装gulp###

`npm install -g gulp`

输入`gulp -v`，查看版本号确保正确安装

接着将gulp安装到项目本地

`npm install --save-dev gulp`

这里使用`--save-dev`来更新package.json文件，更新devDependencies值，表明项目需要依赖gulp

###安装yeoman###

yeoman：在web项目立项阶段用来生成项目的文件、代码结构，包括代码校验、测试和压缩等功能。

`npm install -g yo`

###安装bower###

bower：web的包管理器

`npm install -g bower`


##正式开始##

###用gulp-webapp生成项目目录###

安装gulp-webapp

`npm install -g generator-gulp-webapp`

在项目目录下，输入命令行

`yo gulp-webapp 项目名`

安装时会提示是否安装sass，bootstrap和modernizr，除了sass，另外两个都选中安装。（项目主要用less，所以不安装sass）

此时项目下已生成项目目录

按照项目习惯，将app下的styles改为css，scripts改为js，images改为img，.tmp下的styles改为css。此时别忘了修改`gulpfile.js`，修改后的文件如下：

```
/*global -$ */
'use strict';
var gulp = require('gulp');
var $ = require('gulp-load-plugins')();
var browserSync = require('browser-sync');
var reload = browserSync.reload;

gulp.task('styles', function () {
  return gulp.src('app/css/main.css')
    .pipe($.sourcemaps.init())
    .pipe($.postcss([
      require('autoprefixer-core')({browsers: ['last 1 version']})
    ]))
    .pipe($.sourcemaps.write())
    .pipe(gulp.dest('.tmp/css'))
    .pipe(reload({stream: true}));
});

gulp.task('jshint', function () {
  return gulp.src('app/js/**/*.js')
    .pipe(reload({stream: true, once: true}))
    .pipe($.jshint())
    .pipe($.jshint.reporter('jshint-stylish'))
    .pipe($.if(!browserSync.active, $.jshint.reporter('fail')));
});

gulp.task('html', ['styles'], function () {
  var assets = $.useref.assets({searchPath: ['.tmp', 'app', '.']});

  return gulp.src('app/*.html')
    .pipe(assets)
    .pipe($.if('*.js', $.uglify()))
    .pipe($.if('*.css', $.csso()))
    .pipe(assets.restore())
    .pipe($.useref())
    .pipe($.if('*.html', $.minifyHtml({conditionals: true, loose: true})))
    .pipe(gulp.dest('dist'));
});

gulp.task('images', function () {
  return gulp.src('app/img/**/*')
    .pipe($.cache($.imagemin({
      progressive: true,
      interlaced: true,
      // don't remove IDs from SVGs, they are often used
      // as hooks for embedding and styling
      svgoPlugins: [{cleanupIDs: false}]
    })))
    .pipe(gulp.dest('dist/img'));
});

gulp.task('fonts', function () {
  return gulp.src(require('main-bower-files')({
    filter: '**/*.{eot,svg,ttf,woff,woff2}'
  }).concat('app/fonts/**/*'))
    .pipe(gulp.dest('.tmp/fonts'))
    .pipe(gulp.dest('dist/fonts'));
});

gulp.task('extras', function () {
  return gulp.src([
    'app/*.*',
    '!app/*.html'
  ], {
    dot: true
  }).pipe(gulp.dest('dist'));
});

gulp.task('clean', require('del').bind(null, ['.tmp', 'dist']));

gulp.task('serve', ['styles', 'fonts'], function () {
  browserSync({
    notify: false,
    port: 9000,
    server: {
      baseDir: ['.tmp', 'app'],
      routes: {
        '/bower_components': 'bower_components'
      }
    }
  });

  // watch for changes
  gulp.watch([
    'app/*.html',
    'app/js/**/*.js',
    'app/img/**/*',
	'app/tpl/*',
    '.tmp/fonts/**/*'
  ]).on('change', reload);

  gulp.watch('app/css/**/*.css', ['styles']);
  gulp.watch('app/fonts/**/*', ['fonts']);
  gulp.watch('bower.json', ['wiredep', 'fonts']);
});

// inject bower components
gulp.task('wiredep', function () {
  var wiredep = require('wiredep').stream;

  gulp.src('app/*.html')
    .pipe(wiredep({
      ignorePath: /^(\.\.\/)*\.\./
    }))
    .pipe(gulp.dest('app'));
});

gulp.task('build', ['jshint', 'html', 'images', 'fonts', 'extras'], function () {
  return gulp.src('dist/**/*').pipe($.size({title: 'build', gzip: true}));
});

gulp.task('default', ['clean'], function () {
  gulp.start('build');
});

```


目录说明：

- .tmp：临时目录
- app：开发的源代码目录
- bower_components：通过bower下载下来的包
- dist：生成用于发布的项目
- node_modules：nodejs依赖包
- test：测试文件的目录
- .bowerrc：bower属性
- .editorconfig：对开发工具的属性配置
- .gitattributes：git属性的配置
- .gitignore：git管理文件的配置
- .jshintrc：JSHint配置
- bower.json：bower依赖管理
- gulpfile.js：gulp开发过程管理
- package.json：项目依赖文件

app目录下的结构说明：

- css: 存放css文件
- font：存放字体资源
- img：存放图片资源
- js：存放js源文件
- tpl（自己添加）：存放模板文件

##运行##

输入命令行

`gulp serve`

浏览器会自动打开app下的index.html，localhost：9000

任何文件的改动都会自动刷新浏览器

编译文件

`gulp build`

gulp将build后的文件放入dist目录下

