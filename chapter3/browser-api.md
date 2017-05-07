# 浏览器端

在浏览器端，`billund`会更像一个状态管理组件，去统筹全局与组件之间的状态变更以及同步。

同时，因为`billund`同时支持`React`与`Vue`，所以在由不同的渲染框架构成的页面，`billund`会提供不同的`api`。

## 基本api

基本api是无论任何渲染框架，`billund`都会提供的方法。

### Billund.getState

获取整个页面级的`state`。`state`的来源多种多样，可能是来自于[storeData](/chapter2/page.html)，或者是通过与`widget`的交互得到的数据。

### Billund.registOnStartListener

注册`widget`的启动回调，当`widget`同构成功(React -> componentDidMount,Vue -> mounted)时，会回调这个方法。如果组件已经同构过，那么会直接调用这个方法。

#### 参数选项

- name

	String，`widget`的`name`。

- cb

	Function,同构成功时的回调，接收一个参数`props`，代表组件的当前数据。

### Billund.registOnFailListener

注册`widget`的失败回调。当调用方法后一段时间内，`widget`还没有同构成功，就会调用对应的回调。对于已经同构成功的`widget`，什么也不会发生。

#### 参数选项

- name 

	String，`widget`的`name`。

- cb

	Function,同构失败时的回调方法。

- options

	Object，可选参数。里面支持timeout字段，即设定从`registOnFailListener`开始到认定`widget`同构失败的时间长度。默认是2000ms。

### Billund.registOnChangeListener

注册`widget`的数据变更回调，当`widget`对应的`state`发生变更，就会调用这个回调。

#### 参数选项

- name 

	String，`widget`的`name`。

- cb

	Function,`widget`的`state`发生变更时调用的回调。接收三个参数，分别是`nextProps`(变更后的state)，`prevProps`(变更前的state),'initialProps'(组件启动时的state)。

### Billund.setSharedParams

当某些组件因为不满足[requireParams](/chapter2/page.html)(降级组件)的校验条件没有启动时，称之为降级组件。调用`Billund.setSharedParams`方法可以以`key`:`value`的方式设值，从而让组件触发再次校验，如果满足条件，就会开始执行组件的渲染逻辑。

#### 参数选项

- params

	Object，是一个`key` -> 'value'型的对象，其中`key`是对应参数的`key`，'value'就是我们要设置的对应值。

	example:

	`control`层

	```
	'use strict';

	function* action() {
    	this.legoConfig = {
        	widgets: [{
            	name: 'simple-vue-widget',
            	params: {
                	title: 'simple-vue-widget',
                	desc: 'test',
                	now: ''
            	},
            	requireParams: ['now!null!""'],
            	weight: 100
        	}],
        	options: {
            	staticResources: [{
                	entry: 'billund-example/common.js'
            	}, {
                	entry: 'billund-example/require-params.js',
                	styles: 'billund-example/require-params.css'
            	}]
        	}
    	};
	}

	module.exports = {
    	url: '/require-params.html',
    	action
	};
	```

	`browser-js(require-params.js)`:

	```
	'use strict';

	const Billund = require('billund');
	Billund.setSharedParams({
    	now: `time from browser:  ${new Date().valueOf()}`
	});
	```

### Billund.getWidgetBridgeOwnState

获取组件级别的`state`。

#### 参数选项

- widgetId

	组件在页面上的`唯一标识`。
	如果是一个`React`组件，那么可以在组件内部，通过`this.context.legoWidgetId`获取；
	如果是一个`Vue`组件，那么可以在组件内部，通过`this.$root.legoWidgetId`获取。

### Billund.hideWidgets

批量隐藏`widget`。

#### 参数选项

- names

	需要隐藏的`widget`的`name`数组。

### Billund.showWidgets

批量展示`widget`，需要在先调用`Billund.hideWidgets`后使用。

#### 参数选项

- names

	需要展示的`widget`的`name`数组。

## React Api

如果你的页面是由`React`渲染而成，那么我们会使用[Redux](https://github.com/reactjs/redux)来进行`widget`和页面之间的状态管理，建议先对`Redux`有所了解。

### Billund.decorateReducer

向全局`store`注册`reducer`。这个`api`是由`billund`提供的，与`Redux`提供的[replaceReducer](https://github.com/reactjs/redux/blob/master/docs/api/Store.md#replaceReducer)不同，注册新的`reducer`并不会覆盖之前的`reducer`，而是形成一条`reducer`调用链，顺序是从 `New Reducer` -> 'Old Reducer'以此进行处理。

#### 参数选项

- reducer

	`Redux`原生的[Reducer](https://github.com/reactjs/redux/blob/master/docs/basics/Reducers.md)

### Billund.dispatch

等同于`Redux`的[dispatch](https://github.com/reactjs/redux/blob/master/docs/api/Store.md#dispatch)，发出一个[action](https://github.com/reactjs/redux/blob/master/docs/basics/Actions.md)来唤起`Reducer`处理的流程