---
layout: post
title: Webpack CommonsChunkPlugin深度提取公共子模块
description: "webpack1.x利用 minChunks函数提取公共子模块"
date: 2017-03-05
tags: webpack    
---

> 在开发单页项目时, 在默认情况下, webpack给我们打包之后最后生成的js都在一文件中, 这个文件是非常大, 非常不利于传输, 所以我们可能需要根据需要提取一些公共子模块, 将js打包成多个文件, 我们采用 webpack自带的插件 `CommonsChunkPlugin` 来完成公共模块的提取工作.

### 基本使用

```
var webpack = require('webpack')
new webpack.optimize.CommonsChunkPlugin(options)
```

### options配置项(部分, 基于 webpack1.x)

```
{
  name: string, // or

  filename: string,
  // common chunk 的文件名模板。可以包含与 `output.filename` 相同的占位符。
  // 如果被忽略，原本的文件名不会被修改(通常是 `output.filename` 或者 `output.chunkFilename`)。
  // This option is not permitted if you're using `options.async` as well, see below for more details.

  minChunks: number|Infinity|function(module, count) -> boolean,
  // 在传入  公共chunk(commons chunk) 之前所需要包含的最少数量的 chunks 。
  // 数量必须大于等于2，或者少于等于 chunks的数量
  // 传入 `Infinity` 会马上生成 公共chunk，但里面没有模块。
  // 你可以传入一个 `function` ，以添加定制的逻辑（默认是 chunk 的数量）

  chunks: string[],
  // 通过 chunk name 去选择 chunks 的来源。chunk 必须是  公共chunk 的子模块。
  // 如果被忽略，所有的，所有的 入口chunk (entry chunk) 都会被选择。

  minSize: number,
  // 在 公共chunk 被创建立之前，所有 公共模块 (common module) 的最少大小。
}

```

这里的 `minChunks`是一个强大的配置项目, 简单配置时, 我们只需要 设置一个 `minChunks`: 2即可, 这样会把代码引入>=2次的提取出公共部分到一个文件中, 但在我的项目中, 我想自定义把相关的js库打包到一起, 这里就需要用到 `minChunks`的函数形式了 ,有了它我们可以 自定义 2个 chunk 抽取其中的公共部分来打包到一起了, 名字也可以自己起, 有了它可做到模块的 深度提取

在 webpack的配置文件的 plugins 的值中加上一下下面的代码:

```text

  new webpack.optimize.CommonsChunkPlugin({
			name: 'vendor',
			minChunks: function (module, count) {
				return (module.resource &&  /\.js$/.test(module.resource))
			}
		}),
		

    new webpack.optimize.CommonsChunkPlugin({
      name: 'vis',
      chunks: ['vendor'],
      minChunks: function (module, count) {
          const fileName = module.resource.substr(module.resource.lastIndexOf('/')+1).toLowerCase();
          const isEleme = module.resource.indexOf('element-ui') !== -1
          const visArr = ['id_validator.js','dom-to-image.js','city_data.js','vis-network.min.js', 'jquery.js', 'highcharts.js', 'more.js', 'charts.js', 'chart.js', 'echarts.js', 'radialindial.js', 'layer.js', 'date.js', 'html2canvas.js', 'promise.js'];
          return (
              module.resource &&
              /\.js$/.test(module.resource) && visArr.indexOf(fileName) != -1
          ) || isEleme
      }
    }),


    new webpack.optimize.CommonsChunkPlugin({
      name: 'elui',
      chunks: ['vis'],
      minChunks: function (module, count) {
          const fileName = module.resource.substr(module.resource.lastIndexOf('/')+1).toLowerCase();
          const arr = ['jquery.js', 'highcharts.js', 'more.js', 'charts.js', 'chart.js', 'echarts.js', 'radialindial.js', 'city_data.js', 'layer.js', 'date.js', 'html2canvas.js', 'promise.js'];
          const isEleme = module.resource.indexOf('element-ui') !== -1
          return (
              module.resource &&
              /\.js$/.test(module.resource)  && arr.indexOf(fileName) != -1
          ) || isEleme
      }
    }),


    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      chunks: ['elui'],
      minChunks: function (module, count) {
          const fileName = module.resource.substr(module.resource.lastIndexOf('/')+1).toLowerCase();
          const arr = ['jquery.js', 'highcharts.js', 'more.js', 'charts.js', 'chart.js', 'echarts.js', 'radialindial.js', 'city_data.js', 'layer.js', 'date.js', 'html2canvas.js', 'promise.js']
          return (
              module.resource &&
              /\.js$/.test(module.resource)  && arr.indexOf(fileName) != -1
          )
      }
    }),


    new webpack.optimize.CommonsChunkPlugin({
      name: 'tail',
      chunks: ['manifest'],
      minChunks: function (module, count) {
          const fileName = module.resource.substr(module.resource.lastIndexOf('/')+1).toLowerCase();
          const arr = ['radialindial.js', 'city_data.js', 'layer.js', 'date.js', 'html2canvas.js', 'promise.js']
          return (
              module.resource &&
              /\.js$/.test(module.resource)  && arr.indexOf(fileName) != -1
          )
      }
    }),


    new webpack.optimize.CommonsChunkPlugin({
      name: 'extra',
      chunks: ['tail'],
      minChunks: function (module, count) {
          const fileName = module.resource.substr(module.resource.lastIndexOf('/')+1).toLowerCase();
          const arr = ['layer.js', 'html2canvas.js', 'promise.js']
          return (
              module.resource &&
              /\.js$/.test(module.resource)  && arr.indexOf(fileName) != -1
          )
      }
    }),
```

> 在这里, 通过 CommonsChunkPlugin 的 name属性 添加了 6个 chunk, 每两个 chunk 之间都会存在包含关系, 对他们两个求交集, 那么你想要打包到一起的那一部分就单独出来了, 所以这个配置最终会提取出 5 个 chunk的公共子模块. 在提取公共模块的时候要清楚你需要引入的js的文件名是什么, 要保证2个模块之间的包含关系.

最终打包效果如下:

![打包js效果:](/images/CommonsChunkPlugin.png)


下面对细节做一些说明:

* `chunks` 参数是一个数组, 是你要和当前模块提取公共模块的模块.
* `module.resource` 是资源文件的完整路径名(包含文件名)
* minChunks最终的返回值 如果为 true的话, 将会提取到公共模块文件中, 这是深度提取的核心


有了 CommonsChunkPlugin和它的配置minChunks函数, 我们就可以用webpack随意的玩耍啦

( 完 )

本文参考
[webpack中文文档](https://doc.webpack-china.org/plugins/commons-chunk-plugin/)



转载请注明原地址，[本文首发于http://blog.nodejs.tech/](http://blog.nodejs.tech) 谢谢！






