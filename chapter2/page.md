# 页面配置

我们的页面配置其实是由controler层控制的，那我们将首先来介绍一下controler层的结构。每一个controler层我们都会要求暴露出来一个对象，其中必须要有以下几个字段。

## url

url字段可以是字符串，也可以是一个数组，其中的每一个元素都是路由匹配规则。具体可以参考[koa-router](https://github.com/alexmingoia/koa-router)。

## action

action是一个`es6`的`GeneratorFunction`，this指向koa的上下文。如果在实现中，挂载了`this.legoConfig`，后续`billund`就会根据你的配置生成页面。我们可以来看下支持的配置：

```
legoConfig - billund的配置

│── storeData - [Object][optional] - 整个页面共享的全局数据，也是全局store的state
│── storeConfig - [Object][optional] - 整个页面共享的Store的一些配置，可以在这里注册一些Vuex的getters
│── widgets - [Array][required] - 组件列表，每个元素字段如下:
│   │── name: [String][required] - 组件的名称
│   │── params: [Object][optional] - 传递给组件的参数，就是dataGenerator接到的参数
│   │── weight: [Number][optional] - 组件的权重，权重最高的一组组件称为高权重组件
│   │── requireParams: [String][optional] - 传递给组件的校验规则，默认是!undifined，还可以增加!null!''!0等校验，没有通过校验规则的组件会降级回到前端
│   │── index: [String][optional] - 排序规则 应该为'group.sub'形式，例如'001.001'，相同group的组件分为一组，同属于一个大容器，然后按照sub进行排序
│
│── options - [Object][optional] - 组件的自定义配置，允许自定义拓展(详见后续的[renderPlugins](/chapter4/renderplugin.html))，基础字段如下:
│   │── staticResources: [Array][required] - 组件引用的页面级静态资源，对应组件的代码也会自动合入这些资源
│   	 │── entry: [String][required] - 对应的js文件
│   	 │── styles: [String][optional] - 对应的css文件
│   │── pageTitle: [String][optional] - 页面的标题
│   │── backAutoRefresh: [Boolean][optional] - 检测到location的回退时自动刷新，主要用来解决app缓存html的问题，默认为false
│   
│── ...
```

一个常见的controler层应该是这样的:

```
function action*(){
	const cityId = this.query.cityid;
	const data = yield getData(cityId);

	this.legoConfig = {
		storeData: data,
		widgets: [{
			name: 'lego-widget-my-test-1',
			params: {
				cityId
			},
			weight: 1
		}, {
			name: 'lego-widget-my-test-2',
			params: data,
			requiredParams: [`configKey!null!''`]
		}],
		options: {
			staticResources: [{
                entry: 'node-demo-web/common.js',
                styles: 'node-demo-web/common.css'
            }, {
                entry: 'node-demo-web/index/demo.js',
                styles: 'node-demo-web/index/demo.css'
            }]
            pageTitle: 'billund-demo'
		}
	};
}

module.exports = {
	url: '/demo.:suffix',
	action	
};
```

### 首屏组件

对于用户来说，首屏的出现时间和体验密切相关。那么，对于`billund`，就是通过weight这个权重字段来确认一个页面有哪些组件是首屏组件。一个业务上的组件，权重最高的一组(没有写weight的视为0)将被认为是首屏组件。首屏组件将在node端进行渲染出html返回给浏览器，而非首屏组件将在浏览器端进行渲染。同时，在打包上我们也根据首屏组件进行了优化，非首屏组件的js将通过异步引入的方式加载进来。从而使需要加载的初始js变小。在[extract-text-webpack-plugin](https://github.com/webpack-contrib/extract-text-webpack-plugin/issues/455)修复了这个对应bug后，我们还能让首屏css变得更小，让首屏更快。

对于首屏组件来说，如果在服务端渲染失败，将会尝试在浏览器端重试渲染。

### 降级组件

通常来说，h5页面会需要投放多种环境，如native hybird，浏览器等。那么，有些参数在某些环境可能在Node端无法获取，需要依赖前端环境才可以获取。这个时候，我们可以配置requireParams校验规则，不符合规则的组件将会被降级回浏览器，等待参数充足的时候自动继续执行。我们可以通过[setSharedParams](/chapter3/browser-api.html#billundsetsharedparams)方法来设置参数，`billund`会自动校验参数是否已经满足。

### 静态资源

我们会将静态资源通过`staticResources`字段进行引入。`staticResources`是一个数组，其中每个元素的entry字段代表静态js，styles字段代表静态css，具体的路径应该是${你的项目名}/${相对地址}即可引用到对应的资源。我们会要求每一个页面需要引入一个页面级的静态资源，方便我们在打包时注入一些内容。一般来说，你需要自己实现一个[renderPlugin](/chapter4/renderplugin.html)，来进行staticResource -> 真实静态资源地址的映射转换。


