# 安装

## 环境要求

### nodejs

要求 node >= 4.2.0

### browser

`billund`同时支持`Vue`和`React`进行渲染，浏览器的兼容性取决于你所选用的渲染框架以及对应的版本。

参考：

Vue.js 不支持 IE8 及其以下版本，因为 Vue.js 使用了 IE8 不能模拟的 ECMAScript 5 特性。 Vue.js 支持所有兼容 [ECMAScript 5](http://caniuse.com/#feat=es5) 的浏览器。

React >= 15.0.0 后， 不支持 IE8 及其以下版本。之前的版本能够兼容IE8。

## 更新日志

每个版本的更新日志见[GitHub](https://github.com/robinleej/billund)。

## 脚手架

我们提供了一个基础脚手架，从而能够帮助快速搭建项目。

- 如果没有安装[yeoman](http://yeoman.io/) , 请先安装

        $ npm install -g yo

-  安装脚手架

        $ npm install -g generator-billund

- 创建项目 , 根据提示完成初始化

        $ mkdir my-project && cd my-project

        $ yo billund

- 运行项目

        $ npm i

        $ npm run dev

        $ npm run start:browser (另外一个命令行窗口)

- 打开 [react-demo页](localhost:8080/simple-react.html)

- 打开 [vue-demo页](localhost:8080/simple-vue.html)  

## 开发支持

### [Controler](/chapter2/project-config.html)层修改代码自动Reload    