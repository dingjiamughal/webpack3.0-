# webpack3 基础1

## webpack支持的三种模块化模式
```javascript
//es module
import util from 'util'
export default () => {}

//commonjs
const util = require('util')
module.exports = () => {};

// amd
require(['util'],(util)={})
```

## webpack.config.js

commonjs规范

```javascript
module.exports = {
  entry:{
    app:"./app.js"
  },
  output:{
    filename:"[name].[hash:5].js"
  }
}
```

## babel编译js

npm i babel-loader babel-core --save-dev npm i babel-preset-env --save-dev // es2015 es2016 es2017 env babel-preset-react babel-preset-state 0-3

```javascript
module:{
  rules:[
    {
    test:/\.js$/,
    use:{
      loader:"babel-loader",
      options:{
        presets:["@babel/preset-env",{
          targets:{
            browers:[">1%","last 2 versions"]
          }
        }]
      }
    },
    exclude:"/node_modules"
  }
 ]
}
```

babel polyfill //垫片 babel runtime transform 编译es6

npm i babel-plugin-transform-runtime --save-dev npm i babel-runtime babel-polyfill --save

import "babel-polyfill"

```javascript
// .babelrc
{
  "presets":[
    ["@babel/preset-env",{
      "targets":{
        "browers":["last 2 versions"]
      }
    }]
  ]
}
```

## 编译typescript

npm i typescript ts-loader awesome-typescript-loader --save-dev

```json
//tsconfig.json
{
  "compilerOptions":{
    "module":"commonjs",
    "target":"es5",
    "allowJs":true
  },
  "include":[
    "./src/*"
  ],
  "exclude":[
    "./node_modules"
  ]
}
```

```javascript
  // webpack.conf.js
module:{
  rules:[
    {
      test:/\.tsx?$/,
      use:{
        loader:"ts-loader"
      }
    }
  ]
}
```

## webpack.optimize.CommonsChunkPlugin 提取公共代码

CommonsChunkPlugin打包公共代码让浏览器识别到缓存文件，在多页应用中提高第二次之后的读写速度。

```javascript
{
  plugins:[
    new webpack.optimize.CommonsChunkPlugin(options)
  ]  
}
```

options配置项
`name` [name]数组类型</br>
`filename` </br>
`minChunks` 自动识别：模块最少出现次数，自动作为打包文件<br>
`chunks` 提取代码的范围<br>
`children` `deepChildren` `async`

业务场景 `spa` `spa+第三方依赖` `多页应用+第三方依赖+webpack生成代码`

```javascript
// CommonsChunkPlugin基本配置
// pageA.js 中引用了subA.js和sub.js和lodash，subA和subB中引用module.js
// task：进行代码分割的基础配置
const webpack = require("webpack")
const path = require("path")

module.exports = {
  entry:{
    "pageA":"./src/pageA",
    "pageB":"./src/pageB",
    "vendor":["lodash"]
  },
  output:{
    path:path.resolve(_dirname,"./dist"),
    filename:"[name].bundle.js",
    chunkFilename:"[name].chunk.js"//动态打包文件（非入口的js文件）
  },
  plugins:[
    new webpack.optimize.CommonsChunkPlugin({
      name:"['vendor','manifest']",
      minChunks:2
    }),
    //task提取公共代码module.js并异步加载用async
    new webpack.optimize.CommonsChunkPlugin({
      async:"async-common",
      children：true,//向下查找
      minChunks:2
    })
  ]
}
```

配置了`minChunk`必须是多entry，CommonsChunkPlugin只能识别多entry的情况

## 代码分割和懒加载

根据不同需求对js公共文件进行打包和按需加载

> 代码分割的场景 分离 业务代码、第三方依赖 分离 业务代码、公共代码、第三方依赖 分离 首次加载、访问后加载（多页应用）

`require.ensure` `require.include`

```javascript
const webpack = require("webpack")
const path = require("path")

module.exports = {
  entry:{
    "pageA":"./src/pageA",
  },
  output:{
    path:path.resolve(_dirname,"./dist"),
    publicPath:"./dist/",//代码发布路径，最终使发布到服务器的地址
    filename:"[name].bundle.js",
    chunkFilename:"[name].chunk.js"
  },
  // plugins:[
  //   new webpack.optimize.CommonsChunkPlugin({
  //     name:"common",
  //     minChunks:2
  //   })
  // ]
}
// pageA.js 中引用了subA.js和sub.js和lodash，subA和subB中引用module.js
// 任务：将第三方lodash进行代码分割
//pageA.js中
require.ensure(["lodash"],()=>{//require加载到页面
  const _=require("lodash")//执行lodash
},"vendor")

// 分离细分
//task：将subA.js,subB.js进行按需分离
if(true){
  require.ensure(["./subA"],()=>{
    const subA = require("subA")
  },"subA")
} else {
  import(/* webpackChunkName:'subpageB' */"./subB")
    .then(subB => { //  import方法，直接执行，webpack3新功能
    console.log(subB)
  })
}

此时，发现在subA和subB中的module.js没有被单独打包成一个文件。这时候就需要在pageA.js中
require.include("./module.js")
module文件在subA和subB.js中剔除，被单独提取到pageA.js
```

## 处理css
webpack处理css方式是通过webpack的css-loaderdui css文件进行打包，用js来对css进行加载

需要的加载器
`style-loader` 创建link标签</br>
`css-loader` 能在js引入css文件
>cnpm install css-loader style-loader --save-dev

```js
  //在js中
  import 'xxx.css'

  //webpack.config.js
  const path=require("path")
  module.exports={
    entry:{
      app:"./src/app.js"
    },
    output:{
      path:path.resolve(__dirname,"./dist"),
      publicPath:"./dist/ ",
      filename:"[name].bundle.js"
    },
    module:{
      rules:[{
        test:/\.css$/,
        use:[{//顺序是先让js中支持import css再放到页面中进行加载
          loader:"style-loader",
          options:{
            insertInfo:"#app",// 当做style标签打包在id="app"的dom内
            singleton:true,//合并样式
            transform:"./css.transform.js"
          }
        },{
          loader:"css-loader",
          options:{
             modules:true,//允许在css文件中引入别的css文件中的模块 `compire`
             localIdentName:'[path][name]_[local]_[hash:base64:5]'
          }
        }]
      }]
    }
  }
```
style-loader中的transform参数的作用
单独在根目录下创建文件，`css.transform.js`在此文件中可以进行js对css的逻辑处理，比如获取window对象，判断浏览器版本对所有的css文件进行操作
```js
// css.transform.js
module.exports= res => {
  console.log(css)//能够log出所有的css文件
  if(window.innerWidth >= 768){
    return css.replace("red","green")
  }else {
    return css.replace("red","blue")
  }
}
```

## 配置less/sass
>cnpm install less less-loader sass sass-loader

```js
modules:{
  rules:[{
    test:/\.less$/,
    use:[{
      loader:"less-loader"
    }]
  }]
}
```

## 提取css
>cnpm install  extract-text-webpack-plugin --save-dev

```js
const ExtractTextWebpackPluigin=require("extract-text-webpack-plugin")
  module:{
    rules:[{
      test:/\.less$/,
      use:ExtractTextWebpackPluigin.extract({//提取方法
        fallback:{//当不需要提取css的时候页面加载css的方法
          loader:"style-loader",
          options:{
            singleton:true,//合并样式
            transform:"./css.transform.js"
          }
        },
        use:[{  //use:如果要提取css需要用css-loader和less-loader加载到页面
          loader:"css-loader",
          options:{
             minimize:true,
             modules:true,
             localIdentName:'[path][name]_[local]_[hash:base64:5]'
          }
        },{
          loader:"less-loader",
        }]
      })
    }]
  },
  plugins:[
    new ExtractTextWebpackPluigin({
      filename:"[name].min.css",
      allChunks:false
    })
  ]
```

## PostCSS
>cnpm install postcss postcss-loader autoprefixer cssnano postcss-next --save-dev

`Autoprefixer` 加前缀</br>
```js
  div {
    display:flex;
  }

  div {
    display:-webkit-box;
    display:-webkit-flex;
    display:-ms-flexbox;
    display:flex;
  }
```
`css-nano` 优化和压缩css,css-loader中也使用css-nano进行压缩

`postcss-next` 可以使用未来的css新语法

```js
module:{
  rules:[{
    test:/\.less$/,
    use:ExtractTextWebpackPluigin.extract({//提取方法
      fallback:{
        loader:"style-loader",
        options:{
          singleton:true,
          transform:"./css.transform.js"
        }
      },
      use:[{
        loader:"css-loader",
        options:{
           minimize:true,
           modules:true,
           localIdentName:'[path][name]_[local]_[hash:base64:5]'
        }
      },{
        loader:"less-loader",
      },{
        loader:"postcss-loader",
        options:{
          ident:"postcss",//声明接下来使用的插件是给postcss使用的
          plugins:[
            require("autoprefixer")(),
            // :root {--mainColor:#fff;} div {color:var(--mainColor)}
            require("postcss-cssnext")()

          ]
        }
      }]
    })
  }]
}
```
设置浏览器对postcss兼容性
* 方法1：在`package.json`中配置
* 方法2：新建`.browserslistrc`文件
```json
"browserslist":[
  ">= 1%",,
  "last 2 versions"
]
```

## Tree Shaking
剔除无用的代码（项目中没有用到过的代码），针对css和js文件

### js中的tree shaking
```js
// task:引入lodash，并只使用`chunk`方法
// app.js
// import {chunk} from "lodash"

const Webpack = require("webpack")
plugins:[
  new Webpack.optimize.UglifyJsPlguin()
]
```

### css中的tree shaking
借助`purifycss-webpack`
>cnpm install purifycss-webpack glob-all --save-dev

```js
const PurifyCss = require("purifycss-webpack")
const path = require("path")
const glob = require("glob-all")

plugins:[
  new PurifyCss({ //  一定写在UglifyJsPlguin之前才生效
    paths:glob.sync([
      path.join(__dirname,"./*.html"),
      path.join(__dirname,"./src/*.js")
    ])
  }),
  new Webpack.optimize.UglifyJsPlguin()
]
```

## webpack打包图片

### webapck打包css中的图片
优化：自动合成雪碧图、压缩、base64
处理图片的4个加载器：
`file-loader` 帮助css处理图片</br>
`url-loader` base64编码</br>
`img-loader` 压缩图片</br>
`postcss-sprites` 合成雪碧图
>cnpm install file-loader url-loader img-loader postcss-sprites --save-dev

```html
<div class="">
  <img src="" alt="">
</div>
```
```js
module:{
  rules:[{
    test:/\.(jpg|jpeg|png|gif)$/,
    use:[{
      loader:"file-loader",
      options:{
        publicPath:"",
        outputPath:"dist/",
        useRelativePath:true
      }
    },{
      loader:"url-loader",  //  打包图片可以直接使用url-loader
      options:{
        name:"[name][hash:5].min.[ext]",//重命名图片文件[文件名].min.[后缀]
        limit:50000,//5kb
        // publicPath:"",
        outputPath:"dist/",
        // useRelativePath:true
      }
    },{
      loader:"img-loader",
      options:{
        pngquant:{  //调整图片精度
          quality:80
        }
      }
    }，{
      loader:"postcss-loader",//利用postcss配置生成雪碧图
      options:{
        ident:"postcss",
        plugins:[
          require("postcss-sprites")({
            spritePath:"dist/assets/img/sprites",
            retina:true
          })

        ]
      }

    }]
  }]
}
```

## webpack打包字体文件
`file-loader` `url-loader`

```js
module:{
  rules:[{
    test:/\.(eot|woff|woff2|ttf|svg)$/,
    use :[{
      loader:"url-loader",
      options:{
        name:"[name]-[hash:5].[ext]",
        limit:5000,
        publicPath:"",
        outputPath:"dist/",
        useRelativePath:true
      }
    }]
  }]
}
```

## 处理第三方JS库
`webpack.providePlugin` `imports-loader`
```js
resolve:{
  alias:{
    jquery$:path.resolve(__dirname,"src/lib/jquery.min.js")
  }
},
plugins:[
  new webpack.providePlugin({
    $:"jquery"
  })
]
```

```js
  modules:{
    rules:[{
      test:path.resolve(__dirname,"src/app.js"),
      use:[{
        loader:'imports-loader',
        options:{
          $:"jquery"
        }
      }]
    }]
  }
```

## 打包HTML
生成html自动自动识别引入css和js文件
`HtmlWebpackPlugin`
>cnpm install html-webpack-plugin --save-dev

```js
const HtmlWebpackPlugin = require("html-webpack-plugin")
plugins:[
  new HtmlWebpackPlugin({
    filename:"index.html",
    template:"./index.html",
    inject:false, //剔除生产版本引人的css和js
    chunks:["app"],//指定chunks，进行文件分割
    minify:{
      collapseWhitespace:true//html不留间隙
    }
  })
]
```

## HTML中引入图片
* 方法一 `html-loader`
```js
module:{
  rules:[{
    test:/\.html$/,
    use:[{
      loader:"html-loader",
      options:{
        pngquant:{
          quality:80
        }
      }
    }]
  }]
}
```
* 方法二 `require`
```html
  <img src="${require('./assets/xxx.png')}" alt="">
```

## 清除打包文件
每次打包时候清除上一次的文件
>cnpm install clean-webpack-plugin --save-dev

```js
const CleanWebpackPlugin = require("clean-webpack-plugin")
plugins:[
  new CleanWebpackPlugin(["dist"])
]
```

## webpack优化
提前载入webpack加载代码（将webpack生成的manifest文件作为script标签提前生成在html中） `html-inline-chunk-plugin`
>cnpm install html-inline-chunk-plugin --save-dev

```js
const HtmlInlineChunkPlugin = require("html-inline-chunk-plugin")
module:{
  rules:[{
    test:/\.js$/,
    use:[{
      loader:"babel-loader",
      options:{
        presets:["env"]
      }
    }]
  }]
},
plugins:[
  new webpack.optimize.CommonsChunkPlugin({
    name:"manifest"

  }),
  new HtmlInlineChunkPlugin({
    inlineChunks:["manifest"]
  })
]
```

# 搭建开发环境

## webpack watch mode
> webpack -watch

## webpack-dev-server
>cnpm install webpack-dev-server --save-dev
webpack-dev-server --open

* live reloading实时更新
* 不能打包文件
* 路径重定向
* 支持https
* 能在浏览器中显示编译错误
* 接口代理
* 热更新
```js
module.exports ={
  entry:{
    app:"./src/app.js"
  },
  output:{
    path:path.resolve(__dirname,"dist"),
    publicPath:"/",
    filename:"js/[name].bundle-[hash:5].js"
  },
  devServer:{
    inline:true,//在console开启打包状态
    port:8080,
    historyApiFallback:{//html重定向
      rewrite:{
        from:/^\/([a-zA-Z0-9]+\/?)([a-zA-Z0-9]+)/,
        to:function(ctx){
          return "/" + ctx.match[1] + ctx.match[2] + ".html"
        }
      }
    }
  }
}
```

## 热更新 module hot reload
热更新能保持原有的开发状态的基础上，进行模块刷新

```js
devServer:{
  hot:true
  hotOnly:true
},
plugins:[
  new webpack.HotModuleReplacementPlugin(),
  new webpack.NamedModulesPlugin()
]
```

## source-map
编译速度+不失去调试功能
```js
// js的source-map
devtool:"cheap-module-source-map"

//css的source0map 给没一个loader的options加sourceMap:true
modules:{
  rules:[{
    test:/\.less$/,
    use:[{
      loader:"style-loader",
      options:{
        singleton:false,//为true会把样式放在style标签
        sourceMap:true
      }
    },{...},{...}]
  }]
}
```

## esLint
`eslint-loader` ``
>cnpm install eslint eslint-loader eslint-plugin-html eslint-friendly-formatter --save-dev

代码规范：[JavaScript Standard Style](https://standardjs.com)
```js
rules:[{
  test:/\.js$/,
  use:[{
    loader:"babel-loader",
    options:{
      presets:{
        presets:["env"]
      }
    }
  },{
    //在babel-loader之前检测语法错误
    loader:"eslint-loader",
    options:{
      formatter:require("eslint-friendly-formatter")
    }
  }]
}]
```
另加`.eslitrc.js`
>cnpm install eslint-config-standard eslint-plugin-promise eslint-plugin-node eslint-plugin-import eslint-plugin-standard --save-dev

```javascript
// .eslitrc.js
module.exports = {
  root:true,
  extends："standard",
  plugins:[],
  env:{//所支持的环境
    browser:true,
    node:true,
  },
  rules:{//添加规则
    indent:["error",4],
    "eol-last":["error","never"]
  }
}
// webpack.conf.js
devServer:{
  overlay:true,//在浏览器中显示eslint报错信息
},
module:{
  rules:[{
    test:/\.js$/,
    include:[path.resolve(__dirname,"src"],
    exclude:[path.resolve(__dirname,"src/libs")]//eslint不处理这边文件
  }]
}
```

# 开发环境和生产环境

## 两种环境都需要的情况
* 入口
* 代码处理 xxx-loader
* 解析配置

## 两种环境分开的情况
`开发环境`需要的功能：
* 模块热更新
* sourceMap
* proxy
* eslint

`生产环境`需要的功能：
* 提取公共代码
* 压缩混淆
* 文件压缩、图片base64编码
* tree shaking 去除无用代码

## 区分两者的配置webpack-merge
`webpack-merge` 帮助拼接webpack公共配置+各自的配置
`webpack.dev.conf.js` 开发环境<br>
`webpack.prod.conf.js` 生产环境<br>
`webpack.common.conf.js` 公共<br>
