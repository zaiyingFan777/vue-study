#### Vue组件data为什么必须是个函数而Vue的根实例则没有此限制？

源码中找答案：src/core/instance/state.js-initData()

```js
function initData (vm: Component) {
  let data = vm.$options.data
  // 如果data是函数，则执行之并将其结果作为data选项的值
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}
}
```

src/core/util/options.js

```js
strats.data = function (
  parentVal: any,
  childVal: any,
  vm?: Component
): ?Function {
  if (!vm) {
    // 第一次触发这个逻辑是生成组件构造函数的时候(记住不是组件实例)，而不是vue根实例
    // 等vue根实例合并参数的时候不会走下面这个if循环,return mergeDataOrFn(parentVal, childVal, vm)
    // 这样作为vue根实例就很好地规避了下面的逻辑，所以既可以是函数也可以是对象
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

> 函数每次执行都会返回全新data对象实例

测试代码如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="lib/vue.js"></script>
</head>
<body>
    <div id="demo">
        <h1>Vue组件data为什么必须是个函数</h1>
        <comp></comp>
        <comp></comp>
    </div>
</body>
<script>
    Vue.component('comp', {
        // 错误见下面截图
        // data: {
        //     counter: 1
        // },
        data() {
            return {
                counter: 1
            }
        },
        template: `<div @click="counter++">{{counter}}</div>`,
    })
    // 创建实例
    const app = new Vue({
        el: '#demo'
    })
</script>
</html>
```

![截屏2020-01-22上午11.34.57](/Users/zaiyingfan-777/Desktop/截屏2020-01-22上午11.34.57.png)

> 程序甚至无法通过vue检测

##### 结论

vue组件可能存在多个实例，如果使用对象形式定义data，则会导致他们共用一个data对象，那么状态变更将会影响所有组件实例，这是不合理的；采用函数形式定义，在initData时会将其作为工厂函数返回全新data对象，有效规避多实例之间状态污染问题。而在vue根实例创建过程中则不存在该限制，也是因为根实例只能有一个，不需要担心这种情况。