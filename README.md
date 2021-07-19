Webpack学习笔记

简介
- webpack工具的本质：前端资源构建工具、静态模块打包器
- 构建工具诞生的背景：
  - 将高级的语法转化为浏览器能够识别的语法：比如当html文件中直接引入scss文件、包含ES6语法的js文件时，浏览器并不能识别这些语法，此时就需要通过构建工具将其转换为浏览器可以识别的语法。
- 静态模块打包器的工作原理：
  - 静态模块指的是所有静态资源文件都会作为模块被处理
  - 代码全部引进来后，就会形成一个chunk代码块
  - 然后根据模块的依赖关系进行静态分析，将chunk代码块按不同的资源进行不同的处理，编译为浏览器可以识别的代码。这个处理的过程就叫打包。
  - 打包之后会输出一个文件名为bundle的文件

五个核心概念：
- Entry 入口
指示webpack会以哪个文件为入口进行打包，在打包之前会先分析构建内部的依赖图
- Output 出口
指定打包后的bundle文件输出到那个位置 
- Loader 翻译家
用于处理非js文件（webpack本身只能理解js和json），例如将css, img等资源转换为webpack可以识别的东西？？
- Plugins 插件
可以用于执行功能更大的任务，比如：打包优化、压缩、定义环境变量
- Mode模式
  - development 开发者模式：用于本地调试运行
  - Production 生产者模式：优化代码并上线

安装
- npm i webpack webpack-cli -g    
-  webpack-cli 工具包可以通过webpack指令使用webpack的功能


手动打包
webpack ./src/index.js -o ./build/build.js   --mode=development/production

- production模式下会优化代码：
  - 会压缩代码 
  - 在webpack5中会以最终代码的形式优化在输出文件中
- 两种模式都只能打包js和json文件，对于css等文件不能直接处理


配置文件
- webpack.config.js
  - 用途：指示webpack干哪些活
  - 内部采用的是commonjs的模块化语法，因为所有的构建工具都是基于nodejs平台运行的。（注意：项目中用的是ES6的语法，与配置文件中的commonjs语法是不同的）
- 基本的结构：
// resolve可以用于拼接绝对路径
const { resolve } = require('path')
module.exports = {
    // 入口文件
    entry: "./src/index.js",
    output: {
        // 输出文件名
        filename: 'bulid.js',
        // 输出路径
        // __dirname是nodejs的一个变量，代表当前文件所在目录的绝对路径
        path: resolve(__dirname, 'build')
    },
    // loader的配置
    module: {
        rules: [

        ]
    },
    // plugins插件的配置
    plugins: [

    ],
    // 模式
    mode: 'development'
    // mode:'production'
}

- 配置好之后，可以直接通过webpack命令，基于该配置文件，对整个项目进行打包编译


搭建开发环境
打包资源
打包样式文件
- 为什么需要打包样式文件？
  - 因为webpack没办法直接处理非js/json的文件，所以需要通过loader进行处理
- 用什么工具实现？
  - 使用一系列的loader实现
  - loader使用的步骤：1.下载 2.配置
    module: {
        rules: [
            // 每一种处理都在一个对象中配置，这里用于处理css文件
            {
                test: /\.css$/,//通过正则表达式，匹配所有的css文件  ❤️❤️ 踩坑：不小心加上引号的话，比如test="/\.css$/"，则是错的
                // 注意：use中的loader执行的顺序是：从右到左，从下到上
                use: [
                    // style-loader的作用：创建style标签，将css-loader处理后生成的样式资源插入style标签中，然后将style标签添加在<head>中生效
                    'style-loader',
                    // css-loader的作用：将css文件以commonjs模块的形式加载到js文件中，js文件中此时的css代码为样式字符串
                    'css-loader'
                ]
            },
            // 处理less文件 
            {
                test: /\.less$/,
                use: [
                    'style-loader',
                    'css-loader',
                    // less-loader用于将less文件编译为css文件
                    // 注意：下载的时候需要下两个包：less-loader less
                    'less-loader'
                ]
            }
        ]
    },


打包html资源
- 为什么需要打包html资源
  - 为了实现自动创建html文件，并将打包后的所有资源文件引入其中
- 用什么工具实现？
  - 使用html-webpack-plugin插件
  - plugins使用的步骤：1.下载 2.引入 3.配置
const HtmlWebpackPlugin = require('html-webpack-plugin')


// ❤️注意是plugins，不是pulgins
plugins: [
    new HtmlWebpackPlugin({
        template: './src/index.html'
    })
],



打包图片资源
- 涉及到url-loader、file-loder、html-loader
    module: {
        rules: [
            {
                test: /\.less$/,
                use: [
                    'style-loader',
                    'css-loader',
                    'less-loader'
                ]
            },
            {
                test: /\.(png|jpg|gif)$/,
                // 1.如果需要使用的loader超过1个，则必须使用use属性，如果只有一个，则可以直接使用loader属性
                loader: 'url-loader',
                // 2.注意：需要下载的包有两个：url-loader, file-loader (url-loader依赖于file-loader)
                // 3.通过options属性可以进行自定义的配置，相当于向这个loader传递了某些参数？
                options: {
                    // 4.若图片文件小于等于20kb，则会被base64处理
                    // base64处理的优缺点：
                    // 优点：base64会将图片转换为代码字符串 可以减少请求的数量，（从而减轻服务器的压力）
                    // 缺点：图片的体积会更大，（所以会导致文件请求速度变慢）
                    // 一般10kb以下的文件可以通过base64来处理
                    limit: 20 * 1024,
                    // 7.通过name属性可以给打包后的图片进行重命名
                    name: 'liuxiaokang[hash:10].[ext]',//取图片原hash值的前10位，[ext]表示取默认的文件扩展名
                    //可以指定输出的文件夹名称
                    outputPath: 'images'
                }
            },
            // 5.问题：url-loader默认可以处理less文件中引入的图片文件，但是处理不了在html中通过img标签的src引入的图片文件
            {
                test: /\.html/,
                // html-loader可以将html中img标签的图片引入进来，进而被url-loader处理(注意：html-loader是专门用来处理图片的)
                // 6.但是由于url-loader内部在处理的时候使用的是es6模块化解析，而html-loader内部使用的是commonjs，
                // 所以为了兼容两者，需要在配置项中将esModule设置为false
                loader: 'html-loader',
                options: {
                    esModule: false
                }
            }
        ]
    },


打包其他资源
比如打包字体图标文件：直接使用file-loader处理
html文件中引入：
<span class="iconfont icon-xianxingxinxi"></span>

index.js文件中引入字体图标样式：
import './iconfont/iconfont.css'

webpack配置文件进行配置：
module: {
    rules: [
        {
            test: /\.css$/,
            use: [
            //踩坑：注意！是style-loader，不是style.loader！！！！
                'style-loader',
                'css-loader'
            ]
        },
        {
        //排除css,js,html等资源
        // ❤️踩坑：这里exclude的内容必须比较全面，比如下方如果不排除png|jpg|gif这些文件，则处理图片的loader最终会变为file-loader，而不是url-loader。
            exclude: /\.(css|js|html)$/,
            loader: 'file-loader',
        }
    ]
},


devServer 启动热部署
module.exports={
    // 配置devServer可以用来实现热部署：自动编译、自动打开浏览器、自动刷新页面
    // 特点：只会在内存中编译打包，不会有任何输出【意思是如果我们把build文件夹删了，再执行一次热部署，此时是不会在build文件夹看到编译的产物的，因为热部署是在内存中打包到build文件夹下的，相对的如果直接使用webpack命令则会将打包后的产物放在build文件夹下】
    // 安装：npm i webpack-dev-server -D
    devServer: {
        // 指定热部署的路径，注意：这里是构建后的路径
        contentBase: resolve(__dirname, 'build'),
        // 启动gzip压缩
        compress: true,
        // 运行的端口号
        port: 3000,
        // 默认打开浏览器
        open: true
    }
    // ❤️❤️ 注意：在webpack5中必须配置target属性为web才能让热部署生效
    target: 'web'
    // 启动热部署的命令：
    // webpack4: npx webpack-dev-Server
    // webpack5: npx webpack server 或 webpack server
}


搭建生产环境
- 搭建生产环境的目的：使项目运行更快、更平稳（兼容性更好）

css相关处理
提取css成单独的文件
- 为什么需要提取成单独的文件？
  - 因为原来所有的css代码都被编译在了一个js文件中，css代码会导致js的体积变大，在解析渲染的时候，js的加载时间就会特别长，所以可能导致页面白屏，无法即使的看到页面。
  - 而使用插件将css文件提取出来，可以减小js的体积，从而可以加快页面的响应速度。
//引入mini-css-extract-plugin插件
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

module:{
    rules:[
            {
                test: /\.css$/,
                use: [
                    //'style-loader',
                    //使用该插件的loader代替style-loader，相当于换了一种css样式的引用方式
                    //style-loader是在html中创建一个style标签，然后将js文件中的css字符串引入进去。
                    //而MiniCssExtractPlugin.loader会将js文件中的css单独提取成一个文件，然后在html中直接通过link标签引入这个css文件
                    MiniCssExtractPlugin.loader,
                    'css-loader'
                ],
            },
    ]
}
plugins: [
    Ïnew MiniCssExtractPlugin({
        // 指定最终打包后的css文件存储的位置
        filename: 'css/build.css'
    })
]


css兼容性处理
- 目的：解放程序员的双手，让工具去帮我们解决浏览器的兼容性问题
- 处理css的兼容性需要使用到postcss库
-  npm i postcss-loader postcss-preset-env -D
package.json文件:

//browserslist的配置可以在github上搜索
"browserslist": {
    "development": [
    //兼容最新一版的谷歌浏览器
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ],
    "production": [
      // 兼容99.9%以上的浏览器
      ">0.1%",
      // 兼容还存在的浏览器,
      "not dead"
    ]
}
  
webpack.config.js文件：
// 默认情况下会去找package.json文件中browserslist属性里production生产环境指定的浏览器，如果想设置为开发环境，需要在文件开头设置环境变量为development: process.env.NODE_ENV = 'development'
process.env.NODE_ENV = 'development'

module.exports={
    module:{
        rules:[
           {
                test: /\.css$/,
                use: [
                    'style-loader',
                    'css-loader',
                    {
                        loader: 'postcss-loader',
                        options: {
                            postcssOptions: {
                            // postcss-preset-env插件的作用：
                            // 它会去package.json中找browserslist里面的配置，通过配置确认兼容的浏览器，从而加载指定的css来兼容样式
                                plugins: [
                                    [
                                        'postcss-preset-env'
                                    ]
                                ]
                            }
                        }
                    }
                ],
            },
        ]
    }
}
  


压缩css
- 目的：较小打包后的体积，提升样式加载的速度
- npm i optimize-css-assets-webpack-plugin -D
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin')


plugins:[
    new OptimizeCssAssetsWebpackPlugin()
]


js相关处理
js语法检查eslint
- 语法检查的目的：
  - 1. 统一团队的代码风格
  - 2.检查代码的错误
- npm i eslint-loader eslint -D 
- Airbnb  
  - 比较推荐的js代码编写风格：Airbnb https://github.com/lin-123/javascript
  - eslint-config-airbnb 可以作为扩展性的配置，让airbnb代码风格的规范在eslint中生效
  - eslint-config-airbnb 与 eslint-config-airbnb-base的区别：前者还包括了对react的定制化代码规范
  - npm i eslint-config-airbnb-base eslint eslint-plugin-import -D
- eslint-loader已经废弃，现在使用的是eslint-webpack-plugin插件
const ESLintPlugin = require('eslint-webpack-plugin')

plugins:[
    new ESLintPlugin()
]

js的兼容性处理
- 目的：默认情况下，如果js文件中使用的是es6的语法，则打包后产物也是es6的语法。而部分浏览器是不能识别es6的语法的，只能识别es5，所以就需要通过babel将es6的语法进行转换后打包。

- 1.基本的js兼容性处理：
  - npm i babel-loader @babel/core @babel/preset-env -D
  - 但@babel/preset-env只能兼容部分的es6语法，例如promise等高级语法就不能处理
module:{
    rules:[
            {
                test: /\.js$/,
                exclude: /node_modules/,
                loader: 'babel-loader',
                options: {
                    // 通过预设指示babel做怎样的兼容性处理
                    presets: ['@babel/preset-env']
                }
            }
    ]
}


- 2.全部的js兼容处理
  - npm i @babel/polyfill -D
  - 存在的问题：由于引入所有的兼容性代码，所以导致js打包后的体积太大
直接在index.js文件中手动引入 import '@babel/polyfill'
//原理：会引入各种兼容es6语法的方法
//后果：index.js打包后的文件会有很多用于兼容的方法，因此体积会变大
import '@babel/polyfill'


- 3.按需加载 
  - 使用core-js 进行按需加载兼容方法，可以减少打包后的体积大小
  - npm i babel-loader @babel/core @babel/preset-env  core-js -D
module:{
    rules:[
            {
                test: /\.js$/,
                exclude: /node_modules/,
                loader: 'babel-loader',
                options: {
                    presets: [
                        // ❤️注意：这里还有一个中括号
                        [
                            '@babel/preset-env',
                            {
                                // 按需加载
                                useBuiltIns: 'usage',
                                //指定core-js的版本
                                corejs: {
                                    version: 3
                                },
                                // 指定兼容到浏览器的哪些版本
                                targets: {
                                    chrome: '60',
                                    firefox: '60',
                                    ie: '9',
                                    safari: '10',
                                    edge: '17'
                                }
                            }
                        ]
                    ]
                }
            }
    ]
}


html和js压缩
- webpack4: 
  - js压缩：只需要将mode设置为production就行
  - html压缩：需要在HtmlWebpackPlugin插件中配置minify属性
plugins: [
    new HtmlWebpackPlugin({
        template: './src/index.html',
        minify: {
            // 移除空格
            collapseWhitespace: true,
            // 移除注释
            removeComments: true
        }
    }),
],
mode: 'production'

- webpack5:
  - 将mode设置为production后，js、css、html都会被自动压缩
 mode: 'production'


❤️ 总结：
- less/css 文件的完整处理过程：
  - 1.通过less-loader编译为css 
  - 2.通过post-css做css代码的兼容
  - 3.通过css-loader将css代码编译到js文件中
  - 4.通过style-loader将css代码插入在html标签中
  - 4.或通过MiniCssExtractPlugin插件和MiniCssExtractPlugin.loader将css文件提取出来，以link标签的形式引入在index.html中
  - 5.通过OptimizeCssAssetsWebpackPlugin插件进行代码的压缩
- js文件的完整处理过程
  - 1.通过eslint-webpack-plugin插件进行代码格式的检验
  - 2.通过babel进行兼容性处理
    - 注意⚠️：eslint应在babel之前进行处理。如果babel先对js代码进行了转换，那可能在通过eslint检查的时候会报错。所以应该先通过eslint进行语法检查，然后再通过babel做语法兼容。
  - 3.通过mode="production"对代码进行压缩
- 图片文件的完整处理过程：
  - 1.对于在css 中通过background-image引入的图片，需要使用url-loader进行处理
  - 2.对于在html中直接引入的img，直接使用html-loader处理
- html文件的完整处理过程：
  - 通过HtmlWebpackPlugin插件实现：
    - 引入index.html模版
    - 打包
    - 代码压缩
- 其他文件的处理：
  - 使用file-loader进行处理


优化配置
介绍
- 开发环境的性能优化：
  - 优化打包构建速度
  - 优化代码调试，让代码调试更加方便 sourcemap
- 生产环境的性能优化：
  - 优化打包构建的速度
  - 优化代码运行的性能
 
【开发环境】优化打包构建速度
- 背景
  - 默认情况下，即使只修改一个文件，也会导致所有的文件被重新打包。
- HMR：
  - hot module replacement 热模块替换
  - 作用：当一个模块发生变化的时候，只会重新打包这一个模块（而不是所有的模块）
  - 可以极大的提升打包构建的速度
  - ❤️❤️ ps：热部署/热更新 与 热模块替换 是不同的概念，热部署是指在保存后会自动的打包，而热模块替换指的是只打包代码发生改变的模块

对于样式文件
- style-loader内部实现了HMR的功能
devServer: {
    contentBase: resolve(__dirname, 'build'),
    compress: true,
    port: 3000,
    open: true,
    // 开启HMR
    hot: true
},

- 效果：

对于html文件：
- 默认没有HMR功能，但html文件也不需要开启HMR功能，因为单页面应用的话你是一般不会去修改html文件的代码的。
- 但是：修改html文件是不会启动热部署的，如何解决？⬇️
实现修改html文件触发热部署的方法:
entry:['./src/index.js','.src/index.html']

对于js文件：
- 默认没有HMR的功能
- 开启HMR功能：
在入口文件index.js文件中进行配置：
if (module.hot) {
    // 一旦module.hot为true，则说明开启了HMR功能
    // 通过accept方法可以监听指定文件的变化，若发生了变化，则其他模块就不会进行重新打包构建
    module.hot.accept('./test.js')
}

- 效果：


【开发环境】优化代码调试
source-map的概念
  - 这是一种将 源代码 与 构建后的代码 形成 一种映射关系的技术
  - 源代码往往是由很多模块组成，但是打包构建后的代码只有一个。若程序出现了错误，通过构建后的代码是没办法定位错误的原因的。
  - 而source-map技术可以通过映射追踪到源代码出错的位置。
- 开启source-map：
在webpack.config.js文件中配置：
module.exports={
    devtool: 'source-map'
}

- 最终会在打包后的build.js的同级目录下生成一个build.js.map文件，这个文件就是用来实现映射追踪的。
开启source-map的类型：
  - [inline-|hidden-|eval-] [nosources-][cheap-[module-]] source-map   三种可以任何组合
  - source-map
    - 内联
    - 错误信息+源代码错误位置
  - inline-source-map
    - 内联：没有生成外部的build.js.map文件，而是将所有的文件生成一整个source-map，然后将其放在了build.js的最后
    - 有错误原因+源代码错误位置
  - hidden-source-map
    - 外联：在build.js的同级目录下生成了build.js.map文件，用于存放source-map
    - 有错误原因，但是给出的代码错误定位是打包编译后的代码中的位置
  - eval-source-map
    - 内联：没有生成外部的build.js.map文件，而是为每一个文件生成一个source-map，并将其分别加载到build.js文件的不同位置处
    - 有错误原因+源代码错误位置
  - nosources-source-map
    - 外联
    - 有错误原因，有错误定位的路径，但是没有源代码
  - cheap-source-map
    - 外联
    - 提示的错误定位只能精确到行，不能精确到列
  - cheap-module-source-map
    - 外联
    - 会将loader的source-map也加进来
    - 提供的错误定位可以精确到列
  - 总结：
    - hidden和nosources是为了在 提供错误定位 的同时隐藏公司的源代码，为了安全。
使用场景
- 开发环境
  - 考虑的因素：速度快、调试更友好
  - 速度：
    - 内联>外联  
    - eval>inline>其他  
    - 但是比eval-source-map更快的是eval-cheap-source-map，因为cheap只会把定位精确到行，所以更快
  - 调试更友好：
    - source-map > cheap-module-source-map > cheap-souce-map
  - 综上：最好的是eval-source-map
- 开发环境
  - 考虑的因素：源代码要不要隐藏、调试要不要友好
  - 由于内联会让build.js的体积变大，所以内联的source-map都舍弃
  - 隐藏源代码：
    - nosources-source-map 隐藏源代码+构建后的代码
    - hidden-source-map 隐藏源代码
  - 调试：source-map、cheap-module-source-map


【生产环境】优化打包构建速度
利用oneOf提升构建速度
- 核心：让一个文件只匹配一个loader，减少匹配的时间，加快打包构建的速度。
module:{
    rules:[
        {
            这里单独提取出eslint的loader
        }，
        {
        通过oneOf属性可以实现单个文件只匹配其中一个loader，若匹配到了，则不会去匹配其他loader,这样可以提升打包构建的速度。
        对于一个文件需要多个loader处理的情况，比如js文件被eslint和babel处理的情况，可以将eslint单独提取出来放在最前面，此时也表明了所有文件都需要先通过eslint处理。
            oneOf:[
               css-loader等配置
            ]
        }
    ]
}


缓存
babel缓存
- 背景：
  - HDM可以解决开发环境下一个文件更新，导致所有文件被重新构建的问题。从而加快打包速度。
  - 但是对于生产环境而言，由于不存在HDM的概念（HDM是源自于devServer的），所以在某个文件更新后，其他所有文件也会被打包。
- 在生产环境下开发的时候，如何实现修改一个文件，其他文件不会被重新打包？
  - 使用babel缓存
  - 原理：babel会在项目第一次构建的时候将编译后的文件缓存起来，再第二次重新进行打包构建的时候，对于没有修改的文件，就会直接去缓存取数据。
module:{
    rules:[
            {
                test: /\.js$/,
                exclude: /node_modules/,
                loader: 'babel-loader',
                options: {
                    // 通过预设指示babel做怎样的兼容性处理
                    presets: ['@babel/preset-env'],
                    要点：开启babel缓存
                    cacheDirectory:true
                }
            }
    ]
}

文件资源缓存
- 背景：
  - babel缓存存在的问题：当项目更新时，由于打包后的文件名没变，此时由于用户端也缓存了打包后的文件，而此时需要请求的文件名没有发生变化，则会直接使用缓存的数据。导致项目没有更新
- 解决方案：
  - 使用hash｜chunkhash｜contenthash 让缓存更好使。

hash
- 如何让用户在项目更新后使用新的打包后的资源？
  - 使用hash对打包构建的文件名进行处理，使得每次重新打包时打包文件名都不同，这样用户在访问打包资源的时候，发现缓存中没有该文件名的文件，就会直接使用新获取的资源。
module.exports={
    output:{
        filename:'js/build.[hash:10].js',
        path:resolve(__dirname,'build')
    },
    plugins:[
        new MiniCssExtractPlugin({
            filename:'css/built/[hash:10].css'
        })
    ]
}

chunkhash
- hash缓存存在的问题：在webpack.config.js配置文件中，由于打包后的js和css文件名都是用的同一个hash值，则当js文件修改的同时，由于hash值的改变，会导致打包后的css文件名也发生改变。从而导致用户访问的css资源的缓存也失效。
- 解决方法：chunkhash
  - 如果打包来源于同一个chunk，则生成的hash值都是一样的。而不同的chunk会生成不同的chunk。【使用了MiniCssExtractPlugin 的css属于一个构建打包的chunk，js属于另一个构建打包的chunk。它们是分开打包的】
  - 这里要保证js和css属于不同的chunk，如果使用的是style-loader，则css也会被归类为js一个chunk。
module.exports={
    output:{
        filename:'js/build.[chunkhash:10].js',
        path:resolve(__dirname,'build')
    },
    plugins:[
        filename:'css/built/[chunkhash:10].css'
    ]
}


contenthash
- 根据文件的内容生成hash值，不同的文件的hash是不同的。
module.exports={
    output:{
        filename:'js/build.[contenthash:10].js',
        path:resolve(__dirname,'build')
    },
    plugins:[
        filename:'css/built/[contenthash:10].css'
    ]
}


我的理解
- 我的理解是babel缓存本来是面向开发者的，但是也影响了用户端，有利有弊，然后可以通过hash来解决弊端（项目更新后不走缓存的问题）


【生产环境】优化代码运行的性能
tree shaking
- 概念：打包时会自动的将没有使用的代码过滤掉，从而减少打包的体积。就类似于将一颗树tree上多余的叶子给摇掉shaking。
- 场景：
  - 1.项目中会引用大量的第三方库，而在这些库中，有的依赖我们是没有使用的，tree shaking可以在打包的时候将这些去除掉。
  - 2.开发中比如一些函数定义了没使用的代码，tree shaking会自动的将其过滤掉
- tree shaking会自动的开启，但是有两个前提：
  - 必须使用ES6模块化
  - 开启produciton环境 
- 需要注意的点：
  - 对于通过import引入的css这种，tree shaking时可能（由于webpack版本的问题）会将其过滤掉，在打包后如果发现没有css文件生成，那可能是tree shaking 的缘故【深表怀疑....】
  - 解决方法：在package.json中设置sideEffects为["*.css"]

code split 代码分割

通过多入口的方式来拆分文件
module.exports = {
    // 当提供多个入口的时候，对于每一个入口，都会单独的输出一个bundle，默认的bundle名称就是entry对象中各个属性的名称，其实就是该属性就代表了一个bundle
    entry: {
        index: './src/index.js',
        test1: './src/test1.js',
        test222: './src/test2.js'
    },
    output: {
        // name取的就是每个bundle默认的名称，即上面entry中的各个属性名
        filename: '[name].contenthash.[ext]',
        path: resolve(__dirname, 'build')
    },
}

- 实际应用：
  - 对于单页面应用，就是单入口。对于多页面应用，就是需要使用多入口。

打包第三方库到单独的chunk
module.exports={
    // optimization可以将使用到的node_modules中的代码单独打包成一个chunk输出
    // 且若多入口的chunk中引入了相同的第三方库，则该库只会被打包一次输出到这个chunk中
    optimization: {
        // 内部使用的是splitChunks这个插件
        splitChunks: {
            chunks: 'all'
        }
    },
}


打包自己的js文件到单独的chunk
在引入其他js文件的时候使用import动态语法，就能将该文件单独打包。
import('./test.js')
import('./test1.js').then(data => {
    // data中包含了该文件导出的变量
})
上述操作会单独打包出两个chunk文件，即js文件



webpack4中打包出的文件名默认是一串数字，可以通过webpackChunkName指定chunk名的前缀：
import(/* webpackChunkName: 'test' */'./test.js')
webpack5中自动将前缀设置为了路径+文件名



懒加载和预加载

webpack的懒加载：
- 触发某个条件的时候，再去加载引入对应的代码 ，可以减少初始加载页面时的响应时间
document.getElementById('btn').onClick = function(){
    通过import动态引入实现懒加载，这里也可以发现，懒加载会进行分块操作
    import('./test.js').then((add)=>{
        console.log(add(1,2));
    })
    当再次触发这个事件的时候，不会重复去加载资源，而是从缓存中取数据
}


预加载：
- 懒加载存在的问题：如果动态引入的文件过大，势必会影响用户的体验。
- 解决方法：通过webpackPrefetch实现预加载，原理是在等其他资源加载完毕后，等浏览器空闲了，再去加载该资源。该种方式与直接引入该文件的不同之处在于，直接引入会是同步加载的方式，同步加载会导致加载速度变慢。
document.getElementById('btn').onClick = function(){
    import(/* webpackPrefetch: true */'./test.js').then((add)=>{
        console.log(add(1,2));
    })
}

- webpackPrefetch兼容性不太好，移动端慎用


PWA
- 概念：
  - 渐进式网络开发应用程序
  - 让web应用具有离线访问的能力，即当网络中断时，刷新当前页面，仍能查看部分数据。
- 核心：workbox  npm i workbox-webpack-plugin -D

1.在webpack.config.js中引入workbox插件：
const WorkboxWebpackPlugin = require('workbox-webpack-plugin')
plugins: [
    // 该插件默认会打包生成一个serviceWork配置文件
    new WorkboxWebpackPlugin.GenerateSW({
        clientsClaim: true,//用于帮助serviceWork快速启动
        skipWaiting: true,//删除旧的serviceWork
    })
],


2.在入口文件中注册serviceWorker:
// 处理兼容性问题，需要先判断一下
if ('serviceworker' in navigator) {
    // 在资源加载完毕后注册serviceworker
    window.addEventListener('load', () => {
        navigator.serviceWorker.register('/service-worker.js')
            .then(() => {
                console.log('sw注册完毕');
            }).catch(() => {
                console.log('注册失败');
            })
    })
}

注意：serviceWorker必须运行在服务器上，所以在开发的过程中如果想进行检验测试，则需要创建一个node服务器，去加载build后的资源。

3.快速搭建一个node服务器：
npm i serve -g
serve -s build // 启动一个服务器，将build目录下的所有资源作为静态资源暴露出去


4.测试：
访问serve -s build后显示的node服务器的访问地址，然后将浏览器中network选项设置为offline，此时刷新数据会发现页面仍能展示，此时的数据就是从serviceWorker所缓存的数据中取得的。



多进程打包
- 通过多进程打包加快打包速度。
- 需要使用到thread-loader，该loader主要是通过开启多进程来加速其他loader的打包速度。主要针对的是对babel-loader的加速。也可以放在其他loader的前面。
- 注意：若babel-loader执行的工作耗时比较短，则使用thread-loader可能反而会增加打包的时间。因为进程启动大概需要600ms，而且进程通信也有开销时间。所以工作消耗时间比较长的项目才推荐多进程打包。
module: {
    rules: [
        {
            test: /\.js$/,
            exclude: /node_modules/,
            use: [
                // 要点：
                {
                    loader: 'thread-loader',
                    options: {
                        // 可以手动设置开启的进程数
                        workers: 2
                    }
                },
                {
                    loader: 'babel-loader',
                    options: {
                        presets: ['@babel/preset-env']
                    }
                }
            ],
        }
    ]
},


externals
- 避免某些包被打包到bundle
- 比如某些通过cdn引用的资源，比如jQuery，配置externals就可以让其不被打包
module.exports={
    externals: {
        jquery: 'jQuery'
    }
}


dll
- 背景：默认情况下node_modules中使用到的内容都会被打包成一个chunk，所以这个chunk的体积是很庞大的，而通过dll将这些库单独的拆分成不同的chunk单独打包，可以加快加载chunk的时间。

1.先随便新建一个js文件：比如取名为webpack.dll.js
- 用于实现单独打包某些库的功能。并且在将来第二次构建的时候不再重新打包这些库。可以提升打包构建的速度。
const resolve = require('path')
const webpack = require('webpack')
// 使用dll技术对某些库进行单独打包
module.exports = {
    entry: {
        // 左边表示打包后的chunk名，右边表示被打包的库
        jquery: ['jquery']
    },
    output: {
        // 指定打包后的文件名，所以这里打包后是jquery.js文件
        filename: '[name].js',
        // 打包后的路径
        path: resolve(__dirname, 'dll'),
        // 打包后的库里面 向外暴露的内容叫什么名字
        library: '[name]_[hash]'
    },
    plugins: [
        // 插件的作用：打包生成一个manifest.json文件，用于指定哪些库不用在进行重复打包，并且指定了这些库与单独打包后的chunk之间的映射关系，这样在使用这些库的时候，根据manifest.json文件，就会去匹配对应打包后的chunk
        new webpack.DllPlugin({
            // 将打包后的chunk名映射到manifest.json文件中
            name: '[name]_[hash]',
            // 输出文件的路径
            path: resolve(__dirname, 'dll/manifest.json')
        })
    ]
}

2.在webpack.config.js中进行配置
const resolve = require('path')
const webpack = require('webpack')
// 使用dll技术对某些库进行单独打包
module.exports = {
    entry: {
        // 左边表示打包后的chunk名，右边表示被打包的库
        jquery: ['jquery']
    },
    output: {
        filename: '[name].js',
        path: resolve(__dirname, 'dll'),
        // 打包后的库里面向外暴露的内容叫什么名字
        library: '[name]_[hash]'
    },
    plugins: [
        // 插件的作用：打包生成一个manifest.json文件，用于提供源代码与单独打包出的jquery chunk之间的映射关系
        new webpack.DllPlugin({
            // 映射 库的 暴露内容的昵称
            name: '[name]_[hash]',
            // 输出文件的路径
            path: resolve(__dirname, 'dll/manifest.json')
        })
    ]
}

3.第一次构建的时候，需要在通过webpack进行构建之前，通过webpack --config webpack.dll.js 对指定的第三方库单独进行一次打包。

webpack配置详细讲解

entry
module.exports={
    // 1.单入口，输出单个chunk
    entry: './src/index.js',
    // 2.多入口，输出单个chunk --- 唯一的用途：添加./src/index.html，使修改html时启动HMR
    entry: ['./src/index.js', './src/add.js'],
    // 3.多入口，输出多个chunk。key的属性即为chunk的名称
    entry: {
         index11: './src/index.js',
         add11: './src/add.js'
    },
    // 4.特殊：
    entry: {
        index11: ['./src/index.js', './src/count.js'],//这里只会打包为一个chunk
        add11: './src/add.js'//这里会打包为一个chunk
    },
}


output
module.exports ={
    output: {
        // 打包输出文件的路径+名称
        filename: 'js/[name].js',//默认的[name]为main
        // 打包输出的文件所在的目录
        path: resolve(__dirname, 'build'),
        // 给所有引入的资源路径加上一个公共的前缀
        publicPath: '/build',
        // 非入口chunk的名称，比如通过import()方式引入的文件，都会单独的打包成一个chunk，这里可以设置这个chunk的名称
        chunkFilename: 'js/[name]_chunk.js',
        // 指定整个库向外暴露的变量名
        library: '[name]',//name表示取值为原chunk的名称
        // 指定上述变量添加在哪个属性上
        // libraryTarget: 'window'//这里添加在window上，一般的话会用在browser中
        // libraryTarget: 'global'//添加在global上，会用在nodejs中
        libraryTarget: 'commonjs'//特殊的例子，如果设置为commonjs,则表示会以commonjs的方式暴露上述变量
    },
}


module
module.exports ={
    module: {
        rules: [
            {
                // 匹配规则
                test: /\.css$/,
                // 使用多个loader
                use: ['style-loader', 'css-loader']
            },
            {
                test: /\.js$/,
                // 排除的文件
                exclude: /node_modules/,
                // 缩小处理的范围
                include: resolve(__dirname, 'src'),
                // 优先执行
                // enforce:'pre',
                // 延后执行
                enforce: 'post',
                // 使用单个loader
                loader: 'eslint-loader',
                //loader的配置
                options: {

                }
            }, {
                // 一个文件只匹配其中一个loader
                oneOf: [

                ]
            }
        ]
    },
}


resolve
module.exports ={
    resolve: {
        // ❤️❤️❤️ 配置路径的别名，这样可以在路径层数较多时方便的写出指定文件的路径
        alias: {
            '$css@': resolve(__dirname, 'src/css')
        },
        // 配置引用路径时可以省略的后缀名，默认可以省略.js和.json
        extensions: ['.js', '.json', '.jsx'],
        // 告诉webpack解析模块去找哪个目录，默认会找webpack.config.js所在目录下的node_modules，如果找不到，则会往上一级继续查询
        // 可以手动添加查找的路径，若找不到，再采用原来的往上查找的规则
        modules: [resolve(__dirname, '../../node_modules', 'node_modles')]
    }
}


devServer
module.exports={
    devServer: {
        // 指定热部署的路径，注意：这里是构建后的路径
        contentBase: resolve(__dirname, 'build'),
        // 监听contentBase目录下的所有文件，一旦文件变化浏览器就会重新加载
        watchContentBase: true,
        // 启动gzip压缩
        compress: true,
        // 运行的端口号
        port: 3000,
        // 域名
        host: 'localhost',
        // 默认打开浏览器
        open: true,
        // 开启HMR功能
        hot: true,
        // 关闭启动服务器时的日志信息
        clientLogLevel: 'none',
        // ❤️ 只展示基本的启动日志
        quiet: true,
        // 如果出错了，不要全屏展示错误信息
        overlay: false,
        // ❤️❤️❤️ 服务器代理：解决开发环境的跨域问题
        proxy: {
            // 原理：一旦devServer所开启的服务器监听到了以/api开头的请求，就会把该请求转发到指定的端口为5000的服务器上，由这个服务器发送请求请求数据，这样就不存在跨域的问题
            '/api': {
                target: 'http://localhost:5000',
                // 发送请求的时候，将请求的路径重写：这里是将/api前缀去掉，即 /api/xxx -->  /xxx
                pathRewrite: {
                    '^/api': ''
                }
            }
        }
    }
}


optimization
module.exports={
    optimization: {
        // optimization可以将使用到的node_modules中的代码单独打包成一个chunk输出
        // 且若多入口的chunk中引入了相同的第三方库，则该库只会被打包一次输出到这个chunk中
        // 内部使用的是splitChunks这个插件
        splitChunks: {
            chunks: 'all',
            miniSize: 30 * 1024,//超过指定大小的才能被打包，这里是必须超过30kb
            maxSize: 0,//设置为0，表示体积没有上限，再大也可以被单独打包
            minChunks: 1,//该库至少要被引用一次，否则不会进行打包
            maxAsyncRequests: 30,//这个库按需加载时并行加载的文件的最大数量，如果超过30个，则这个库就不会被打包
            maxInitialRequests: 3,//入口js文件最大并行请求数量
            name: true,//表示打包后的文件可以使用自定义的命名规则
            automaticNameDelimiter: '~',//指定文件名名称之间的连接符
            // 可以将引用的第三方库按照不同的风格分开打包，下面每一个对象表示一种分割打包的风格
            cacheGroups: {
                // 这里表示node_modules中的库会被打包到vendors组的chunk中，这里打包后的文件名为vendors~xxx.js xxx为模块名称
                // 注意：这里仍然满足上面设置的规则，比如大小需要超过30kb
                vendors: {
                    test: /[\\/]node_modules[\\/]/,
                    priority: -10//设置优先级
                },
                default: {
                    // 该库至少要被引用两次，这里会覆盖前面的minChunks
                    minChunks: 2,
                    prority: -20,//设置优先级
                    reuseExistingChunk: true//如果当前打包的模块和之前被提取的模块是同一个，则会复用这个模块，而不是重新进行打包。比如在多入口文件中，多个文件都引用了相同的库，则此时只会打包一个，其他文件会复用之前打包好的chunk
                }
            }
        },
        // 若当前模块通过import('')引用了其他模块，则该模块被打包后在chunk中会存一个被引用模块文件名的hash值，当被引用的文件修改时，hash也跟着变了，此时原chunk由于内部的hash改变了，也会重新进行打包。这样产生的效果就是缓存的chunk失效了。
        // 而runtimeChunk可以实现将原chunk中的hash单独打包为一个runtime.js文件，此时通过import('')被引用的文件改变时，原chunk不会被重新打包，而是runtime.js发生变化，这是因为原chunk中不再存储该hash值了。这样就避免了原chunk再次被打包。这样产生的效果就是
        runtimeChunk: {
            name: entrypoint => `runtime-${entrypoint.name}`
        },
        // 配置生产环境的压缩方案：js、css
        minimizer: [
            new TerserWebpackPlugin({
                // 开启缓存
                cache: true,
                // 开启多进程打包
                parallel: true,
                // 启动source-map
                sourceMap: true
            })
        ]
    },
}



webpack5
- 要点：
  - 通过持久缓存来提高构建性能
  - 使用更好的hush值算法和默认值来改善长期缓存.
  - 通过更好的树摇和代码生成来改善捆绑包大小.
  - 清除处于怪异状态的内部结构
