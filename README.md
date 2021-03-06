# webpack-vue

[项目预览](http://119.23.244.71:8092)

## webpack核心概念
1. entry：入口，webpack执行构建第一步
2. Output：输出
3. plugins：扩展插件，在 Webpack 构建流程中的特定时机注入扩展逻辑来改变构建结果或做你想要的事情。
4. Module：模块，webpack一切皆模块，一个模块对应一个文件。
5. Chunk：代码块，一个 Chunk 由多个模块组合而成，用于代码合并与分割。 Loader：模块转换器，用于把模块原内容按照需求转换成新内容。


## webpack执行流程
![ webpack执行流程](https://note.youdao.com/yws/public/resource/dfc531ff259bc48a2f1e1ba0086a5b05/xmlnote/WEBRESOURCEa685f4bd2184b26ac84319b04dd622e9/5547)


---

环境：`node > 8.2`

- 首先在项目中`npm init -y`， 生成`package.json`
- 出来安装`webpack`外，还需要安装`webpack-cli`
```
npm init -y
// 需要先安装webpack-cli
npm i webpack webpack-cli -D 
```

在`package.json`中配置
```
"script": {
    "dev":"webpack --color --progress --mode development", 
    "build":"webpack --mode production" // 生产环境js会压缩
}

/*
--progress 打包进度
--watch 自动监控文件变化
--color 彩色显示打包提示信息
--display-modules: 打包的模块 
--display-reasons: 打包原因
*/

```


##### development mode默认
> - 开启 output.pathinfo
>- 关闭 optimization.minimize
 
##### production mode 默认
> - 关闭 in-memory caching
> - 开启 NoEmitOnErrorsPlugin
> - 开启 ModuleConcatenationPlugin
> - 开启 optimization.minimize


webpack4.0的0配置，就是无需像以前一样配置`webpack.config.js`。
- webpack4.0默认`entry`路径为`./src/index.js`
- 默认`output`路径为`./dist/main.js`


但是，像这种0配置在实际开发中根本没什么用，还是需要重新配置`webpack.config.js` 。

--- 

## 配置webpack.config.js

```
// webpack.config.js

const path = require('path')

module.exports = {
    entry: './src/index.js',    // 入口文件
    output: {
        filename: "[name].[hash].js",
        path: path.join(__dirname, 'dist'),
    }
}
```

然后运行`npm run dev`，打包生产环境.

- 把两个文件分别打包
```
module.exports = {
    entry: {
        index: './src/index.js',  
        main: './src/main.js'
    }
    output: {
        filename: "[name].[hash].js",
        path: path.join(__dirname, 'dist'),
    }
}
```


- ### webpack-html-plugin
创建`index.html`，将生成的js动态引入到`index.html`中，我们需要安装插件`webpack-html-plugin`。

在`webpack.config.js`中引入插件

```
const HtmlWebpackPlugin = require('webpack-html-plugin');
module.exports = {
    entry: {
        index: './src/index.js',  
        main: './src/main.js'
    },
    output: {
        filename: "[name].[hash].js",
        path: path.join(__dirname, 'dist'),
    },
    plugins: [
        new HtmlWebpackPlugin()
    ]
}
```

但是，这样在`./dist`中生成的`index.html`和项目目录下的`index.html`没有一点关系。为了以项目中的`index.html`为模板，只需给`html-webpack-plugin`的构造函数传入`template`。
```
    plugins: [
        new HtmlWebpackPlugin({
            template: './index.html'
        })
    ]
```

> template：就是以目录下的这个index.html文件为模板生成`dist/index.html`文件，然后执行 npm run dev 打包命令就能重新生成了。

在实际的开发中，最好将打包出的`dist`下的`js`分开存放。所以我们将输出路径加一个`js`目录

```
output: {
    filename: "js/[name].[hash].js",
    path: path.join(__dirname, 'dist'),
}
```

插件配置选项 `filename` 和 `inject`
```
plugins: [
    new HtmlWebpackPlugin({
        template: './index.html',
        filename: 'index-[hash].html',
        title: 'webpack'
        inject: 'head',
        minify: {
            removeComments: true,   // 去掉注释
            collapseWhitespace: true    // 去掉空行
        }
    })
]
```
- filename：打包生成的文件名，还可以加目录，默认没有写的时候是index.html。
- inject：有四个值：`true` | `head` | `body` | `false
    > 如果设置为head, 就是把js引入放在head标签里面, 如果设置为body，就是把js引入放在body里面， false: 不会引入js文件  true:引入js文件
- title: html的title标签
    >`<title><%= htmlWebpackPlugin.options.title %></title>`
- minify：压缩代码
- ......

1. chunk 属性
> 指定加载哪些chunk(如：js文件)

比如在开发多页面的时候，就会经常用到这个属性。

```
...
output: {
    index: './src/index.js',
    main: './src/index.js',
    app: './src/app.js'
}
...
plugins: [
    new HtmlWebpackPlugin({
        template: './index.html'
        filename: 'index.html',
        inject: true,
        chunk: ['index', 'app']
    }),
    new HtmlWebpackPlugin({
        template: './main.html'
        filename: 'main.html',
        inject: true,
        chunk: ['main', 'app']
    })
]

```

2. excludeChunks选项
> 这个很好理解，就是有很多chunks，排除不要加载的


- ### loader预处理
处理静态资源，需要加载各种loader，如html要用`html-loader`，css要用`css-loader`, `style-loader`等。

- 处理css文件
> npm i css-loader style-loader -D

配置`webpack.config.js`
```
 module: {
        rules: [
            {
                test: /\.css$/,
                exclude: /(node_modules)/,
                use: ['style.loader', 'css-loader']
            }
        ]
    }
```
use是从右到左执行，执行webpack打包时，先执行`css-loader`处理css文件，`style-loader`把css文件内嵌到浏览器。


- 把es6转为es5
> 安装:  npm i babel-loader babel-core babel-preset-env -D

```
module: {
    rules: [
        {
            test: /\.css$/,
            exclude: /(node_modules)/,
            use: ['style.loader', 'css-loader']
        },
        {
            test: /\.js$/,
            exclude: /(node_modules)/,
            include: [
                path.join(__dirname, 'src'),
            ],
            use: {
                loader: 'babel-loader',
                options: {
                    presets: ['env']
                }
            }
        }
    ]
}
```

- css3 添加浏览器前缀
> 安装postcss-loader和autoprefixer。`npm i postcss-loader autoprefixer -D`

在目录下新建posscss.config.js
```
module.exports = {
    plugins: [
        require('autoprefixer')({
            browsers : ['last 5 versions']
        })
    ]
}
```
意思是在postcss-loader中引入autoprefixer插件

在`webpack.config.js`中修改配置
```
module: {
    rules: [
        {
            test: /\.css$/,
            exclude: /(node_modules)/,
            use: [
                'style.loader', 
                {
                    loader: 'css-loader',
                    options: {
                        importLoaders: 1   
                    }
                },
                'postcss-loader'
            ]
        },
        {
            test: /\.js$/,
            exclude: /(node_modules)/,
            include: [
                path.join(__dirname, 'src'),
            ],
            use: {
                loader: 'babel-loader',
                options: {
                    presets: ['env']
                }
            }
        }
    ]
}

```


- 安装 less-loader 插件
> 安装: npm i less-loader less -D

安装完成之后，配置webpack.config.js
```
module: {
    rules: [
        ...
        {
            test: /\.less$/,
            use: ['style.loader', 'css-loader', 'less-loader']
        }
        ...
    ]
}
```

- 安装 html-loader
```
module: {
    rules: [
        ...
        {
            test: /\.(html)$/,
            use: ['html-loader']
        }
        ...
    ]
}
```

- 修改模板后缀，为模板赋值
 
> 安装ejs-loader

```
module: {
    rules: [
        ...
        {
            test: /\.(ejs)$/,
            use: ['ejs-loader']
        }
        ...
    ]
}
```

- file-loader 静态资源加载

安装 `npm i file-loader -D`

```
module: {
    rules: [
        ...
        {
            test: /\.(png|gif|jpg|svg|jpeg)$/i,
            use: {
                loader: 'file-loader',
                query: {
                    name: 'assets/[hash].[ext]'
                }
            }
        }
        ...
    ]
}
```
query部分的配置，是为打包的图片设置一个自定义的存储路径和文件名称。



- 分离css文件

> 安装 extract-text-webpack-plugin

```js
module: {
    rules: [
        ...
        {
            test: /\.css$/,
            // loader:['style-loader','css-loader']
            // 不再需要style-loader
            use: ExtractTextWebapckPlugin.extract({
                use: ['css-loader','postcss-loader']
            }),
        },
        ...
    ]
}
```



- ## 优化

#### 动态链接库DLL
把基础模块代码打包进入动态链接库，比如vue、react等。这样做打包速度会提升60%~70%。

1. 创建webpack.dll.config.js
```
const path = require('path');
const webpack = require('webpack');

module.exports = {
  entry: {
      vendor: ['vue', 'vue-router']
  },
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].dll.js',
    library: '[name]_library'
  },
  plugins: [
    new webpack.DllPlugin({
      path: path.join(__dirname, 'dist', '[name]-manifest.json'),
      name: '[name]_library'
    })
  ]
};
```

2. 在webpack.config.js中配置
```
    plugins: [
       new webpack.DllReferencePlugin({
            context: __dirname,
            manifest: require('./dist/vendor-manifest.json')
        })
    ]
```

3. 生成动态链接库
```
// package.json
// "dll": "webpack --config webpack.dll.config.js"

webpack --config webpack.dll.config.js
```

将会在当前目录生成dist目录，该目录下面的文件有 vendor.dll.js、vendor-manifest.json


在`index.html`中将文件引入

```
...
<body>
    <div id="root"></div>
    <script type="text/javascript" src="./vendor.dll.js"></script>
</body>
...
```

然后就可以开始调试了