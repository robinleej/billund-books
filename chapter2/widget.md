# 组件

billund定位是一个模块化框架，组件无疑就是一个重要的组成部分。在billund的组件概念，组件是由数据生成逻辑(`dataGenerator`)、模板(`react`|`vue`)、store配置(`redux`|`vuex`)组成，是一个可以完整自运行的自洽组件。

组件通过一个配置文件将这些内容关联起来，配置文件可以理解为就是组件的入口。配置文件要求暴露出来一个对象，其中字段如下：

## name

组件的唯一标识，controler层通过`name`来引用组件。同一个项目内注册的组件的name不能重复。

### actions

暴露出去的action常量，让代码能更优雅，例如：

```
 - action.js

const BTN_CLICKED = 'billund-widget-test-widget/btn-clicked';

module.exports = {
	BTN_CLICKED
};


 - config.js
module.exports = {
	name: 'billund-widget-test-widget',
	actions: require('./action.js')	
};
```

### constants

希望暴露出去给外部使用的常量

## dataGenerator

dataGenerator的定义是组件的初始化数据逻辑，是一个`es6`的`GeneratorFunction`，有可能会同时运行在nodejs端和browser端。方法接收一个从`controler`传入的`object`参数，同时this指向对应的上下文(nodejs端是指向[koa](http://koa.bootcss.com/)的上下文，browser端是指向[前置处理器](/chapter2/project-config.html?q=#supportorpreprocessorjs)的上下文)。方法要求返回一个`object`结果，这个`object`会是传入模板的props，同时也是store的组件state，从而后续可以通过`redux`|`vuex`进行操作。一个常见的`dataGenerator`如下:

```
'use strict';

const REVIEW_URL = '/my/test/api.json';

/**
 * 获取数据
 *
 * @param {Object} context - 上下文
 * @param {Object} query - 查询对象
 * @return {Object}
 */
function* getReviews(context, query) {
	/*
		我们有一个fetch中间件，将它挂载了koa & billund-supportor的上下文上，从而可以通过调用
	 */
    return context.fetch(REVIEW_URL, {
        data: query
    }).then((json) => {
        if (!(json && json.respCode == 200 && json.data)) {
            throw new Error('invalid data');
        }
        return json.data.items || [];
    }).catch((e) => {
        console.log(e);
        return [];
    });
}

function* execute(params) {
    const query = {
        id: params.brandId
    };
    const reviews = yield getReviews(this, query);
    return {
        items: reviews
    };
}

module.exports = execute;
```

在执行过程中，如果发生了错误，可以直接抛出异常，`billund`会捕获这个异常，并且体现在[renderPlugin](/chapter4/renderplugin.html)的执行结果中。同时，如果这个组件是一个[首屏模块](/chapter2/page.html#首屏组件)，会自动在前端进行重试。

ps: 如果没有提供`dataGenerator`，我们会默认将`controler`传递的[params](/chapter2/page.html#action)直接传递给模板。

## template

`billund`的组件模板同时支持`React`和`Vue`，会通过文件名的后缀来自动识别。和所有同构框架一样需要注意的是，如果你在模板文件中引用了一个非同构的模块，请在组件对应的生命周期内(例如`React`的`componentDidMount` 或 `Vue`的`mounted`)引入。模板最后需要暴露出来一个模板对象(ReactElement|VueElement)。

## storeConfig

通常来说，组件都会存在交互行为，无论是响应其他组件的行为或者是自身的行为。我们默认根据组件的类型引入了对应的状态管理工具(React -> Redux,Vue -> Vuex)，从而能够通过单向数据流来减轻复杂交互的实现难度。我们的store中的state也是模块化的，除了全局的state，每个组件会掌握一个自身的state。我们可以通过改变这个自身的state来改变组件的展示，也可以通过`dispatch`一个`action`来向外部发出交互的通知。

`billund`支持通过配置一个`storeConfig`对象来设置对应的store。

ps:以下内容可能需要部分[Redux](http://redux.js.org/docs)或[Vuex](https://vuex.vuejs.org/zh-cn)的知识(React -> Redux,Vue -> Vuex，取决于您所使用的渲染框架)，如果您对他们不太了解，请先移步了解一下它们。

### React

React组件的storeConfig支持两个字段：

#### ownReducer

`ownReducer`是一个用来操作组件自身的`state`的`reducer`，概念和`Redux`的[combineReducer](http://redux.js.org/docs/api/combineReducers.html)非常相似。ownReducer会接收两个参数，state与action，其中`state`是组件自身的局部`state`，`action`是dispatch出来的action。ownReducer在需要更改组件state的情况下，要求返回一个新的对象，这个新的对象会成为组件新的state。

#### mapStateToProps

`mapStateToProps`就是`Redux`概念中的`mapStateToProps`。方法接收三个参数，分别是`state`(全局store的state)，`ownState`(组件自身的局部state)，`initialProps`(组件的初始数据，就是`dataGenerator`传递过来的对象)。

### Vue

我们通过Vuex的[modules](https://vuex.vuejs.org/zh-cn/modules.html)来实现了store的模块化，所以`storeConfig`支持以下字段：

#### actions

actions是一个对象，里面包括了组件注册需要响应的`action`。对于模块内部的 action，它接收两个参数，分别是`context`与`payload`。其中，`context.state` 是局部组件state，根节点的状态是 `context.rootState`，`payload`是dispatch action时携带的参数。

#### mutations

mutations是一个对象，里面包含了组件注册需要响应`mutation`。`mutation`接收两个参数，分别是`state`(组件的局部状态)与`payload`(commit时携带的参数)。

#### getters

getters是一个对象，里面包含了组件要注册的`getter`。`getter`同时接收三个参数，分别是`state`(组件的局部状态),`getters`(所有的getters),`rootState`(全局的state)。

#### state -> props

默认情况下，我们会将`template`中组件的`props`直接与组件对应`module`中对应的`state`对应，state变更会直接自动引起props的变更，不需要开发者手动关联。
