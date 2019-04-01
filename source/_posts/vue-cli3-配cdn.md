---
title: vue-cli3-配cdn
date: 2019-04-01 15:57:50
categories: ['前端'] 
tags: 前端
comments: true
---

### 一、chainWebpack
```javascript
const cdn = {
	css: [
		'https://cdn.xxxx.com/cdn/element-ui/2.7.0/index.css',
		'https://cdn.xxxx.com/cdn/quill/1.3.6/quill.snow.min.css',
		'https://cdn.xxxx.com/cdn/quill/1.3.6/quill.core.min.css',
		'https://cdn.xxxx.com/cdn/quill/1.3.6/quill.bubble.min.css',
	],
	js: [
		'https://cdn.xxxx.com/cdn/vue/2.6.10/vue.min.js',
		'https://cdn.xxxx.com/cdn/element-ui/2.7.0/index.js',
		'https://cdn.xxxx.com/cdn/vue-router/3.0.2/vue-router.min.js',
		'https://cdn.xxxx.com/cdn/moment/2.6.10/moment.min.js',
		'https://cdn.xxxx.com/cdn/vuex/3.1.0/vuex.min.js',
		'https://cdn.xxxx.com/cdn/vue-i18n/8.9.0/vue-i18n.min.js',
		'https://cdn.xxxx.com/cdn/lodash/4.17.11/lodash.min.js',
		'https://cdn.xxxx.com/cdn/antv-g2/3.4.10/g2.min.js',
		'https://cdn.xxxx.com/cdn/axios/0.18.0/axios.min.js',
		'https://cdn.xxxx.com/cdn/quill/1.3.6/quill.min.js',
	]
}
module.exports ={
	chainWebpack: config=>{
		if (process.env.NODE_ENV === 'production') { // 为生产环境修改配置...process.env.NODE_ENV !== 'development'
            config.plugin('html')
            .tap(args => {
                args[0].cdn = cdn;
                return args;
            })
        }
	}
}
```

### 二、configureWebpack
```javascript
module.exports ={
	configureWebpack: config=>{
		if (process.env.NODE_ENV === 'production') {
			// 'vue': 'Vue', key值需要去源代码里面找export
            config.externals = {
                'vue': 'Vue',
                'vue-router': 'VueRouter',
                'moment': 'moment',
                'vuex': 'Vuex',
                'element-ui': 'ELEMENT',
                'lodash': '_',
                'vue-i18n': 'VueI18n',
                '@antv/g2': 'G2_3',
                'axios': 'axios',
                'quill': 'Quill',
            }
        }
	}
}
```

### 三、修改 public/index.html
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <title>管理系统</title>
    <!-- 使用CDN的CSS文件 -->
    <% for (var i in htmlWebpackPlugin.options.cdn && htmlWebpackPlugin.options.cdn.css) { %>
    <link href="<%= htmlWebpackPlugin.options.cdn.css[i] %>" rel="external nofollow" rel="external nofollow" rel="preload" as="style">
    <link href="<%= htmlWebpackPlugin.options.cdn.css[i] %>" rel="external nofollow" rel="external nofollow" rel="stylesheet">
    <% } %>
    <!-- 使用CDN的JS文件 -->
    <% for (var i in htmlWebpackPlugin.options.cdn && htmlWebpackPlugin.options.cdn.js) { %>
    <link href="<%= htmlWebpackPlugin.options.cdn.js[i] %>" rel="external nofollow" rel="preload" as="script">
    <% } %>
  </head>
  <body>
    <noscript>
      <strong>We're sorry but 后台管理系统 doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
    </noscript>
    <div id="app"></div>
    <!-- built files will be auto injected -->
    <!-- 使用CDN的JS文件 -->
    <% for (var i in htmlWebpackPlugin.options.cdn && htmlWebpackPlugin.options.cdn.js) { %>
    <script src="<%= htmlWebpackPlugin.options.cdn.js[i] %>"></script>
    <% } %>
  </body>
</html>

```

### 关于vue-cli3热更新失效的 只需要把css分离插件 注释掉 便可解决
css: {
    extract: true
}
## 附上vue-cli3 vue.config.js 完整配置
```javascript
const path = require('path');
// const vConsolePlugin = require('vconsole-webpack-plugin'); // 引入 移动端模拟开发者工具 插件 （另：https://github.com/liriliri/eruda）
// const CompressionPlugin = require('compression-webpack-plugin'); //Gzip
// const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin; //Webpack包文件分析器
// const baseUrl = process.env.NODE_ENV === "production" ? "/static/" : "/"; //font scss资源路径 不同环境切换控制
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');

function resolve(dir) {
	return path.join(__dirname, dir);
}

const date = new Date()
const appendPath = `${date.getFullYear()}/${date.getMonth() + 1}/${date.getDate()}/`

const cdn = {
	css: [
		'https://cdn.xxxx.com/cdn/element-ui/2.7.0/index.css',
		'https://cdn.xxxx.com/cdn/quill/1.3.6/quill.snow.min.css',
		'https://cdn.xxxx.com/cdn/quill/1.3.6/quill.core.min.css',
		'https://cdn.xxxx.com/cdn/quill/1.3.6/quill.bubble.min.css',
	],
	js: [
		'https://cdn.xxxx.com/cdn/vue/2.6.10/vue.min.js',
		'https://cdn.xxxx.com/cdn/element-ui/2.7.0/index.js',
		'https://cdn.xxxx.com/cdn/vue-router/3.0.2/vue-router.min.js',
		'https://cdn.xxxx.com/cdn/moment/2.6.10/moment.min.js',
		'https://cdn.xxxx.com/cdn/vuex/3.1.0/vuex.min.js',
		'https://cdn.xxxx.com/cdn/vue-i18n/8.9.0/vue-i18n.min.js',
		'https://cdn.xxxx.com/cdn/lodash/4.17.11/lodash.min.js',
		'https://cdn.xxxx.com/cdn/antv-g2/3.4.10/g2.min.js',
		'https://cdn.xxxx.com/cdn/axios/0.18.0/axios.min.js',
		'https://cdn.xxxx.com/cdn/quill/1.3.6/quill.min.js',
	]
}
module.exports = {
	//基本路径 publicPath baseUrl
	publicPath: process.env.NODE_ENV === 'production' && process.env.APP_PATH ? process.env.APP_PATH + appendPath : '/',
	indexPath: './index.html',
	//输出文件目录
	outputDir: './dist',
	// eslint-loader 是否在保存的时候检查
	lintOnSave: false,
	// lintOnSave: process.env.NODE_ENV !== 'production',
	//放置生成的静态资源 (js、css、img、fonts) 的 (相对于 outputDir 的) 目录。
	assetsDir: 'static',
	//以多页模式构建应用程序。
	pages: undefined,
	//是否使用包含运行时编译器的 Vue 构建版本
	runtimeCompiler: false,
	//是否为 Babel 或 TypeScript 使用 thread-loader。该选项在系统的 CPU 有多于一个内核时自动启用，仅作用于生产构建，在适当的时候开启几个子进程去并发的执行压缩
	parallel: require('os').cpus().length > 1,
	//生产环境是否生成 sourceMap 文件，一般情况不建议打开
	productionSourceMap: false,
	// webpack配置
	//对内部的 webpack 配置进行更细粒度的修改 https://github.com/neutrinojs/webpack-chain see https://github.com/vuejs/vue-cli/blob/dev/docs/webpack.md
	chainWebpack: config => {
		/**
		 * 删除懒加载模块的prefetch，降低带宽压力
		 * https://cli.vuejs.org/zh/guide/html-and-static-assets.html#prefetch
		 * 而且预渲染时生成的prefetch标签是modern版本的，低版本浏览器是不需要的
		 */
		config.plugins.delete('prefetch');
		if (process.env.NODE_ENV === 'production') { // 为生产环境修改配置...process.env.NODE_ENV !== 'development'
			config.plugin('html')
			.tap(args => {
				args[0].cdn = cdn;
				return args;
			})
		}
		config.resolve.alias
		.set("vue$", "vue/dist/vue.esm.js")
		.set("@", resolve("src"))
		.set("src", resolve("src"))
		.set("assets", resolve("src/assets"))
		.set("components", resolve("src/components"))
		.set("pages", resolve("src/pages"))
		.set("base", resolve("src/components/base"))
		.set("Common", resolve("src/components/Common"));
	},
	//调整 webpack 配置 https://cli.vuejs.org/zh/guide/webpack.html#%E7%AE%80%E5%8D%95%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%B9%E5%BC%8F
	configureWebpack: config => {
		if (process.env.NODE_ENV === 'production') {
			config.externals = {
				'vue': 'Vue',
				'vue-router': 'VueRouter',
				'moment': 'moment',
				'vuex': 'Vuex',
				'element-ui': 'ELEMENT',
				'lodash': '_',
				'vue-i18n': 'VueI18n',
				'@antv/g2': 'G2_3',
				'axios': 'axios',
				'quill': 'Quill',
			}
			// 以及修改 public/index.html
		}
		//生产and测试环境
		let pluginsPro = [
			// new CompressionPlugin({ //文件开启Gzip，也可以通过服务端(如：nginx)(https://github.com/webpack-contrib/compression-webpack-plugin)
			// 	filename: '[path].gz[query]',
			// 	algorithm: 'gzip',
			// 	// test: new RegExp('\\.(' + ['js', 'css'].join('|') + ')$',),
			// 	test: new RegExp('\\.(js|css)$',),
			// 	threshold: 8192,
			// 	minRatio: 0.8,
			// }),
			//	Webpack包文件分析器(https://github.com/webpack-contrib/webpack-bundle-analyzer)
			// new BundleAnalyzerPlugin(),
			// 添加自定义代码压缩配置 压缩影响打包速度,增加访问速度
			new UglifyJsPlugin({
				uglifyOptions: {
					compress: {
						warnings: false,
						drop_debugger: true,
						drop_console: true,
					},
				},
				sourceMap: false,
				parallel: true,
			})
		];

		//开发环境
		/*let pluginsDev = [
			//移动端模拟开发者工具(https://github.com/diamont1001/vconsole-webpack-plugin  https://github.com/Tencent/vConsole)
			new vConsolePlugin({
				filter: [], // 需要过滤的入口文件
				enable: true // 发布代码前记得改回 false
			}),
		];*/
		if (process.env.NODE_ENV === 'production') {
			config.plugins = [...config.plugins, ...pluginsPro];
		} else {
			// 为开发环境修改配置...
			// config.plugins = [...config.plugins, ...pluginsDev];
		}
	},
	css: {
		// 启用 CSS modules
		modules: false,
		// 是否使用css分离插件 注釋掉 解决热更新失效问题
		// extract: true,
		// 开启 CSS source maps，一般不建议开启
		sourceMap: false,
		// css预设器配置项
		loaderOptions: {
			sass: {
				//设置css中引用文件的路径，引入通用使用的scss文件（如包含的@mixin）
				data: `$baseUrl:"/";`
				// @import '@/assets/scss/_common.scss';
				//data: `$baseUrl: "/";`
			}
		}
	},
// webpack-dev-server 相关配置 https://webpack.js.org/configuration/dev-server/
	devServer: {
		host: "0.0.0.0", //localhost
		port: 8910, // 端口号
		https: false, // https:{type:Boolean}
		open: false, // 配置自动启动浏览器
		hotOnly: true, // 热更新
		// proxy: 'http://localhost:8000'   // 配置跨域处理,只有一个代理
		/*proxy: { //配置自动启动浏览器
			"/rest/!*": {
				target: "http://172.16.1.12:7071",
				changeOrigin: true,
				// ws: true,//websocket支持
				secure: false
			}
		}*/
	},
// 第三方插件配置 https://www.npmjs.com/package/vue-cli-plugin-style-resources-loader
	pluginOptions: {
		'style-resources-loader':
			{//https://github.com/yenshih/style-resources-loader
				preProcessor: 'scss',//声明类型
				'patterns':
					[
						//path.resolve(__dirname, './src/assets/scss/_common.scss'),
					],
				//injector: 'append'
			}
	}
};



```
