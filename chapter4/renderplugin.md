# renderPlugin

`renderPlugin`，顾名思义就是可以让开发者能够以编程的方式掌控页面的局部渲染的插件。在`billund`的体系中，为了便利开发，页面中的大部分内容是由`billund`执行生成的，但是难免存在有开发者对于个性化页面内容的需求。那么，只要通过实现和配置`renderPlugin`，就能很好的覆盖这一部分个性化需求。
renderPlugin将在`服务器端`生成渲染html时调用，此时`widget`都已经执行。

## 开发

`renderPlugin`需要满足以下几点要求：

- `renderPlugin`必须是一个`GeneratorFunction`

- `renderPlugin`执行返回的结果是一个`Object`，其中的`result`字段是一个字符串，就是会添加到页面上对应位置的`html`内容。

### 参数选项

renderPlugin必须暴露一个`GeneratorFunction`，函数的传参如下：

- config - 当前的执行配置，有以下几个字段:
	
	storeData: [Object], // 传给前端的全局数据,

    widgets: [Array], // widgets的配置结果，里面还包括了执行结果，在result字段中

    mostImportantWidgets: [Array], // 重要的widget执行模块

    executeResults: [Object], // 模块执行的结果,有success和fail两个字段,分别对应数组，数组元素就是widget

    staticResources: [Array], // 引用的静态资源,以entry和styles为区分

    options: [Object], // 对应的可选配置，可以自由拓展，也就是在`action`层时的`legoConfig`里的`options`

- context - 当前koa的上下文


在我们的脚手架生成的项目中，就有给出一个简单的`renderPlugin`的示例:

```
'use strict';

/**
 * 分割静态资源,分割为entry与style(js & css)
 *
 * @param  {Array} staticResources - 静态资源列表
 * @return {Object}
 */
function splitResources(staticResources) {
    staticResources = staticResources || [];
    const entries = [];
    const styles = [];

    staticResources.forEach((resource) => {
        if (resource.entry) {
            entries.push(resource.entry);
        }
        if (resource.styles) {
            styles.push(resource.styles);
        }
    });
    return {
        entries,
        styles
    };
}

/**
 * 解析样式标签
 *
 * @param  {Array} styles - 样式文件
 * @return {Array}
 */
function parseStyles(styles) {
    return (styles || []).map((style) => {
        return `<link rel="stylesheet" href="//localhost:9074/app/${style}" type="text/css">`;
    });
}

/**
 * 解析静态资源文件
 *
 * @param  {Array} entries - 静态资源文件
 * @return {Array}
 */
function parseEntries(entries) {
    return (entries || []).map((entry) => {
        return `<script defer="true" src="//localhost:9074/app/${entry}"></script>`;
    });
}

/**
 * billund的render-plugin,我们只是提供一个简单的实现，你应该根据你的具体情况进行实现
 *
 * @param {Object} config - billund提供的渲染配置
 * @return {Object}
 */
function* parse(config) {
    if (!(config && config.staticResources)) {
        return {
            result: ''
        };
    }

    const splitedResources = splitResources(config.staticResources);

    const parsedStyles = parseStyles(splitedResources.styles) || [];
    const parsedEntries = parseEntries(splitedResources.entries) || [];

    return {
        result: (parsedStyles.concat(parsedEntries)).join('')
    };
}

module.exports = parse;
```

ps: 以上示例代码是我们为您提供的样例`staticResource` -> '静态资源地址'的一个转换，真实的路径逻辑请您按照真实情况具体实现。

