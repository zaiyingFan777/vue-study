#### Vue组件中data选项为什么必须是个函数而Vue的根实例则没有此限制？

官方api：

> **一个组件的 `data` 选项必须是一个函数**，因此每个实例可以维护一份被返回对象的独立的拷贝。

当一个组件被定义，data必须声明为返回一个初始数据对象的函数，因为组件可能被用来创建多个实例。如果data仍然是一个纯粹的对象，则所有的实例将共享引用同一个数据对象！通过提供data函数，每次创建一个新实例后，我们能够调用data函数，从而返回初始数据的一个全新副本数据对象。

源码中找答案src/core/util/options.js

```js
strats.data = function (
  parentVal: any,
  childVal: any,
  vm?: Component
): ?Function {
  if (!vm) {
    if (childVal && typeof childVal !== 'function') {
      process.env.NODE_ENV !== 'production' && warn(
        'The "data" option should be a function ' +
        'that returns a per-instance value in component ' +
        'definitions.',
        vm
      )
      return parentVal
    }
    return mergeDataOrFn(parentVal, childVal)
  }
  return mergeDataOrFn(parentVal, childVal, vm)
}
```

##### 1.实例代码：

```html
<div id="app">
    <h1>Vue组件中data选项为什么必须是个函数而Vue的根实例则没有此限制？</h1>
    <!-- 根实例data -->
    <div>{{foo}}</div>
    <!-- 组件实例data -->
    <my-comp></my-comp>
    <my-comp></my-comp>
</div>
```

```js
Vue.component('my-comp', {
    data() {
      return {
        count: 0
      }
    },
    template: 
    `
    	<button @click="add">you clicked me {{count}} times.</button>
    `,
    methods: {
        add() {
          	this.count++;
        }
    },
})
```

上例子中，通过提供data函数，每次创建一个新实例后，各个组件实例使用其独立的data。

##### 2.为什么data必须是函数

  为什么组件中data必须是一个函数而不是对象，我们通过上面的例子来看组件是可以被复用的，我们通过注册一个全局组件来分析，我们假设先将data作为一个对象，我们前面说组件是可以被复用的，那么注册了一个组件本质上就是创建了一个组件构造器的引用，而真正当我们使用组件的时候才会去将组件实例化

```js
// 创建一个组件
var Component = function() {}
Component.prototype.data = {
  a: 1,
  b: 2
}
// 使用组件
var comp1 = new Component();
var comp2 = new Component();
comp1.data.b = 3;
comp2.data.b  // 3 
```

  我们可以发现当我们使用组件的时候，虽然data是在构造器的原型上被创建的，但是实例化的comp1和comp2确是共享同样的data对象，当你修改一个属性的时候，data也会发生改变，这明显不是我们想要的效果。

```js
// 创建一个组件
var Component = function() {}
Component.prototype.data = function() {
	return {
		a: 1,
		b: 2
	}
}
// 使用组件
var comp1 = new Component();
var comp2 = new Component();
comp1.data.b = 3;
comp2.data.b  // 2
```

  当我们的data是一个函数的时候，我们每个实例的data属性都是独立的，不会相互影响了，现在就可以知道为什么vue组件的data必须是函数了，是因为js特性带来的

  js本身的面向对象编程也是基于原型链和构造函数，应该会注意原型链上添加一般都是一个函数方法而不会去添加一个对象。