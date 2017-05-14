# 服务端

`billund`在服务端时，充当是一个中间件的职责，是一个执行流程组件。对应的`api`也会与此紧密相关。

## Billund.init

```
const koa = require('koa');
const Billund = require('billund');

const app = koa();

co(function*() {
    app.use(Billund.init({ ...options }));
```

`Billund.init`会返回一个`GeneratorFunction`的中间件函数，从而`koa`可以将`billund`注册成自身的一个中间件。

### Billund.init 参数选项

- actionDir

	必填，是一个绝对路径，`billund`的`controler`所在的文件夹，`billund`会结合参数`actionNameRegex`遍历`actionDir`下的所有文件，将文件名匹配`actionNameRegex`作为`billund`的`controler`文件注册。

- actionNameRegex

	必填，正则表达式，与参数`actionDir`搭配使用，遍历`actionDir`下面的所有文件，将文件名匹配`actionNameRegex`作为`billund`的`controler`文件注册。

	```
	Billund.init({
        actionDir: path.resolve(__dirname, './action'),
        actionNameRegex: new RegExp('.(action).(js)$'),
        ...
    });
	```
- widgetDir

	必填，是一个绝对路径，`server`端的`widget`(通过webpack打包后)所在的文件夹，`billund`会结合参数`widgetNameRegex`，遍历`widgetDir`下所有文件，将文件名匹配`widgetDir`作为`billund`的server端组件进行注册。

- widgetNameRegex

	必填，是一个正则表达式，与参数`widgetNameRegex`搭配使用，遍历`widgetDir`下所有文件，将文件名匹配`widgetDir`作为`billund`的server端组件进行注册。

	```
	Billund.init({
        ...
        widgetDir: path.resolve(__dirname, './serverdist'),
        widgetNameRegex: /\.(js)$/,
        ...
    });
	```
- vendors

	必填，是一个`Object`。`billund`同时支持'React'与'Vue'的渲染，在浏览器端使用的时候，会自动根据页面的组成引入不同的`lib`库。那么，`vendors`字段就是用来提供对应的lib库。`vendors`字段支持两个key,分别是`react`与`vue`，内容与[staticResource](/chapter2/page.html)的格式相同。
	范例：

	```
	Billund.init({
        ...
        vendors: {
            react: 'billund-example/react.js',
            vue: 'billund-example/vue.js'
        },
        ...
    });
	```

- renderPlugins

	选填，是一个`Object`，分别是`header`和`body`，分别对应了`renderPlugin`渲染出来的内容所插入的位置。
	[renderPlugin的详细概念](/chapter4/renderplugin.html)。

## Billund.updateWidgets

```
webpackServerCompiler.watch({}, (err, stats) => {
    ....
    // 提示Billund删除自己cache的内容
    Billund.updateWidgets();
});
```

更新`服务端`的`widget`内容，一般与`webpack`的watch功能搭配使用。当发现`widget`源代码发生变更，即可调用这个方法更新`服务端`的`widget`。



