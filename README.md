# webpack3.0-note
My note for learning webpack3.0 !

##一、webpack简述
一个写在 webpack.config.js 里的JS配置文件而已
webpack 是把css和js 压缩成一个 js 为了节省HTTP 请求数量，然后缓存达到客户端缓存的目的


准备知识：
hash
 // 每个版本的项目打包后都会生成一个属于这个版本的哈希值。
// 因为JS也可以在浏览器做缓存，所以哈希值可以帮我判断，如果src的url和缓存的url不同，就会去服务器下载，否则繁殖，所以这样做会减少请求,减轻服务器压力，省带宽，省钱，总之就是好啊。

 @1.target : 'web' //target : 'node' 目标用于哪里
              web : 打包后让浏览器能读明白（默认是web）
              node ： 打包后能让node读明白


```JavaScript
既然webpack的功能是用来打包文件，减少http请求的，那么，webpack到底是如何打包呢？它的打包流程到底是怎样的呢？
//             -- 按照webpack.config.js的配置顺序打包  entry里文件倒序来进行打包
//             -- 根据entry的js文件，判断需要打包的文件，
//             -- 打包的文件在module中找到对应loader处理，生成对应的map文件或临时文件
//             -- 再根据plugins数组中定义的插件处理map中的文件按照output选项输出到相应配置中

```
##二、安装
```JavaScript
1. 环境：安装 node.js和npm 并初始化 npm init-y
2. npm install webpack --save-dev
```
##三、入口和出口
```JavaScript
Path :
        const path = require("path");
Entry ：
 // 1.入口文件可以是一个也可以是多个
 // 2.一般像SPA应用都是只有一个main.js作为入口
        entry : {
            main : './src/main.js',
            index : './src/index.js'
        }
Output ：   
        output : {
             path : path.resolve(__dirname,'./dist')，//这里的是绝对路径
             // filename : 输出文件的文件名，[name]智能化的取entry里对应的key名来作为输出文件的文件名
             filename : '[name].js'
        },
```
##四、模块文件打包处理
```JavaScript
module是干嘛的？
  module 也就是模块的意思，在webpack的眼里一切皆模块，比如 .css .js .html .jpg .less ... ;webpack的官方图就无比的形象
  对于不同的模块需要用不同的加载器（loader）来处理，而加载器就是webpack最重要的功能，通过安装不同的加载器可以对各种后缀名的文件进行处理，
module 如何进行配置？ 
  -- 在module对象的rules属性中可以指向一系列的loaders,每一个loader都必须包含test和use两个选项。
  -- 下面CSS文件打包的意思就是：当webpack编译过程中遇到require()或import语句导入一个后缀名为.css的文件时,先将他通过css-loader转换，
     在通过style-loader转换，然后继续打包。
  -- use选项的值可以是数组或字符串，如果是数组，它的编译顺序就是从后往前。
  -- 最简单的写法就是当你需要什么loader的时候，在npm中取搜索，然后对照人家的文档照抄。
```

```JavaScript
  1.css文件打包
        npm install --save-dev style-loader css-loader
        module : {
            //规则 -- 对应文件的打包的规则
            rules : [
                 {
                    test : /\.css$/,
                    use : ['style-loader','css-loader']
                  }
             ]
         }    
    这样打包后所有的CSS样式都是通过在html的<head>标签下创建<style>标签来引入的，所以CSS是通过JavaScript动态创建的，
    也就意味着样式代码都已经编译在JS文件里了，但在实际业务中，并不希望这样做，因为项目大了样式会很多，都放在JS里太占体积，还不能做缓存。
    所以要将CSS代码从JS里抽离出来，单独打包成 .css文件,那么如何从js中抽离出css呢？
        npm install --save-dev extract-text-webpack-plugin  (注意对应webpack的版本)
        const ExtractTextPlugin = require('extract-text-webpack-plugin')
            module ：{
                rules : [
                    {
                       test : /\.css&/,
                       use : ExtractTextPlugin.extract({
                           fallback : 'style-loader',
                           use : 'css-loader'
                       })
                    }
                ]
            }
            plugins : [
              new ExtractTextPlugin('assets/css/[hash].css');
            ]
     如此就把散落在各地的CSS提取了出来，并且生成了.css文件，最终在html里通过<link>的形式加载它   
```
```JavaScript
2.图片文件打包
  webpack 打两种包 一种是JS中引入的图片，一种是CSS中引入的图片
       值得注意在JS中引入图片的方式
         -- js文件中需要用require引入图片路径
         -- var imgUrl = require('xxxx/xxx.png')
       npm install --save-dev file-loader url-loader   // url-loader的很多依赖是file-loader的第三方组件，所以一般都会同时安装
       module: {
             rules: [
                 {
                    test: /\.(jpg|gif|png|woff|svg|eot|ttf)\??.*$/,
                    use: [
                           {
                              loader: "url-loader",
                              options : {
                                 //如果图片的大小小于 5000B 那么就将其打包成 base64格式
                                 limit : 5000,
                                 outputPath : 'static/img/',
                                 name : '[hash].[ext]',
                                  //公有路径：
                                  publicPath : 'http://www.cddXXXX.com/'
                                  //publicPath : webSetPublicPath.publicPath
                               }
                             }
                      ]
                   }
               ]
       }
         
        
  一般情况下我们不会选择在html中插入图片，但是如果有特殊需要想要打包html中的图片，可以module中配置如下
        npm install --save-dev html-withimg-loader
           {
              test : /\.(htm|html)$/i,
              use : ['html-withimg-loader']
           }
```
```JavaScript
3. json 读取,webpack 3.0 之后就不在需要相关的 loader
   举个栗子 ： 
   //json文件
    {
      "name":"ddy",
      "age":"26"
    }
   //JS文件
   const jsonData = require('./assets/json/jsonData.json')
   document.write(jsonData.name + '--' +jsonData.age)
```
##五、插件（Plugins）的磅礴与魅力
```JavaScript
简述
webpack 的插件功能很强大而且可以定制。
```
```JavaScript
常用插件介绍 ：
  1.JS压缩插件 
     npm install uglifyjs-webpack-plugin --save-dev
     const UglifyJsPlugin = require("uglifyjs-webpack-plugin");
     plugins : [
         new UglifyJsPlugin()
     ]
  2.提取模板，并保存入口 html 文件,html打包发布，用于服务器访问
    html页面中不在需要 script/link 标签引入 js/css文件,因为打包后 html 中会自动引入，如下
    <script type=text/javascript src=/temp/main.js?7ea06b34eff84858f645></script>

    npm install html-webpack-plugin --save-dev
    const path = require("path")
    const htmlWebpackPlugin = require("html-webpack-plugin")
    plugins : [
         new HtmlWebpackPlugin({
             //minify 是HtmlWebpackPlugin 引用别人造的轮子
             minify : {
               // removeAttributeQuotes 的作用就是 去除 src href ... 后面的引号，减少体积。
               removeAttributeQuotes :true
             },
             hash : true,
             // 需要提取的 html模板 路径
             template : './index.html',
             //替换 html 中 title标签中的内容
             // 用法 ：在title标签中 <title><%= htmlWebpackPlugin.options.title %><title>
             title : 'webpack',
             //webpack打包后 替换 我们所有相关的 url ,因为开发时我们写入的路径和生产中的路径不太一样。
             // 用法 : <img src="<%= htmlWebpackPlugin.options.projectPath %>/assets/img/a/jpg">
             // 打包后就变成了 <img src="static/assets/img/a/jpg" >
             // projectPath 后面还可以跟各种 CDN 以及 http://...
              projectPath : 'static'
          })
     ]
  3.正确姿势引入第三方库
    第一种 ：JS文件中 import $ from "jquery"   这种方式 不管你是否使用了，都会将引入的库进行打包
    第二种 ：利用webpack插件 这种方式 只有在你调用了的时候才会被打包进去
            举个栗子 ：
                npm install jquery --save
                new webpack.ProvidePlugin({
                    $ : "jquery",
                    "jQuery" : "jquery"
                })
  4.热加载插件
    new webpack.HotModuleReplacementPlugin()

```
##六、watch
```JavaScript
作用 ：监听文件的改动，自动将文件改动更新到dist目录中对应文件
场景 ： 
配置 ：
         watchOptions : {
         	  poll : 1000,               //时间间隔，间隔多长时间查看一次文件是否有改动
         	  aggregeateTimeout : 500,   //防止手贱 
         	  ignored : /node_modules/   //不需要监听的目录
         }
         这时文件的监听会在dist目录产生垃圾（.js/.json）文件，如何归类这些垃圾文件呢？
         在output中添加设置：
         output：{
            hotUpdateChunkFilename : './hot/hot-update.js',
            hotUpdateMainFilename : './hot/hot-update.json',
         }
```



##七、启动神奇的 webpack-dev-server
```JavaScript
简述
在开发环境中使用。 通过建立一个WebSocket链接来实时响应代码的修改。
功能 ：启动一个服务器 、接口代理 、热更新等
 webpack 3.0+ 内置了webpack-dev-server ,如果是低于3.0版本，需要npm install webpack-dev-server --save-dev

```
```JavaScript
1.配置 devServer 启动服务器和接口代理
           devServer : {
               // 1.启动服务的 内容目录
               // 2.默认服务启动的是当前内容目录下index.html
               contentBase : path.join(__dirname,'./'),
               host : '192.168.1.151',
               port : '9000',
               // HTML 服务端压缩
               compress : true
            }
2. 终端输入命令 webpack-dev-server
3.打开浏览器输入 http://192.168.1.151:9000 这时你就看到了index.html的内容了
4.但是此时我们并没有利用到它最神奇的地方 --- 热更新
5.可以尝试修改下JS文件的内容，然后保存这时终端就开始编译了，但是页面却没有任何的变化，
  因为编译后将文件输出到了一个临时的位置，而不是index.html调用的位置。
6.如何启动热更新的作用呢 ?
            //在出口output处设置 publicPath
                output : {
                    // 公有路径，不会创建，虚拟在内存中,拥有较快的读取速度 ：作用是为了启动 webpack-dev-server的热更新功能
                    publicPath : '/temp/'
                }
7. 在index.html中修改 script的src
       <script type="text/javascript" src="temp/main.js"></script>
```

##八、生产环境与开发环境设置 及 模块化写法
```JavaScript
     @1.在package.json中的scripts中设置环境变量
        在windows：
        scripts : {
            'dev' : 'set token=dev&webpack',
            'build' : 'set token=build&webpack'
        }
        在Mac：
        scripts : {
            'dev' : 'export token=dev&&webpack',
            'build' : 'export token=build&&webpack'
        }
      @2.新建一个entry.js文件
         const entry = {}
         entry.dev = { build : './src/build.js' }
         entry.build = { build : './src/build.js' }
         module.exports = entry
      @3.在webpack.config.js中
        const entryFile = require("./entry.js");
        //在控制台中输出环境变量
        console.log(encodeURIComponent(process.env.aaa))
        if(process.env.aaa === 'build'){
            //生产环境
            var website = {
                "publicPath" : "http://www.qianduangongchengshi.com/",
                "version":"1.0.0
            };
            var entry = entryFile.build;
        }else {
            //开发环境
            website = {
                "publicPath" : "http://192.168.1.131:9000/",
                "version":"1.0.0"
            };
            var entry = entryFile.dev;
        }
```
##九、独立公共模块
```JavaScript
//简单来说其意思就是：把在公共入口处定义的名称拿出来单独去打包
//打包的顺序是从后往前，index.html 引入的顺序就是也就是从后往前
  @1.在入口文件
      entry : {
          index : './src/index.js',
          jquery : 'jquery',
          vue : 'vue'
      }
   @2.在插件配置中
      plugins : [
        new webpack.optimize.CommonsChunkPlugin({
            name : ['jquery','vue'],
            filename : 'assets/js/[name].min.js',   //注意前面不需要写 './'
            minChunks : 2
        })

      ]
```
##十、静态资源文件发布到指定输出环境中
```
      npm install --save-dev copy-webpack-plugin
      const copyWebpackPlugin = require('copy-webpack-plugin')
      new copyWebpackPlugin([{
          from : __dirname + '/src/public',
          to : './public'
      }])
```

##十一、vue-loader
```JavaScript
使用 .vue 文件需要先安装vue-loader、vue-style-loader 等加载器并做配置。
因为要使用ES6 语法，还需要安装 babel 和 babel-loader 等 加载器。使用npm 逐个安装 ：

npm install --save vue
npm install --save-dev vue-loader
npm install --save-dev vue-style-loader
npm install --save-dev vue-template-compiler
npm install --save-dev vue-hot-reload-api
npm install --save-dev babel
npm install --save-dev babel-loader
npm install --save-dev babel-core
npm install --save-dev babel-plugin-transform-runtime
npm install --save-dev babel-preset-es2015
npm install --save-dev babel-runtime
npm install --save-dev extract-text-webpack-plugin
安装完成之后，修改配置文件webpack.config.js 来支持对 .vue 文件及ES6 的解析

const path = require("path");
const ExtractTextPlugin = require("extract-text-webpack-plugin");

module : {
  rules : [
     {
        test : /\.vue$/,
        use : [
            {
              loader : "vue-loader",
              options : {
                  loaders : {
                      css : ExtractTextPlugin.extract({
                         use : 'css-loader',
                         fallback : 'vue-style-loader'
                      })
                  }
               }
             }
         ]
     },
     {
        test : /\.js$/,
        use : "babel-loader",
        exclude: /node_modules/,
     }
  ]
}

vue-loader在编译 .vue 文件时会对 <template> <script> <style> 分别处理，所以在 vue-loader 选项里多了一项options来进一步对不同语言进行配置。
比如对CSS 进行处理时，会先通过css-loader解析，然后把处理结果在交给vue-style-loader处理，
当技术栈多样化时，可以给 <template> <script> <style> 都指定不同的语言，比如 <template lang="jade"> 和 <style lang="less">,然后配置loader就可以了。

新建一个 .babelrc 的文件，并写入babel配置，webpack 会依赖此配置文件来使用babel编译ES6代码：
      {
          "presets" : ["es2015"],
          "plugins" : ["transform-runtime"],
          "comments" : false
      }

配置好这些后就可以使用 .vue 文件了


```















