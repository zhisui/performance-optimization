**_以下实践基于 vue-cli3_**

# 查看 vue-cli 默认的 webpack 配置

vue-cli3 中的[vue-cli-service inspect](https://cli.vuejs.org/zh/guide/cli-service.html#vue-cli-service-inspect)
可以查看 vue-cli 默认的 webpack 配置，可在 pakeage.json 文件中的 scripts 配置`"inspect": "vue-cli-service inspect  > output.js ",`,其中`output.js`是输出的配置文件名，配置默认环境为开发环境。在终端中输入` npm run inspect` 或者 `yarn inspect`即可获取 output.js 文件

vue-cli 中关于 webpack 的配置可以在[configureWebpack](https://cli.vuejs.org/zh/guide/webpack.html#%E7%AE%80%E5%8D%95%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%B9%E5%BC%8F)中进行配置。

## module.noParse（无需解析内部依赖的包）

[noParse](https://www.webpackjs.com/configuration/module/#modulenoparse)是解析引入模块里面的相关依赖，如果项目中使用的库并没有引入其他的依赖，如 lodash 这种存 js 库、jquery 等等，这些库就没有引入其他的库，配置此项就可以减少解析时间

```js
module.exports = {
    module: {
        noParse: /^(lodash|dayjs|cesium)$/,
    },
}
```

## resolve.extensions

在开发中我们会有各种各样的模块依赖，这些模块可能来自于自己编写的代码，也可能来自第三方库， resolve 可以帮助 webpack 从每个 require/import 语句中，找到需要引入到合适的模块代码,通过 resolve.extensions 是解析到文件时自动添加拓展名，vue-cli 配置默认情况如下：

```js
 module.exports = {
    resolve: {
    extensions: [
      '.tsx',
      '.ts',
      '.mjs',
      '.js',
      '.jsx',
      '.vue',
      '.json',
      '.wasm'
    ],

  }
  },
```

当我们引入文件的时候，若没有文件后缀名，则会根据数组内的值依次查找.

当我们配置的时候，则不要随便把所有后缀都写在里面，这会调用多次文件的查找，这样就会减慢打包速度

## splitChunks 分包

配置 splitChunks 将一些公用的组件或者第三方库打包成一个 js 文件（具体情况看项目结构），可以避免重复打包，从而减少打包体积

```js
module.exports = {
    optimization: {
        splitChunks: {
            chunks: 'async',
            minSize: 1000,
            minChunks: 1,
            maxAsyncRequests: 10,
            maxInitialRequests: 5,
            automaticNameDelimiter: '~',
            name: true,
            cacheGroups: {
                vendors: {
                    test: /[\\/]node_modules[\\/]/,
                    priority: -10,
                },
                default: {
                    minChunks: 1,
                    priority: -20,
                    reuseExistingChunk: true,
                },
                dll: {
                    test: /[\\/]node_modules[\\/](vue|vue-router|vuex|view-design|element-ui)[\\/]/,
                    name: 'dll',
                    chunks: 'all',
                },
                api: {
                    test: /[\\/]api[\\/]/,
                    name: 'api',
                    priority: 0,
                    chunks: 'all',
                },

                echarts: {
                    name: 'echarts',
                    test: /[\\/](echarts|echarts-gl|ECharts|eCharts)[\\/]/,
                    priority: 0,
                    chunks: 'all',
                },
                lodash: {
                    name: 'lodash',
                    test: /[\\/]node_modules[\\/](lodash-es|lodash)[\\/]/,
                    priority: 0,
                    chunks: 'all',
                },

                map: {
                    name: 'map',
                    test: /[\\/]components[\\/](Map|map[\\/])/,
                    priority: 0,
                    chunks: 'all',
                },
            },
        },
        minimizer: [
            new UglifyJsPlugin({
                cache: true,
                parallel: 4,
                uglifyOptions: {
                    output: {
                        comments: false, //去掉注释
                    },
                },
            }),
        ],
    },
}
```

## 开启多进程打包

可以使用`UglifyJsPlugin`或者`ParallelUglifyPlugin`来开启多进程打包，
同时也可以删除删除多余的代码、注释。

```js
module.export = {
    optimization: {
        minimizer: [
            new UglifyJsPlugin({
                cache: true, //开启缓存
                parallel: 4,
                uglifyOptions: {
                    output: {
                        comments: false, //去掉注释
                    },
                },
            }),
        ],
    },
}
```

## 开启 gzip 压缩

可通过安装`compression-webpack-plugin` 插件对文件的大小进行压缩(同时需要后端配置支持 gzip)

```bash
npm install compression-webpack-plugin --save-dev
```

在配置文件中引入并使用

```js
const CompressionPlugin = require('compression-webpack-plugin')
module.exports = {
    plugins: [
        new CompressionPlugin({
            test: new RegExp('\\.(' + productionGzipExtensions.join('|') + ')$'),
            threshold: 10240,
            deleteOriginalAssets: false,
        }),
    ],
}
```
