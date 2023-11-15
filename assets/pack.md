**_以下实践基于 vue-cli3_**

# 查看 vue-cli 默认的 webpack 配置

vue-cli3 中的[vue-cli-service inspect](https://cli.vuejs.org/zh/guide/cli-service.html#vue-cli-service-inspect)
可以查看 vue-cli 默认的 webpack 配置，可在 pakeage.json 文件中的 scripts 配置`"inspect": "vue-cli-service inspect  > output.js ",`,其中`output.js`是输出的配置文件名，配置默认环境为开发环境。在终端中输入` npm run inspect` 或者 `yarn inspect`即可获取 output.js 文件

vue-cli 中关于 webpack 的配置可以在[configureWebpack](https://cli.vuejs.org/zh/guide/webpack.html#%E7%AE%80%E5%8D%95%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%B9%E5%BC%8F)中进行配置。

## module.noParse（无需解析内部依赖的包）

[noParse](https://www.webpackjs.com/configuration/module/#modulenoparse)是解析引入模块里面的相关依赖，如果项目中使用的库并没有引入其他的依赖，如 lodash 这种存 js 库、jquery 等等，这些库就没有引入其他的库，配置此项就可以减少解析时间

```js
module: {
        noParse: /^(lodash|dayjs|cesium)$/,
    },
```

## resolve.extensions

在开发中我们会有各种各样的模块依赖，这些模块可能来自于自己编写的代码，也可能来自第三方库， resolve可以帮助webpack从每个 require/import 语句中，找到需要引入到合适的模块代码,通过resolve.extensions是解析到文件时自动添加拓展名，vue-cli配置默认情况如下：

```js
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

  },
```

当我们引入文件的时候，若没有文件后缀名，则会根据数组内的值依次查找.

当我们配置的时候，则不要随便把所有后缀都写在里面，这会调用多次文件的查找，这样就会减慢打包速度
