---
title: Angular-预编译和摇树
date: 2017-09-03 17:48:48
tags: Augular
categories: 前端
comments: true
---

## AOT预编译

使用AOT，编译器仅仅使用一组库在构建期间运行一次；使用JIT，编译器在每个用户的每次运行期间都要用不同的库运行一次，更适合实时开发
AOT编译的好处：
<!--more-->

- 渲染得更快
使用AOT，浏览器下载预编译版本的应用程序。 浏览器直接加载运行代码，所以它可以立即渲染该应用，而不用等应用完成首次编译。
- 需要的异步请求更少
编译器把外部HTML模板和CSS样式表内联到了该应用的JavaScript中。 消除了用来下载那些源文件的Ajax请求。
- 需要下载的Angular框架体积更小
如果应用已经编译过了，自然不需要再下载Angular编译器了。 该编译器差不多占了Angular自身体积的一半儿，所以，省略它可以显著减小应用的体积。
- 提早检测模板错误
AOT编译器在构建过程中检测和报告模板绑定错误，避免用户遇到这些错误。
- 更安全
AOT编译远在HTML模版和组件被服务到客户端之前，将它们编译到JavaScript文件。 没有模版可以阅读，没有高风险客户端HTML或JavaScript可利用，所以注入攻击的机会较少。

### Aot设置

首先,使用git clone https://github.com/angular/quickstart.git quickstart 设置本地简单的项目开发环境
在项目根目录/quickstart，安装依赖

    npm install @angular/compiler-cli @angular/platform-server --save

复制原tsconfig.json到项目根目录tsconfig-aot.json文件，修改如下

    //tsconfig-aot.json
    {
      "compilerOptions": {
        "target": "es5",
        "module": "es2015",
        "moduleResolution": "node",
        "sourceMap": true,
        "emitDecoratorMetadata": true,
        "experimentalDecorators": true,
        "lib": ["es2015", "dom"],
        "noImplicitAny": true,
        "suppressImplicitAnyIndexErrors": true,
        "typeRoots": [
          "./node_modules/@types/"
        ]
      },

      "files": [
        "src/app/app.module.ts",
        "src/main.ts"
      ],

      "angularCompilerOptions": {
       "genDir": "aot",
       "skipMetadataEmit" : true
     }
    }

执行命令ngc编译器来启动AOT编译

    node_modules/.bin/ngc -p tsconfig-aot.json

ngc希望-p选项指向一个tsconfig.json文件，或者一个包含tsconfig.json文件的目录。
在ngc完成时，会在aot目录下看到一组NgFactory文件（该目录是在tsconfig-aot.json的genDir属性中指定的）。
这些工厂文件对于编译后的应用是必要的。 每个组件工厂都可以在运行时创建一个组件的实例，其中带有一个原始的类文件和一个用JavaScript表示的组件模板。注意，原始的组件类依然是由所生成的这个工厂进行内部引用的。

**修改启动**
备份原main.ts为main-jit.ts，将main.ts改为

    import { platformBrowser }    from '@angular/platform-browser';
    import { AppModuleNgFactory } from '../aot/src/app/app.module.ngfactory';

    console.log('Running AOT compiled');
    platformBrowser().bootstrapModuleFactory(AppModuleNgFactory);

重新编译应用

### Rollup摇树优化

Rollup会通过跟踪import和export语句来对本应用进行静态分析。 它所生成的最终代码捆中会排除那些被导出过但又从未被导入的代码。
Rollup只能对ES2015模块摇树，因为那里有import和export语句
安装Rollup依赖

    npm install rollup rollup-plugin-node-resolve rollup-plugin-commonjs rollup-plugin-uglify --save-dev

在项目根目录新建一个配置文件（rollup-config.js），来告诉Rollup如何处理应用

    //rollup-config.js
    import nodeResolve from 'rollup-plugin-node-resolve';
    import commonjs    from 'rollup-plugin-commonjs';
    import uglify      from 'rollup-plugin-uglify';

    export default {
      entry: 'src/main.js',
      dest: 'src/build.js', // output a single application bundle
      sourceMap: false,
      format: 'iife',
      onwarn: function(warning) {
        // Skip certain warnings

        // should intercept ... but doesn't in some rollup versions
        if ( warning.code === 'THIS_IS_UNDEFINED' ) { return; }

        // console.warn everything else
        console.warn( warning.message );
      },
      plugins: [
          nodeResolve({jsnext: true, module: true}),
          commonjs({
            include: 'node_modules/rxjs/**',
          }),
          uglify()
      ]
    };

执行摇树优化

    node_modules/.bin/rollup -c rollup-config.js

备份原index.html为index-jit.html，修改脚本配置：

    System.import('main-jit.js').catch(function(err){ console.error(err); });

修改index.html

    <!DOCTYPE html>
    <html>
      <head>
        <title>Ahead of time compilation</title>
        <base href="/">
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <link rel="stylesheet" href="styles.css">

        <script src="node_modules/core-js/client/shim.min.js"></script>
        <script src="node_modules/zone.js/dist/zone.js"></script>
      </head>
      <body>
        <my-app>Loading...</my-app>
      </body>
      <script src="build.js"></script>
    </html>

在package,json的scripts加上

    "build:aot": "ngc -p tsconfig-aot.json && rollup -c rollup-config.js",

可通过**npm run build:aot**同时进行编译和摇树优化

此时，npm start可启动应用，但这时候时按AOT方式启动的，修改内容不会即时编译生效；在url后面加上index-jit.html，比如

    http://localhost:3000/index-jit.html

可切换值jit开发方式，可以实时编译，实现了aot和jit同时存在

项目根目录添加文件copy-dist-files.js

    var fs = require('fs');
    var resources = [
      'node_modules/core-js/client/shim.min.js',
      'node_modules/zone.js/dist/zone.min.js',
      'src/styles.css'
    ];
    resources.map(function(f) {
      var path = f.split('/');
      var t = 'aot/' + path[path.length-1];
      fs.createReadStream(f).pipe(fs.createWriteStream(t));
    });

使用node copy-dist-files拷贝AOT发布文件到/aot/目录

在package,json的scripts加上

    "serve:aot": "lite-server -c bs-config.aot.json"

使用

    npm run build:aot && npm run serve:aot

同时编译和启动应用，但这时是AOT预编译的，不能实时开发