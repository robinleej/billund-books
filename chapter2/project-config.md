# 项目结构

对于一个`billund`生成的页面，我们需要有以下几层：

- controler层

	用以控制整个页面的结构配置，以及定义该页面的路由。

- 组件注册文件

	用以声明项目需要使用哪些组件。

- 静态资源

	页面需要引用的静态js & css等

- 组件

	用以组成页面的组件，可以跨页面或跨页面复用。

一般来说，我们的项目结构是这样的：

```
app - 应用的根目录
│── action - controler层
│   │── actionA.action.js - 页面A
│   │── actionB.action.js - 页面B
│   └── ...
│── config - 配置文件夹
│   │── widgets - 项目的组件注册文件
│── entries - 静态资源文件(页面级)
│── widget - 组件
│   │── widgetA
│   │── widgetB
│   └── ...
│── ...
```

对于在使用脚手架中生成的项目，会在package.json里中看到如下的字段`legoconfig`，其中的字段有：

- actiondir

	controler层所在的文件夹

- actionRegExp

	字符串形式的正则表达式，文件名匹配的才是`billund`可以加载的controler文件

- staticjsdir

	静态资源js所在的文件夹

- widgetconfig

	组件注册声明文件的相对地址

- serverdist

	`node`端的组件的打包地址

- commonchunkname

	页面通用的静态资源的文件名称，具体请参考`webpack`的[CommonsChunkPlugin](https://webpack.js.org/plugins/commons-chunk-plugin/)

- supportorpreprocessor

	[前端预置处理器](todo)的位置

- vuechunkname

	项目内vue静态资源打包后的名称

- reactchunkname

	项目内react静态资源打包后的名称

