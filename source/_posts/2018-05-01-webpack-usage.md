---
title: webpack笔记
tags: webpack
---
<p>项目使用了vue单文件组件模式，webpack的版本升级为4，为了实现某些需求需要再格外配置。</p>
<p>1.提取css文件 [mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)</p>
<!-- more -->
```javascript
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
module.exports = {
  plugins: [
    new MiniCssExtractPlugin({
      // Options similar to the same options in webpackOptions.output
      // both options are optional
      filename: "[name].css",
      chunkFilename: "[id].css"
    })
  ],
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          MiniCssExtractPlugin.loader,
          "css-loader"
        ]
      }
    ]
  }
}
```
<p>为了压缩css，在webpack4中还得自行配制插件 [optimize-css-assets-webpack-plugin](https://github.com/NMFR/optimize-css-assets-webpack-plugin)</p>
```javascript
const OptimizeCSSAssetsPlugin = require("optimize-css-assets-webpack-plugin");
module.exports = {
  optimization: {
    minimizer: [
      new OptimizeCSSAssetsPlugin({})
    ]
  },
  ...
```
<p>2.压缩JS [uglifyjs-webpack-plugin](https://github.com/webpack-contrib/uglifyjs-webpack-plugin)</p>
<p>在webpack4中，如果继续使用webpack.optimize.UglifyJsPlugin自带插件会提示警告，应该单独引入插件后在optimization.minimizer中来配置，和压缩css一样。</p>
```javascript
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
optimization: {
	minimizer: [
      	new UglifyJsPlugin({
	        cache: true,
	        parallel: true,
	        sourceMap: true // set to true if you want JS source maps
	    })
	］
}
```
<p>3.图片精灵 [webpack-spritesmith](https://www.npmjs.com/package/webpack-spritesmith)</p>
<p>把所有icon放在路径src.cwd指定的路径下</p>
```javascript
const SpritesmithPlugin = require('webpack-spritesmith');

new SpritesmithPlugin({
    src: {
        cwd: path.resolve(__dirname, 'src/assets/ico'),
        glob: '*.png'
    },
    target: {
        image: path.resolve(__dirname, 'src/assets/spritesmith-generated/sprite.png'),
        css: path.resolve(__dirname, 'src/assets/spritesmith-generated/sprite.styl')
    },
    apiOptions: {
        cssImageRef: "~sprite.png"
    }
})
```
<p>安装stylus-loader处理styl类型文件</p>
```javascript
module: {
    rules: [
      {
        test: /\.styl$/,
        use: [
          'style-loader',
          'css-loader',
          'stylus-loader'
        ],
      },
    ...
```
<p>新建一个styl文件，引入sprite.styl，用sprite()指定icon名称的方式来实现图片精灵。</p>
{% codeblock lang:css %}
@import './spritesmith-generated/sprite.styl'
.icon-satellite
	sprite($icon_satellite)
{% endcodeblock%}
4.懒加载模块
<p>项目很大时，使用懒加载，按需载入，例如每个路由对应的组件。</p>
```javascript
const home = () => import(/* webpackChunkName: "home" */ 'pages/home/index.vue');
{
  path: '/',
  name: 'home',
  meta: {title:'首页'},
  component: home
}
```
5.跨域调用api
```javascript
devServer: {
    port: 3003,
    proxy: {
        '/api': {
          target: 'http://localhost:8088'
        }
    }
}
```
这样，我们运行项目端口在3003，但如果请求接口包含 */api* 都会指向端口8088。
















