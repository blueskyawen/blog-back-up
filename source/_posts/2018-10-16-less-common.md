---
title: less.js用法（译）
date: 2018-08-10 00:00:15
tags: Less
categories: 前端
comments: true
---

# 命令行用法
使用命令行将.less文件编译为.css
## 安装

    npm install less -g

-g选项代表安装全局，安装后可用于命令行。 如果需要安装特定版本或标记的less，可以在包名称之后添加@VERSION，例如
<!--more-->

    npm install less@2.7.1 -g

## 开发本地安装
如果您不想全局安装，也可以本地安装

    npm i less --save-dev

这将在项目文件夹中安装最新的lessc正式版，并将其添加到项目的package.json的devDependencies中

## lessc的Beta版本
随着新功能的开发，lessc版本将定期的发布到npm，标记为beta。
这些版本不会作为@latest官方发行版发布，并且通常会在版本中发布beta版（使用lessc -v获取当前版本）。
由于补丁版本不会中断，我们将立即发布补丁版本，alpha / beta /候选版本将作为次要或主要版本升级发布（我们努力从1.4.0开始遵循语义版本控制）。

## 服务端命令行用法
二进制文件bin / lessc可与* nix，OS X和Windows上的Node.js一起使用。
### 用法

    lessc [option option=parameter ...] <source> [destination]

如果source设置为` - '（破折号或连字符减号），则从stdin读取输入。
例子：

    lessc bootstrap.less bootstrap.css

## lessc配置参数
想要获取更多的参数和用法，请参看[Less Options][1]
##### Silent
停止显示任何警告。

    lessc -s lessc --silent

##### Version
查看版本。

    lessc -v  / lessc --version

##### help
打印包含可用选项和退出的帮助消息。

    lessc --help / lessc -h

##### Makefile
将导入到依赖关系列表的生成文件输出到stdout。

    lessc -M
    lessc --depends

##### No Color

    lessc --no-color

##### Clean CSS
在v2中，Clean CSS不再作为直接依赖项包含在内。 要使用Clean CSS，请使用clean css插件。

# 浏览器端用法
在浏览器中使用Less.js是最简单的入门方式，便于使用Less进行开发，但在生产中，当性能和可靠性很重要时，我们建议使用Node.js或许多第三方工具之一进行预编译
首先，将.less样式表与rel属性设置为“stylesheet / less”链接：

    <link rel =“stylesheet / less”type =“text / css”href =“styles.less”/>

然后，将less.js包含在页面head元素的`<script> </ script>`标记中：

    <script src =“less.js”type =“text / javascript”> </ script>

## 配置参数
可以通过在less脚本加载之前使用对象来设置参数选项，这会影响所有初始链接标记和less使用。

    <script>
      less = {
        env: "development",
        async: false,
        fileAsync: false,
        poll: 1000,
        functions: {},
        dumpLineNumbers: "comments",
        relativeUrls: false,
        rootpath: ":/a.com/"
      };
    </script>
    <script src="less.js"></script>

另一张方式也可以再标签上直接设置参数

    <script>
      less = {
        env: "development"
      };
    </script>
    <script src="less.js" data-env="development"></script>

或者为了简洁起见，可以将它们设置为脚本和链接标记上的属性：

    <script src="less.js" data-poll="1000" data-relative-urls="false"></script>
    <link data-dump-line-numbers="all" data-global-vars='{ "myvar": "#ddffee", "mystr": "\"quoted\"" }' rel="stylesheet/less" type="text/css" href="less/styles.less">

## 浏览器环境支持
Less.js支持所有现代浏览器（Chrome，Firefox，Safari，IE11 +和Edge的最新版本）。 虽然在生产中可以在客户端使用Less，但请注意这样做会对性能产生影响（尽管Less的最新版本速度要快得多）。 此外，如果发生JavaScript错误，有时会出现装饰性问题。 这是灵活性与速度的折衷。 为了尽可能提高静态网站的性能，我们建议在服务器端编译Less。

## 注意事项

- 确保在脚本之前包含样式表。
- 当您链接多个.less样式表时，每个样式表都是独立编译的
- 因此，您在样式表中定义的任何变量，混合或名称空间都无法访问。
- 由于浏览器的原始策略相同，因此加载外部资源需要启用CORS

## 监控模式
要启用监视模式，必须将选项env设置为开发。然后在包含less.js文件之后，调用less.watch（），如下所示：

    <script>less = { env: 'development'};</script>
    <script src="less.js"></script>
    <script>less.watch();</script>

或者，您可以通过将＃！watch附加到URL来临时启用“监视”模式。

## 变量修改
启用Less变量的运行时修改。 使用新值调用时，将重新编译Less文件而不重新加载。 简单的基本用法：

    less.modifyVars({
      '@buttonFace': '#5B83AD',
      '@buttonText': '#D9EEF2'
    });

## 调试
可以在CSS中输出规则，允许工具找到规则的来源。
如上所述指定选项dumpLineNumbers或将！dumpLineNumbers：mediaquery添加到url
您可以将mediaquery选项与FireLESS一起使用（它与SCSS媒体查询调试格式相同）。 另请参阅FireLess和Less v2。 comment选项可用于在内联编译的CSS代码中显示文件信息和行号。

## 选项

    <!-- set options before less.js script -->
    <script>
      less = {
        env: "development",
        logLevel: 2,
        async: false,
        fileAsync: false,
        poll: 1000,
        functions: {},
        dumpLineNumbers: "comments",
        relativeUrls: false,
        globalVars: {
          var1: '"quoted value"',
          var2: 'regular value'
        },
        rootpath: ":/a.com/"
      };
    </script>
    <script src="less.js"></script>

## lessc配置参数
想要获取更多的参数和用法，请参看[Less Options][1]
##### 异步async
类型：布尔值
默认值：false
是否使用async选项请求导入文件。请参阅fileAsync。
##### env运行环境
类型：字符串默认值：取决于页面URL，运行环境可以是开发或生产。
例如：less = {env：'production'};
在生产中，您的css缓存在本地存储中，信息消息不会输出到控制台。
如果文档的URL以file：//开头，或者在本地计算机上或具有非标准端口，则它将自动设置为开发。
##### errorReporting
类型：字符串
选项：html | console | function
默认值：html
编译失败时设置错误报告方法。
##### fileAsync
类型：布尔值
默认值：false
是否在具有文件协议的页面中异步请求导入。
函数（不推荐使用 - 使用@plugin）
类型：对象
用户功能，按名称键入。

    less = {
        功能： {
            myfunc：function（）{
                返回less.Dimension（1）;
            }
        }
    };

它可以像原生的Less函数一样使用，例如

    .myclass {
      border-width：unit（myfunc（），px）;
    }
##### logLevel
类型：数字
默认值：2
2 - 信息和错误
1 - 错误
0 - 没什么
javascript控制台中的日志记录量。注意：如果您在生产环境中，则不会进行任何记录。
##### poll
类型：整数
默认值：1000
处于监视模式时轮询之间的时间量（以毫秒为单位）。
##### relativeUrls
类型：布尔值
默认值：false
（可选）将URL调整为相对。如果为false，则URL已经相对于entry-less文件
##### useFileCache
类型：布尔值
默认值：true（之前在v2之前为false）
是否使用每个会话文件缓存。这会缓存较少的文件，以便您可以调用modifyVars，并且它不会再次检索所有较少的文件。如果使用观察程序或使用reload设置为true调用refresh，则在运行之前将清除缓存。

# Less.js选项
## 跨平台选项
##### 包括路径

    lessc --include-path = PATH1; PATH2 {paths：['PATH1'，'PATH2']}

如果@import规则中的文件不存在于该确切位置，则Less将在传递给此选项的位置查找该文件。 例如，您可以使用它在Less文件中指定要引用库的相对路径。
##### Rootpath

    lessc -rp=resources/
    lessc --rootpath=resources/	{ rootpath: 'resources/' }

如果@import规则中的文件不存在于该确切位置，则Less将在传递给此选项的位置查找该文件。 例如，您可以使用它在Less文件中指定要引用库的相对路径。
##### 相对URL
    lessc -ru
    lessc --relative-urls {relativeUrls：true}

默认情况下，URL保持原样，因此如果您在引用图像的子目录中导入文件，则将在css中输出完全相同的URL。此选项允许您在导入的文件中重写URL，以便URL始终相对于基本导入的文件。例如。

    // main.less
    @import“files / backgrounds.less”;
    // files / backgrounds.less
    .icon-1 {
      background-image：url（'images / lamp-post.png'）;
    }

正常输出以下内容

    .icon-1 {
      background-image：url（'images / lamp-post.png'）;
    }

但是使用relativeUrls：true，它将输出

    .icon-1 {
      background-image：url（'files / images / lamp-post.png'）;
    }

您可能还需要考虑使用data-uri函数而不是此选项，它会将图像嵌入到css中。
##### Strict Math
    lessc -sm = on
    lessc --strict-math = on {strictMath：true}

默认为off / false。
如果没有此选项，Less将尝试处理您的CSS中的所有数学，例如在：

    .class {
      grid-column：3/6;
    }
    3/6将导致2。

严格的数学运算，只处理不必要的括号内的数学。例如。

    .class {
      grid-column：3/6;
      height：40px + 2px;
      width：（40px + 2px）;
    }
    .class {
      grid-column：3/6;
      width：40px + 2px;
      width：42px;
    }

我们原计划在未来将此默认为true，但它一直是一个有争议的选项，我们正在考虑我们是否以正确的方式解决了问题，或者Less是否应该只对有效或无效的实例有例外。
##### Strict Units
    lessc -su = on
    lessc --strict-units = on {strictUnits：true}

默认为off / false。
如果没有此选项，则在数学运算时尝试猜测输出单位。例如

    .class {
      property：1px * 2px;
    }

在这种情况下，事情显然是不对的 - 长度乘以长度给出一个区域，但css不支持指定区域。所以我们假设用户意味着其中一个值是一个值，而不是长度单位，我们输出2px。
如果打开严格的单位，我们假设这是计算中的错误并抛出错误。
##### 全局变量
    lessc --global-var =“color1 = red”{globalVars：{color1：'red'}}

此选项定义可由文件引用的变量。变量的声明放在基础Less文件的顶部，这意味着可以使用它，也可以覆盖它。
##### 修改变量
    lessc --modify-var =“color1 = red”{modifyVars：{color1：'red'}}

与全局变量选项相反，这会将声明放在基本文件的末尾，这意味着它将覆盖Less文件中定义的任何内容。
##### 修改变量
    lessc --modify-var =“color1 = red”{modifyVars：{color1：'red'}}

与全局变量选项相反，这会将声明放在基本文件的末尾，这意味着它将覆盖Less文件中定义的任何内容。
##### URL参数
    lessc --url-args =“cache726357”{urlArgs：'cache726357'}

此选项允许您指定要转到每个URL的参数。例如，这可以用于缓存清除。
##### 预装插件
请参阅：预加载插件
##### lint检查
    lessc --lint -l {lint：true}

运行较少的解析器，只报告错误而不输出任何内容。
##### 压缩
    lessc --compress -x {compress：true}

使用较少的内置压缩进行压缩。这项工作做得不错，但没有利用专用css压缩的所有技巧。请随时通过pull请求改进我们的压缩输出。
##### 允许从不安全的HTTPS主机导入
    lessc --insecure {insecure：true}

## 源映射选项
大多数这些选项不适用于在浏览器中使用，因为您应该使用预编译的Less文件生成源映射。
##### 生成源映射
    lessc --source-map {sourceMap：{}}

##### 源映射输出文件名
    lessc --source-map=file.map	{ sourceMap: { outputFilename: 'file.map' } }

##### 源映射Rootpath

lessc --source-map-rootpath=dev-files/	{ sourceMap: { sourceMapRootpath: 'dev-files/' } }

指定添加前导路径到源映射内的每个less文件路径，以及输出css中指定的映射文件的路径.
由于basepath默认为输入较少文件的目录，因此根路径默认为从sourcemap输出文件到input less文件的基目录的路径。
例如，如果您在Web服务器的根目录中生成了css文件，但将源/ less / css / map文件放在不同的文件夹中，请使用此选项

    output.css
    dev-files/output.map
    dev-files/main.less

##### 源映射Basepath

    lessc --source-map-basepath=less-files/	{ sourceMap: { sourceMapBasepath: 'less-files/' } }

这与rootpath选项相反，它指定应从输出路径中删除的路径。 例如，如果要编译less-files目录中的文件，但源文件将在根目录或当前目录中的Web服务器上可用，则可以指定此文件以删除路径中的其他less-files部分。
它默认为输入less文件的路径。

##### 在源映射中包含less源文件
    lessc --source-map-include-source {sourceMap：{outputSourceFiles：true}}

此选项指定将所有Less文件包含在sourcemap中。这意味着您只需要映射文件即可获得原始来源。
这可以与map inline选项一起使用，这样您根本不需要任何其他外部文件。

##### 源映射内联
    lessc --source-map-inline {sourceMap：{sourceMapFileInline：true}}

此选项指定映射文件应在输出内联CSS。这不建议用于生产，但是对于开发，它允许编译器在支持它的浏览器中生成单个输出文件，表现为使用的是编译后的css但显示的却是未编译的less源文件。

##### 源映射URL
    lessc --source-map-url = .. / my-map.json {sourceMap：{sourceMapURL：'.. / my-map.json'}}

允许覆盖css中指向映文件的URL。这适用于rootpath和basepath选项未完全生成所需内容的情况。

# 预加载插件
在Less.js开始解析之前加载插件
虽然使用插件的最简单方法是使用[@plugin-at-rule][2] ，但在Node.js环境中，您可以通过命令行预先加载全局Less.js插件，或者在Less选项中指定它。
## 预处理
如果要添加Less.js预处理器，则需要预加载插件。 也就是说，在解析之前会传递原始Less源并调用插件。 一个例子是Sass-To-Less预处理器插件。
注意：预评估插件不需要预加载（在解析Less源之后，但在评估之前）。
## Node.js使用
使用命令行
如果你使用lessc，你需要做的第一件事是安装该插件。 像NPM一样，我们建议使用“less-plugin-”前缀来注册Less.js插件（以便于搜索），尽管这不是必需的。 因此，对于自定义插件，您可以安装：

    npm install less-plugin-myplugin

要使用该插件，只需编写代码即可在命令行上传递：

    lessc --myplugin

每当有一个未知的Less选项（如“myplugin”）时，Less.js将尝试加载“less-plugin-myplugin”和“myplugin”模块作为插件。
您还可以使用以下命令显式指定插件：

    lessc --plugin = myplugin

要将选项传递给插件，您可以使用以下两种方式之一来编写它。

    lessc --myplugin =“advanced”
    lessc --plugin = myplugin = advanced

## 通过Less.js加载插件

在Node中，require插件时需要将其作为插件数组来传递less插件。 例如。

    var LessPlugin = require('less-plugin-myplugin');
    less.render(myCSS, { plugins: [LessPlugin] })
      .then(
        function(output) { },
        function(error) { }
      );

# 编程使用
编程中只要使用less的less.render函数来实现，比如

    //返回承诺
    less.render(lessInput, options)
        .then(function(output) {
            // output.css = string of css
            // output.map = string of sourcemap
            // output.imports = array of string filenames of the imports referenced
        },
        function(error) {
        });

    // or...
    //回调方法
    less.render(css, options, function(error, output) {})

如上所示，options参数是可选的。如果指定了回调方法，则不会返回承诺，如果您没有指定回调方法则会给出承诺。在编译引擎下，使用回调版本，以便可以同步使用less
如果要渲染文件，首先要将其转化为字符串（传递给less.render），然后将options选项参数上的filename字段设置为主文件的文件名。 less会处理所有的导入。
sourceMap选项是一个对象，可用于设置子源图选项。 可用的子选项包括：sourceMapURL，sourceMapBasepath，sourceMapRootpath，outputSourceFiles和sourceMapFileInline。 请注意，sourceMap选项现在不适用于浏览器编译器中的less.js.

    less.render(lessInput)
        .then(function(output) {
            // output.css = string of css
            // output.map = undefined
    }
    //,
    less.render(lessInput, {sourceMap: {}})
        .then(function(output) {
            // output.css = string of css
            // output.map = string of sourcemap
    }
    //or,
    less.render(lessInput, {sourceMap: {sourceMapFileInline: true}})
        .then(function(output) {
            // output.css = string of css \n /*# sourceMappingURL=data:application/json;base64,eyJ2ZXJ..= */
            // output.map = undefined
    }

## 访问日志
您可以使用以下代码添加日志侦听器

    less.logger.addListener({
        debug: function(msg) {
        },
        info: function(msg) {
        },
        warn: function(msg) {
        },
        error: function(msg) {
        }
    });

注意：所有功能都是可选的。 less本身不会记录错误，而是将错误传递回less.render中的回调或承诺

# 为Less.js做贡献
这部分就描述了，有兴趣的同学可以访问下源文[为Less.js做贡献](https://less.bootcss.com/usage/#programmatic-usage)

[2]: https://less.bootcss.com/features/#plugin-atrules-feature
[1]: https://less.bootcss.com/usage/#less-options


