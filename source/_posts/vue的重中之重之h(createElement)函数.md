---
title: vue的重中之重之h(createElement)函数
date: 
tags: 
---

> 大家都知道render函数在vue中非常重要，但其实本质上执行渲染工作的是h函数，本质上也就是createElement函数！
--哲华施沃硕德

下面来看个简单例子：
箭头函数：

```
h=>h(App)
(function (h) {
    return h(App);
});
```
调用了一个名为h的函数，并且返回对App的处理结果，但是不够直观，看个更加直观的例子：
```
var h = function(x){
    console.log(x)
};
(function (h) {
    return h('App');
});
console.log(h('App'));
结果：App
```
https://segmentfault.com/q/1010000007130348
```
render: function (createElement){
    return createElement(app);
}
```
可见，在vuejs中，h函数仅是作为createElement函数之缩写，而render只是暴露给是开发者去使用createElement的钩子，因为本质上createElement是为了做渲染，因此笼统地称作是渲染函数也是可以的，只要心里清除背后真正在做事情的是createElement函数就好。

>将 h作为createElement的别名是Vue生态系统中的一个通用惯例，实际也是JSX所要求的，如果在作用域中h失去作用，在应用中会出发报错。

1.那么vue中的createElement到底是个什么？[参考](https://cn.vuejs.org/v2/guide/render-function.html#createElement-%E5%8F%82%E6%95%B0)
2.和Document.createElement()是什么关系？
3.和React.createElement()之间有何异同呢？
```
// @returns {VNode}
createElement(
  // {String | Object | Function}
  // 一个 HTML 标签字符串，组件选项对象，或者一个返回值
  // 类型为 String/Object 的函数，必要参数
  'div',

  // {Object}
  // 一个包含模板相关属性的数据对象
  // 这样，您可以在 template 中使用这些属性。可选参数。
  {
    // (详情见下一节)
  },

  // {String | Array}
  // 子节点 (VNodes)，由 `createElement()` 构建而成，
  // 或使用字符串来生成“文本节点”。可选参数。
  [
    '先写一些文字',
    createElement('h1', '一则头条'),
    createElement(MyComponent, {
      props: {
        someProp: 'foobar'
      }
    })
  ]
)
```
vNode是什么？
vNode是vue中的一个类，也就是数据队对象。https://github.com/vuejs/vue/blob/dev/src/core/vdom/vnode.js

深入理解data对象
```
{
  // 和`v-bind:class`一样的 API
  'class': {
    foo: true,
    bar: false
  },
  // 和`v-bind:style`一样的 API
  style: {
    color: 'red',
    fontSize: '14px'
  },
  // 正常的 HTML 特性
  attrs: {
    id: 'foo'
  },
  // 组件 props
  props: {
    myProp: 'bar'
  },
  // DOM 属性
  domProps: {
    innerHTML: 'baz'
  },
  // 事件监听器基于 `on`
  // 所以不再支持如 `v-on:keyup.enter` 修饰器
  // 需要手动匹配 keyCode。
  on: {
    click: this.clickHandler
  },
  // 仅对于组件，用于监听原生事件，而不是组件内部使用
  // `vm.$emit` 触发的事件。
  nativeOn: {
    click: this.nativeClickHandler
  },
  // 自定义指令。注意，你无法对 `binding` 中的 `oldValue`
  // 赋值，因为 Vue 已经自动为你进行了同步。
  directives: [
    {
      name: 'my-custom-directive',
      value: '2',
      expression: '1 + 1',
      arg: 'foo',
      modifiers: {
        bar: true
      }
    }
  ],
  // Scoped slots in the form of
  // { name: props => VNode | Array<VNode> }
  scopedSlots: {
    default: props => createElement('span', props.text)
  },
  // 如果组件是其他组件的子组件，需为插槽指定名称
  slot: 'name-of-slot',
  // 其他特殊顶层属性
  key: 'myKey',
  ref: 'myRef'
}
```
举一个例子：
```
    {
          title: 'Name',
          key: 'name',
          render: (h, params) => {
            return h('div', [
              h('a',{
                  class: {
                      'a-font-size': true
                  }
              }, params.row.name),
              h('p',{
                  class: {
                      'p-margintb': true
                  }
              }, '从 ' + params.row.gmtModified + ' 到 ' + params.row.endDate ),
              h('p',{
                  class: {
                      'p-marginb': true
                  }
              },'负责人：'+ params.row.users.name)
            ])
          }
        }
```
渲染结果是：
```
<div>
    <a class=“a-font-size">”Name"</a>
    <p class="p-margintb">从2016年7月1日到2018年3月19日</p>
    <p class=“p-marginb”>负责人：Frank</p>
</div>
```