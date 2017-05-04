# webpack-loader

目前，`billund`提供了许多的`webpack`的`loader`，进行一些静态的代码转义，帮助开发者便利开发。

## billund-config-loader

`billund-config-loader` 主要用来将billund的组件的配置文件，根据我们对于的`loader-options`打出不同的内容。

### 全量打包(默认)

默认情况下，我们会对`配置文件`进行全量打包。配置时，`template`，`storeConfig`等字段对应的值为对应的路径，通过loader我们会将它转换成对应`require`引用，从而将组件进行完整的打包。

### 静态打包

在静态打包模式下，组件只会暴露出一些静态资源，如`name`(WIDGET_NAME)，`actions`，`constants`等字段。区分这两种打包模式的原因是，让非异步加载的包尽量小，优化性能。

## billund-action-loader

`billund-action-loader`主要会通过解析我们的`controler`层的代码，从而完成对应功能：


- 组件打包

获知我们要在页面上展示哪些组件，从而将组件打包进入对应的`页面静态资源文件`。同时，会通过组件的权重，自动优化成为首屏js与异步加载js，优化页面性能。

- storeConfig的同构注册

解析页面组件里的`storeConfig`配置，然后将其进行同构注册，从而让两端都拥有对应的能力(如`vuex`的`getters`等，就很推荐注册在这里)。

`billund-action-loader`支持两个参数：

- widgetNameToPath

`widgetNameToPath`是一个必传参数，内容是组件名称与组件完整路径的映射对象。在`controler`层中，我们都只注册了组件的名称，需要将组件打包的话需要有对应的路径引用。

- widgetLoaders

这个参数表示了开发者对`controler`层解析的组件，怎么对其的内容进行打包。这个参数不是必填的，默认我们会使用对应的`loaders`:

```
const DEFAULT_WIDGET_LOADERS = [{
    loader: 'babel-loader'
}, {
    loader: 'billund-config-loader',
    options: {
        include: 'all'
    }
}];
```

## billund-supportor-preprocessor-loader

`billund-supportor-preprocessor-loader`是将`preprocessor`的代码转换成能够让`billund`理解的注册代码，从而让开发者在使用时不用考虑太多细节。

