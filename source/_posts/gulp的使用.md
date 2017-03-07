---
title:  "gulp的使用"
tag: gulp
---
gulp，自动化的构建工具增加工作流程。gulp.js是基于Node.js构建的，利用Node.js流的威力，快速构建项目并减少频繁的io操作。
<!-- more --> 
### gulp.src(globs[,options)
 输出符合所提供的匹配模式或者匹配模式数组的文件。
  {% codeblock %}
     gulp.src([ ‘./src/js/**/*.js',
      ‘!./src/js/templates', 
      ‘!./src/js/?(build|examples|server|src-test|resources)/**/**',
      ‘!./src/{component,component/**}' ])
 {% endcodeblock %}
忽略文件夹：`‘!./src/{component,component/**}’` 可以忽略掉component文件夹和component文件夹里面的东西。（也可以写成‘!./src/component{,/\*\*}’）
>‘!./src/component/**’只能忽略掉component文件夹里面的东西，不能忽略掉文件夹本身。

Gulp内部使用了node-glob模块来实现其文件匹配功能,用到的glob的匹配规则以及一些文件匹配技巧如下。
- \* 匹配文件路径中的0个或多个字符，但不会匹配路径分隔符，除非路径分隔符出现在末尾
- \** 匹配路径中的0个或多个目录及其子目录,需要单独出现，即它左右不能有其他东西了。如果出现在末尾，也能匹配文件。
- ? 匹配文件路径中的一个字符(不会匹配路径分隔符)
- [...] 匹配方括号中出现的字符中的任意一个，当方括号中第一个字符为^或!时，则表示不匹配方括号中出现的其他字符中的任意一个，类似js正则表达式中的用法
- !(pattern|pattern|pattern) 匹配任何与括号中给定的任一模式都不匹配的
- ?(pattern|pattern|pattern) 匹配括号中给定的任一模式0次或1次，类似于js正则中的(pattern|pattern|pattern)?
- +(pattern|pattern|pattern) 匹配括号中给定的任一模式至少1次，类似于js正则中的(pattern|pattern|pattern)+
- *(pattern|pattern|pattern) 匹配括号中给定的任一模式0次或多次，类似于js正则中的(pattern|pattern|pattern)*
- @(pattern|pattern|pattern) 匹配括号中给定的任一模式1次，类似于js正则中的(pattern|pattern|pattern)

下面以一系列例子来加深理解

- \* 能匹配 a.js,x.y,abc,abc/,但不能匹配a/b.js
- \*\.\* 能匹配 a.js,style.css,a.b,x.y
- \*/\*/\*.js 能匹配 a/b/c.js,x/y/z.js,不能匹配a/b.js,a/b/c/d.js
- ** 能匹配 abc,a/b.js,a/b/c.js,x/y/z,x/y/z/a.b,能用来匹配所有的目录和文件
- **/*.js 能匹配 foo.js,a/foo.js,a/b/foo.js,a/b/c/foo.js
- a/**/z 能匹配 a/z,a/b/z,a/b/c/z,a/d/g/h/j/k/z
- a/\*\*b/z 能匹配 a/b/z,a/sb/z,但不能匹配a/x/sb/z,因为只有单**单独出现才能匹配多级目录
- ?.js 能匹配 a.js,b.js,c.js
- a?? 能匹配 a.b,abc,但不能匹配ab/,因为它不会匹配路径分隔符
- [xyz].js 只能匹配 x.js,y.js,z.js,不会匹配xy.js,xyz.js等,整个中括号只代表一个字符
- [^xyz].js 能匹配 a.js,b.js,c.js等,不能匹配x.js,y.js,z.js

[参考资料](http://www.cnblogs.com/2050/p/4198792.html)
### gulp.dest(path[, options])
 写入文件, 作为pipe的一个流程.文件夹不存在时会被自动创建.
### gulp.task(name[, deps], fn)
 定义一个任务,声明它的名称, 任务依赖, 和任务内容.
 gulp中的任务是异步运行的，gulp便默认将并行所有的任务。
 {% codeblock %}
 gulp.task('release', ['clean', 'minify'], function(){
    // do stuff
 
 });
 {% endcodeblock %}
 
 如果希望任务T2在T1之后运行，那么：
 
 1 在 T1 中返回 Promise、Stream 类型的值，或者接受一个 callback 回调函数作为参数
 2 在 T2 的声明中定义 T1 为其依赖
 {% codeblock %}
 gulp.task('clean', function(){
     return gulp.src("./dist/**/*.js", { read: false }).pipe(rimraf());
 });
 
 gulp.task('minify', ['clean'], function(){
     return  gulp.src('./js/**/*.js').pipe(uglify()).pipe(gulp.dest("./dist/js"));
 });
 gulp.task('release', ['clean', 'minify'], function(){
    // do stuff
 
 });
 {% endcodeblock %}
 
 弊端：
 
 minify 任务现在依赖了 clean 任务，而它的业务原本是不依赖 clean 的——我们只是为了“屈从 Gulp 的设计”才定义了这样的依赖关系。这种在 minify 中隐式包含 clean 的做法有时候会带来麻烦。比如，你在执行的时候，确实只是需要执行一个 minify，怎么办？那时，你就需要定义一个专门的 minify-only
 那么，怎样才能“优雅地”逐个同步地运行 Gulp 任务呢？
  **run-sequence可以按顺序地运行多个或多组任务；**
  {% codeblock %}
  var runSequence = require('run-sequence');
  gulp.task('default', function(callback) {
      runSequence('clean',
          ['less', 'scripts'],
          'watch',
          callback);
  });
  {% endcodeblock %}
  在上述代码中，clean 先于所有其他任务运行，在 clean 完成后，less 与 scripts 同时运行；在 less 与 scripts 都运行完成之后，watch 最后运行。并且，在 watch 运行完毕后，会调用 callback，以通知 Gulp 引擎。
  [参考链接](https://blog.jijiechen.com/post/run-gulp-tasks-and-steps-synchronously)
  ###gulp.watch(glob [, opts], tasks)
  监控文件,执行任务.
  
>  常用插件集合

  压缩图片
  https://www.npmjs.com/package/gulp-imagemin
  自动生成文件版本
  https://github.com/sindresorhus/gulp-rev
  使用gulp压缩js
  https://github.com/nimojs/gulp-book/blob/master/chapter2.md
   