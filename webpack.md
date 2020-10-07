## 一：入门

## 二：配置

配置文件：webpack.config.js

* entry：配置模块的入口
* output：配置如何输出最终想要的代码
* module：配置处理模块的原则
* resolve：配置寻找模块的规则
* plugins：配置扩展插件
* deserver：配置DevServer
* 其他配置项：其他零散的配置项
* 整体配置结构：整体地描述各配置项的结构
* 多种配置类型：配置文件不止返回一个Object，也可返回54其他形式

## 三：

## 四： 优化

优化从两点：

* 优化开发体验

  * 优化构建速度：
  * 优化使用体验：通过自动化手段完成一些重复的工作。

* 优化输出质量

  本质：优化构建输出的要发布到线上的代码 

  * 减少用户能感知到的加载时间：如首幕加载时间
  * 提升流畅度：即提升代码新

### 4.1 缩小文件的搜索范围

从entry出发，解析文件。根据文件后缀，用对应的loader去处理文件。

#### loader

由于loader对文件的转换很耗时，需要尽可能少的文件被loader处理

````js
module.exports = {
    module: {
        // 配置babel-loader例子
        rules: {
            // 合适的正则
            test: /\.js$/,
            // babel-loader支持缓存转换出的结果，通过cacheDirectory选项开启
            use: ['babel-loader?cacheDirevtory'],
            // 只对项目根目录下的src目录中的文件采用babel-loader
            includes: path.resolve(__dirname,'src'),
        }
    }
}
````

注：__dirname表示项目根目录

#### resolve 

* mainFields: 用于配置第三方模块使用哪个入口文件

  有多个入口文件的原因是，某些模块可以同时用于多个环境。

  mainFields有三个值：['broswer','module','main']

  ````js
  // package.json 入口文件描述
  {
      "browser": "fetch-npm-browserify.js",
      "main": "fetch-npm-node.js",
  }
  ````

* alias：通过别名，将原导入路径映射城一个新的导入路径

* extension:导入文件时，没带后缀，的判断顺序。

  尽量在写导入语句，尽量戴上后缀

* noParse： 可让webpack忽略对部分没采用模块化的递归解析处理

  住：被忽略掉的文件不应该包含import、require、define等模块化语句。不然会导致在构建出的代码中包含无法在浏览器环境下执行的模块化语句。

````js
module.exports = {
    resolve: {
        // 配置在哪里寻找第三方模块，如果没找到，就去上一层
        modules: [path.resolve(dirname.'node_modules')],
        // 只采用main字段作为入口文件的描述字段，以减少搜索步骤
        mainFields: ['main'],
        // 减少耗时的递归解析操作。
        alias: {
            'react': path.resolve(__dirname,'node_modules/react/dise/react.main.js')
        },
        // 尽可能减少后缀尝试的可能性
        extensions: ['js'],
        // 忽略对react.min.js文件的递归解析处理
        noParse: ['/react\.min\.js$'],
    }
}
````

### 4.2 使用DllPligin

#### dll介绍

.dll文件:动态链接库：可包含**其他模块调用的函数和数据**

要给web项目构建接入ddl，得完成：

* 讲网页依赖的基础模块抽离出来，打包到一个个单独的ddl库中。在一个ddl库中可包含多个模块。
* 当需要导入的模块存在于某个ddl库中，这个模块不能被再次打包，而是去ddl中获取。
* 页面依赖的所有ddl都需要被加载

引入ddl能提升构建速度的原因：

包含大量复用模块的ddl库只需被编译一次，在之后的构建中包含的模块将不会重新编译，而是直接使用ddl库中的代码。只要不升级模块的版本（如react、react-dom），dll库就不用重新编译

#### dll接入webpack

webpack已经内置了对动态链接库的支持，通过以下两个内置的插件接入。

* DllPlugin插件：用于打包出一个个单独的动态链接库文件
* DllReferencePlugin插件：用于在主要的配置文件中引入DllPlugin插件打包好的动态链接库文件。

**notending**

### 4.3 使用HappyPack

前因：运行在Node.js之上的webpack是单线程模型。即webpack不能同时处理多个任务。

HappyPack将任务分解给多个子进程去并发执行，子进程处理完后再将结果发送给主进程。

由于JavaScript是单线程模型，所以要想发挥多核CPU的功能，只能通过多进程实现，而无法通过多线程实现。

* 原理

在webpack构建流程中，最耗时的流程可能是Loader对文件的转换操作，且只能线性执行，HappyPack的核心原理就是将这部分分解到多个进程中去并行处理，从而减少总的构建时间。

每通过一个new HappyPack()实例化一个HappyPack，实际就是HappyPack核心调度器如何通过一系列Loader去转换一类文件，且可以指定如何为这类转换操作分配子进程。

核心调度器的逻辑代码在主进程中，即运行着webpack的进程，核心调度器会将一个个任务分配给当前空闲的子进程，子进程处理完后将结果发送给核心调度器，他们之间的数据通过进程间的通信API实现的。

核心调度器收到来自子进程处理完毕的结果后，会通知webpack该文件已处理完毕。

````js
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const HappyPack = require('happypack');

module.exports = {
    module: {
        rules: [
            {
                test: /\.js$/,
                // 将对.js文件的处理转交给id为babel的HappyPack实例
                use: ['happypack/loader?id=babel'],
                // 排除node_modules目录下的文件，node_modules目录下的文件都采用了ES5语法，没必要再通过Babel去转换
                exclude: path.resolve(__dirname,'node_modules'),
            },
            {
                test: /\.css$/,
                use: ExtractTextPlugin.extract({
                    use: ['happypack/loader?id=css'],
                })
            },
        ],
        plugin: [
            new HappyPack({
                // 用唯一标识符id，来表示当前的HappyPack是用来处理一类特定的文件的。
                id: 'babel',
                // 如何处理.js文件，用法和Loader配置中的一样
                loaders: ['babel-loader?cacheDirectory'],
                // 使用共享进程池中的子进程去处理任务,防止占用的资源过多
                threadPool: happyThreadPool,
                // 代表开启几个子进程去处理着一类型的文件
                threads: 3, // 默认三个
                // 是否允许HappyPack输出日志，
                verbose: true, // 默认true
            }),
            new HappyPack({
                id: 'css',
                loaders: ['css-loader'],
            }),
            new ExtractTextPlugin({
                filename: `[name].css`
            }),
        ],
    }
}
````

构建成功后：npm i -D happypack

### 4.4 使用ParallelUglifyPlugin

UglifyJS，构建用于开发环境的代码，但在构建用于线上的代码时会卡在一个时间点迟迟没有反应，其实在这个卡住得时间点正在进行的就是代码压缩。

由于压缩js代码，需要先将代码解析成用Object地抽象表示的AST语法树，再去应用各种规则分析和处理AST，所以导致这个过程的计算量巨大，耗时非常多。

ParallelUglifyPlugin原理：webpack输出和压缩多个js文件时，原本会使用UglifyJS去一个一个压缩子进程去完成，每个子进程通过UglifyJS去压缩代码，于是变成了并行执行。

在通过通过new ParallelUglifyPlugin()实例化时候，支持一下参数

* test：用正则匹配哪些文件需要被压缩，默认/.js$/
* inclues:用正则匹配需要被压缩的文件，默认[]
* exclude:用正则匹配不需要被压缩的文件，默认[]
* cacheDir:缓存压缩后的结果，用于配置缓存存放的目录路径。默认不会缓存，若需开启，则设置一个目录路径。
* workerCount: 开启几个子进程并发执行压缩。默认为当前运行的计算机的CPU核数减1
* sourceMap:是否输出SourceMap，这会导致压缩过程变慢
* uglifyJS: 用于压缩ES5
* uglifyES:用于压缩ES6代码时的配置，不能与uglifyJS一起使用

````js
const path = require('path');
const DefinePlugin = require('webpack/lib/DefinePlugin');
const ParallelUglifyPlugin = require('wbpack-parallel-uglify-plugin');
module.exports = {
    plugins: [
        // 使用ParallelUglifyPlugin并行压缩输出js代码
        new ParallelUglifyPlugin({
            // 传输给UglifyJS的参数
            uglifyJS: {
                output: {
                    // 最紧凑的输出
                	beatify: false,
                    // 删除所有注释
                    comments: false,
            	},
                // 是否保留代码中的注释，默认保留。
                compress: {
                    // 在UglifyJS删除没有用到的代码时不能输出警告
                    warnings: false,
                    // 删除所有ocnsole语句，可兼容IE浏览器
                    drop_console: true,
                    // 内嵌已定义但是只用到一次的变量
                    collapse_vars: true,
                    // 提取出出现多次但是没有定义变量去引用的静态值
                    reduce_vars: true,
                },
            },
            
        }),
    ],
};
````

npm i -D webpack-parallel-uglify-plugin

### 4.5 使用自动刷新

构建过程中，重复的操作可交给代码完成。

* webpack：负责监听文件
* webpack-dev-server: 负责刷新浏览器

在使用webpack-dev-server模块去启动webpack模块时，webpack模块的监听模式默认会被开启。webpack模块会在文件发生变化时通知webpack-dev-server

#### 4.5.1 文件监听

````js
module.export = {
    // 只有在开启监听模式时，watchOptions才有意义。默认false
    watch: true,
    watchOptions: {
        // 不监听的文件或文件夹，支持正则匹配
        ignored: /node_modules/,
        // 监听到变化后等300ms再去执行，节流。默认300ms
        aggregateTimeout: 300,
        // 判断文件是否发生变化是通过不停地询问系统指定文件有没有变化实现的。
        // 默认每秒询问1000次
        poll: 1000
    }
}
````

让webpack开启监听模式，如下两种方式：

* 在配置文件webpacl.config.js设置watch:true
* 启动webpack时执行：webpack --watch

**文件监听原理**

用watchOptions.poll 定时检查文件的最后编辑时间，即每秒检查多少次。当发生变化后，先缓存起来，通过watchOptions.aggregateTimeout设置保存缓存的时间，然后一次性告诉监听者。

默认，webpack会从配置的Entry文件出发，递归解析出Entry文件所依赖的文件，将这些依赖的文件加入监听列表中。

住：由于保存的文件的路径和最后的编辑时间需要占用内存，定时周期检查需要占用CPU以及文件I/O，所以最好减少需要监听的文件数量和降低检查频率

**优化文件监听的性能**

* watchOptions.agregateTimeout的值越大性能越好，这能降低重新构建的频率
* watchOptions.poll的值越小越好，这能降低检查的频率

但两种优化方法的后果是监听模式的反应和灵敏度降低了。

#### 4.5.2 自动刷新浏览器

**自动刷新的原理**

* 借助浏览器扩展去通过浏览器提供的接口刷新，WebStrom IDE的LiveEdit功能就是这么实现的
* 向要开发的网页注入客户端代码，通过代理客户端去刷新整个页面。（DevServer默认）
* 将要开发的网页装进一个iframe中，通过刷新iframe看到最新效果

启动：webpacl-dev-server

当打开http://localhost:8080/后，查看调用的接口能看到，代理客户端向DevServer发起的WebSocket连接。

**优化自动刷新的性能**

### 4.6 开启模块热替换

Hot Module Replacement

启动：webpack-dev-server --hot

not ending 

模块热替换仅用于开发环境

模块热替换的运行依赖在每个Chunk中都包含代理客户端的代码

### 4.7 区分环境

* 开发环境

  包含类型检查，HTML元素检查等针对开发者的警告日志代码

* 线上环境

  去掉了所有针对开发者的代码，只保留正常运行的代码，以优化代码大小和性能。

住：开发环境连接的接口地址可能与线上环境不同，因为要避免在开发过程中造成对线上数据的影响。

当代码中出现了process，webpack会自动打包进process模块的代码以支持非Node.js的运行环境。

````js
const DefinePlugin = require('webpack/lib/DefinePlugin');
module.exports = {
    plugins: [
        new DefinePlugin({
            // 定义NODE_ENV环境变量为production
            'process.env': {
                // 环境变量的值吸引由一个双引号包裹的字符串
                NODE_ENV: JSON.stringify('production')
            }
        }),
    ],
};
````

若想让webpack通过shell脚本定义的环境变量，则可使用EnviromentPlugin

````js
new Webpack.EnviromentPligin(['NODE_ENV']);
new webpack.DefinePlugin({
    'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV);
})
````

### 4.8 压缩代码

浏览器通过服务器访问网页时获取js、css资源都是文本形式的，文件越大，网页加载的时间越长。为了提升网页加载速度和减少网络传输流量，可对这些资源进行压缩。可通过GZIP算法对文件进行压缩，也可对文件本身进行压缩（也有混淆源码的作用）。

#### 压缩js

通过插件形式在webpack中接入UglifyJS

* UglifyJsPlugin:通过封装UGlifyJS实现压缩

  可直接通过 webpack --optimize-minimize 启动

* ParallelUglifyPlugin:多进程并行处理压缩

#### 压缩ES6

开发过程中，尽量使用ES6。原因：代码量更少，js引擎对ES6中的语法做了性能优化。

webpack接入UglifyES时，不能使用内置的UglifyJsPlugin，而是需要单独安装和使用uglify-webpack-plugin

````js
// npm i -D uglifyjs-webpack-plugin@beta
const UglifyESPlugin = require('uglify-webpack-plugin');
module.exports = {
    plugins: [
        new UglifyESPlugin({
            // 比UglifyEJSPlugin 多嵌套了一层
            uglifyOptions: {
                compress: {
                    warning: false,
                    drop_console: true,
                    collapse_vars: true,
                    reduce_vars: true,
                },
                output: {
                    beautify: false,
                    comments: false,
                }
            }
        })
    ]
}
````

同时，为了不让babel-loader输出ES5语法的代码，需要去掉.babelrc配置文件中的babel-preset-env,但要保留其他Babel插件，如babe;-preset-react,因为babe;-preset-env负责将ES6代码转换成ES5.？

#### 压缩CSS

常用的压缩工具cassnano，基于PostCSS

只需开启css-loader的minimize选项。

````js
const path = require('path');
const {WebPlugin} = require('web-webpack-plugin');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
module.exports = {
    modules: {
        rules: [
            {
                test: /\.css/, // 增加对CSS文件的支持
                // 提取Chunk中的css代码到单独的文件中
                use: ExtractTextPlugin.extract({
                    // 通过minimize选项压缩CSS代码
                    use: ['css-loader?minimize'],
                }),
            },
        ],
    },
    plugins: {
        // 用webplugin生成对应的HTML文件
        new WebPlugin({
            template: './template.html', // HTML模板所在的文件路径
            filename: 'index.html', // 输出的HTML的文件名称
        }),
        new ExtractTextPlugin({
            filename: `[name]_[contenthase:8].css` // 为输出的css文件名称加上hash值
        }),
    },
},
````

### 4.9 CDN加速

虽然代码压缩减少了网络传输的大小，但首次打开时加载慢。

CDN（内容分发网络）：作用是加速网络传输，通过将资源部署到世界各地，使用户在访问时按照就近原则从离其最近的服务器获取资源，来加快资源的获取速度。

CDN实际是优化物理链路层传输过程中的光速有限，丢包等问题来提升网速

CDN服务可看成速度更快的HTTP服务

-------

为网站接入CDN，需要将网页的静态资源上传到CDN服务上。由于CDN服务一般会为资源开启很长时间的缓存，这可能导致新的发布不能立即生效。

解决方式：

* 针对HTML文件：不开启缓存，将HTML放到自己的服务器上且关闭自己服务器上的缓存，而不是CDN服务上。自己服务器只提供HTML文件和数据接口

* 针对静态的js、css、图片等文件，开启CDN和缓存，上传到CDN服务上，同时为每个文件名带上由文件内容算出的Hash值。

  带上has值的原因是文件名会随着文件的内容而变化，只要文件的内容发生变化，其对应的URL就会变化，它会被重新下载，无论缓存时间有多长。

----

浏览器有个规则：在同一时刻针对同一个域名的资源的并行请求有限制（4个左右）。为避免加载资源堵塞，将这些静态资源分散到不同的CDN服务上。

多域名会增加域名解析时间，可增加预解析域名，以减少域名解析带来的延迟

````html
<head>
	<link rel="dns-prefetch" href="//js.cdn.com">
</head>
````

---------

为webpack实现cdn的接入

* 静态资源的文件名带上hash，以防止缓存，
* 将不同类型的资源放到不同域名的cdn服务上，防止资源的并行加载被阻塞
* 静态资源的导入URL地址改成指向CDN服务的绝对路径的URL

````js
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const {WebPlugin} = require('web-webpack-plugin');

module.exports = {
    // 忽略entry配置
    output: {
        // 为输出的js文件加上hash值
        filename: '[name]_[chunkhash:8].js',
        path: path.resolve(__dirname,'./dist'),
        // 指定存放js文件的CDN目录URL
        publicPath: '//js.cdn.com/id/',
    },
    module: {
        rules: [
            {
                // 增加对CSS文件的支持
                test: /\.css/,
                use: ExtractTextPlugin.extract({
                    // 压缩CSS代码
                    use: ['css-loader?minimize'],
                    // 指定存放css中导入的资源（例如图片）的CDN目录URL
                    publicPath: '//img.cdn.com/id'
                }),
            },
            {
                // 增加对PNG文件的支持
                test: /\.png/,
                // 为输出的PNG文件名加上Hash值
                use: ['file-loader?name=[name]_[hash:8].[ext]'],
            },
            // 省略其他loader配置
        ],
    },
    plugins: [
        // 使用webplugin自动生成html
        new WebPlugin({
            // html模板文件所在的文件路径
            template: './template.html',
            // 输出html的文件名
            filename: 'index.html',
            // 指定存放的CSS文件的CDN目录URL
            stylePublicPath: '//css.cdn.com/id/'.
        }),
        new ExtractTextPlugin({
           	// 为输出的css文件名加上hash值
            filename: `[name]_[contenthash:8].css]`,
        }),
        // 省略代码压缩插件配置
    ]
}
````

核心：通过publicPath参数设置存放静态资源的CDN目录URL。为了让不同类型的资源输出到不同的cdn：

* 在output.publlicPath中设置js地址
* 在css-loader.publicPath中设置被css导入的资源的地址
* 在webplugin.stylePublicPath中设置css文件的地址

### 4.10 使用Tree Shaking

用于剔除js中用不上的死代码

````js
// 在.babelrc文件中修改
{
    "presets": [
        [
            "env",
            {
                // 关闭Babel模块的转换功能，保留原本的ES6模块化语法。
                "modules": false
            }
        ]
    ]
}
````

启动：webpack --display-used-exports,以方便追踪tree shaking

### 4.11 提取公共代码

#### 提取公共代码原因

#### 如何提取公共代码

### 通过webpack提取公共代码

###  4.12 分割代码以按需加载

因为网页要承载的功能越来也多，一次性加载所有功能对应的代码，这会导致网页加载缓慢、交互卡顿、使用户体验非常糟糕。

如何按需加载：

* 将整个网站划分成一个个小功能，再按照每个功能的相关程度将它们分成几类
* 将每一类合并为一个Chunk，按需加载对应的Chunk
* 

### 4.13 使用Prepack

### 4.14 开启Scope Hoisting

### 4.15 输出分析

## 五：原理



