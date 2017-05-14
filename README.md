# billund

当下，前端技术发展迅猛，有许多新技术崭露头角，建立起了强大的社区，例如`React`和`Vue`。它们在以新的开发思路席卷前端界时，也支持了`Server-Side-Render`，让为用户体验孜孜以求的前端们提供了便利。逐渐，`同构`这个响亮的名词被大家逐步接受，也被视为是一个提升用户体验的有力武器。

但是，在真实的开发过程中，我们逐步发现了许多令人不快的地方：

- 同构的流程比较复杂

- 在一个大团队内，对`React`和`Vue`的看法众口难调，开发者都希望能使用自己更为熟悉的框架

- `Server-Side-Render`开发过程中，如果某个获取数据逻辑耗时较长，还需要考虑将那一块的html渲染滞后以免影响用户体验，却让开发体验不尽相同

针对以上痛点，`billund`逐渐从业务项目中脱胎而出，成为一个同构组件，致力于为用户和开发和都提供更好的体验：

- 屏蔽了复杂的同构流程

- 对`React`和`Vue`兼而并包

- 将组件的开发 和 在页面的展示进行分离，组件的开发体验一致，通过页面上的简单配置就可以决定组件是否延迟在浏览器端再渲染，同时带有组件错误重试机制，能够在服务端出现问题的时候有所补救。

再次感谢`React`、`Vue`、`Koa`、`webpack`等优秀的前端技术，正在这些新技术让当今的前端世界变得如此美好。

如果您在使用过程或阅读过程中有任何不快的地方，欢迎[吐槽](https://github.com/robinleej/billund/issues)。

* [安装](chapter1/README.md)
* [介绍](chapter2/README.md)
    * [项目结构](chapter2/project-config.md)
    * [组件](chapter2/widget.md)
    * [页面配置](chapter2/page.md)
    * [webpack-loader](chapter2/loader.md)
* [API文档](chapter3/README.md)
    * [服务端](chapter3/server-api.md)
    * [浏览器端](chapter3/browser-api.md)
* [高级配置](chapter4/README.md)
    * [renderPlugin](chapter4/renderplugin.md)
