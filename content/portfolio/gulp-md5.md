+++

draft = false
image = "img/portfolio/gulp.png"
showonlyimage = false
date = "2017-11-14T10:22:59+08:00"
title = "gulp自动为静态资源添加md5戳"
weight = 1
+++
<!--more-->
### 起因
浏览器的缓存机制，导致web项目的静态资源不能及时获取。目前最优解决方案是：基于文件内容的hash版本冗余机制。

我们希望开发的时候静态资源是酱紫的：
    
    <link href="css/style.css" rel="stylesheet">

上线时，静态资源是酱紫的：
    
    <link href="css/style_4cbfcb.css" rel="stylesheet">

### 试试
查找资料，两种办法比较科学，如下：
#### 1.一步到位型
使用gulp-md5-plus（常规方法gulp-rev、gulp-rev-collerctor也可以做到一样的效果，不过不推荐）

    npm install --save-dev gulp-md5-plus

gulp文件中配置

    var md5 = require("gulp-md5-plus");
    
打包的时候，移动css的代码中加上一段

    //config是配置的静态资源路径
    gulp.task('pack:css', function() {
    return gulp.src(config.pack.css.src)
        .pipe(minifyCSS())
        .pipe(md5(6,config.pack.html.src)) //css增加md5版本号，6位
        .pipe(gulp.dest(config.pack.css.dest));
    });

于是自动给静态资源加md5版本号，且自动替换页面中的路径。

    <link href="css/style_4cbfcb.css" rel="stylesheet">

这种做法有以下的好处：

* 线上的文件不是同名文件覆盖，而是文件名+hash的冗余，所以可以先上线静态资源，再上线页面，不存在间隙问题。
* 遇到版本回滚问题，只需要回滚页面，无需回滚静态资源。
* 由于静态资源版本号是文件内容的hash，因此所有静态资源可以开启永久强缓存，只有更新了内容文件才会缓存失效，缓存利用率大增。
* 修改静态资源会在线上产生新的文件，一个文件对应一个版本，不会受到构造CDN缓存形式的攻击。


#### 1.hack插件方法
有些时候，我们需要的是下边的情况

    <link href="css/style.css" rel="stylesheet">
    =>> <link href="css/style.css?v=4cbfcb" rel="stylesheet">

如果这样的话，上边提到的gulp-md5-plus，以及常规方法gulp-rev、gulp-rev-collerctor都无法使用了。

不过需要hack一下，gulp-rev、gulp-rev-collerctor，可以达成这个效果。
    
    //安装对应版本的插件
    npm install --save-dev gulp-rev@5.1.0  
    npm install --save-dev gulp-rev-collector@1.0.0

第一步，打开node_modules\gulp-rev\index.js
    
    第135行 manifest[originalFile] = revisionedFile;
    更新为: manifest[originalFile] = originalFile + '?v=' + file.revHash;

第二步，打开node_modules\rev-path\index.js

    10行 return filename + '-' + hash + ext;
    更新为: return filename + ext;

第三步，打开node_modules\gulp-rev-collector\index.js

    31行 if ( path.basename(json[key]).replace(new RegExp( opts.revSuffix ), '' ) !== path.basename(key) ) {
    更新为: if ( path.basename(json[key]).split('?')0 !== path.basename(key) ) {
    
正常使用gulp-rev和gulp-rev-collector，得到的效果就是
    
    <link href="css/style.css" rel="stylesheet">
    =>> <link href="css/style.css?v=4cbfcb32" rel="stylesheet">

这种方案使用的插件版本分别是
    
    "gulp-rev": "^5.1.0",
    "gulp-rev-collector": "^1.0.0",

如果npm install的是最新的插件，代码位置需要调整，根据版本不一样，改法是不一致的，不推荐大家使用这种方法，除非专门改一个放在npm上，否则不利于团队协作。


>参考文档
>静态资源缓存与更新：https://my.oschina.net/jathon/blog/404968
>gulp-md5-plus：https://www.npmjs.com/package/gulp-md5-plus
>gulp-rev：https://www.npmjs.com/package/gulp-rev
