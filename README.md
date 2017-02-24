# WECH (Wechat Component Helper)

## 微信小程序模块化组件开发“框架”

![https://raw.githubusercontent.com/chenzhuo1992/wech/master/screenshots/1.png](https://raw.githubusercontent.com/chenzhuo1992/wech/master/screenshots/1.png)
![https://raw.githubusercontent.com/chenzhuo1992/wech/master/screenshots/2.png](https://raw.githubusercontent.com/chenzhuo1992/wech/master/screenshots/2.png)

更新日志：[Changelog](https://github.com/chenzhuo1992/wech/blob/master/CHANGELOG.md) 

极轻量（9kb）的微信小程序模块化组件开发工具，没有任何构建相关的骨架或者约束。在运行阶段自动通过getter/setter，将你的“模块化组件”的数据和方法的映射到小程序的实际页面。支持组件嵌套、防止方法名污染、单向数据绑定、监听数据变化。

对wxss、wxml无任何新增语法规则，最大程度地避免引入沉重的轮子，防止小程序官方框架迭代产生的二次成本。

### 查看demo

效果demo：wechat-ide-binding-directory目录可以直接绑定为小程序开发目录，查看演示。

开发demo：demo目录演示了使用wech进行页面开发。本地通过 npm install / yarn 然后 npm run dev 进行调试。会动态更新wechat-ide-binding-directory。

备注：WECH并不要求必须使用demo中的gulp构建方式，这只是个演示。这只是个演示！

### 使用方式

1.把dist/widget.js拷到你的工程中；如果你的项目支持npm，可以使用import wech from 'wech'

2.开发组件时：

```

import wech from 'yourPath/widget.js'; // 可以npm引入的话

const yourConfig = {
    data: {
        // 组件私有数据
        district: '',
    },
    methods: {
        // 组件私有方法
        yourComponentMethod: function () {
            // this指向组件自身（下同）
        },
    },
    events: {
        // 私有模版事件响应
        yourComponentEvent: function () {},
    },
    watch: {
        // 监听组件私有数据的变化（在页面onLoad之后开始监听）
        yourComponentData: function (newValue, oldValue) {},
    },
    onLoad: function () {
        // 先触发页面的生命周期钩子，再触发组件的生命周期钩子
    }
};

module.exports = wech(yourConfig);

```

其中，可以通过 this.data.district / this.yourComponentMethod 访问组件内部的数据和方法，通过 this.setData 更新组件内部的数据，通过 this.$emit('eventName', data) 向外传递事件。模版中可以通过 bindtap="{{ yourComponentEvent }}" 来触发组件内部方法，防止全局事件名污染。

3.引入组件时，通过install／addTo方法。install用于将组件挂载到页面，addTo用于将组件作为子组件，挂载到另一个组件内部：

```

import child1 from 'child1';
const pageConfig = { 微信页面配置 };

child1.install(pageConfig, {
    scope: '这里的名字需要和<template is="component1" data="{{...scope1}}"></template>里面的scope1相符',
    static: {
        age: 25, // 传递给组件的静态参数，一般用于初始化、配置等，组件内部仍可以修改传入的参数（无绑定）
    },
    props: {
        // 传递给组件的动态数据，page的数据更新会同步至组件，而组件内部无法再修改（单向数据流）
        address () {
            return '北京市' + this.data.district + '区';
        },
    },
    events: {
        // 组件暴露到外部的事件
        changeCity (data) {
            this.setData({district: ''});
        }
    }
});

Page(pageConfig);

```

addTo同理，如下：

```

import wech from 'yourPath/widget.js';

import grandson from 'your Grand Component';

var conf = {}; // 子组件

grandson.addTo(conf, {
    scope: 'grandson',
    static: {},
    props: {},
    events: {},
    watch: {}
});

module.exports = widget(conf);

```

备注：这样挂载的是组件自身，如果同一组件同时用于页面多次，可以使用foo().install／foo().addTo进行实例化，避免引起冲突。foo()会立即创建一个foo的实例。

### 关于css和wxml

没有额外的要求或者改动，目的是保持轻量和便于适配微信官方后续的迭代。唯一的补充是，组件内的bindtap等事件可以通过花括号 {{ eventName }} 来绑定内部的方法。引入widget后会保证每个组件的事件名互不冲突。
