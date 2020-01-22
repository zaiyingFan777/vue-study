##### v-if和v-for哪个优先级更高？如果两个同时出现，应该怎么优化得到更好的性能？

源码中找答案compiler/codegen/index.js中genElement函数内

```js
if (el.staticRoot && !el.staticProcessed) {
    // 生成静态节点
    return genStatic(el, state)
} else if (el.once && !el.onceProcessed) {
    // v-once
    return genOnce(el, state)
} else if (el.for && !el.forProcessed) {
    // v-for这个逻辑先执行后执行v-if
    return genFor(el, state)
} else if (el.if && !el.ifProcessed) {
  	// v-if
    return genIf(el, state)
}
```

做个测试如下

```html
<p v-for="item in items" v-if="condition"></p>
```

做个测试如下

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>v-for与v-if</title>
    <script src="./lib/vue.js"></script>
</head>
<body>
    <div id="app">
        <h1>v-for和v-if谁的优先级高？应该如何正确使用避免性能问题</h1>
        <!-- 同时使用 -->
        <!-- <p v-for="child in children" :key="child.id" v-if="isFolder">{{child.title}}</p> -->
        <!-- 将v-if提到父级容器 -->
        <!-- <template v-if="isFolder">
            <p v-for="child in children" :key="child.id">{{child.title}}</p>
        </template> -->
        <!-- 利用计算属性 -->
        <p v-for="child in activeChildren" :key="child.id">{{child.title}}</p>
    </div>
</body>
<script>
    // 创建实例
    const app = new Vue({
        el: '#app',
        data() {
            return {
                children: [
                    {id: 1, title: 'foo', isShow: true},
                    {id: 2, title: 'bar', isShow: false}
                ]
            }
        },
        computed: {
            isFolder() {
                return this.children && this.children.length > 0;
            },
            activeChildren() {
                return this.children.filter(function(child) {
                    return child.isShow;
                })
            }
        },
    })
    console.log(app.$options.render)
</script>
</html>
```

两者同级时，渲染函数如下：

```js
(function anonymous(
) {
with(this){return _c('div',{attrs:{"id":"app"}},[_c('h1',[_v("v-for和v-if谁的优先级高？应该如何正确使用避免性能问题")]),_v(" "),_l((children),function(child){return (isFolder)?_c('p',{key:child.id},[_v(_s(child.title))]):_e()})],2)}
})
```

> _l包含了isFolder的条件判断，同时可以看出循环先执行，三目运算符后执行，从而得到优先级。

两者不同级时，渲染函数如下

```js
(function anonymous(
) {
with(this){return _c('div',{attrs:{"id":"app"}},[_c('h1',[_v("v-for和v-if谁的优先级高？应该如何正确使用避免性能问题")]),_v(" "),(isFolder)?_l((children),function(child){return _c('p',{key:child.id},[_v(_s(child.title))])}):_e()],2)}
})
```

> 先判断了条件再看是否执行_l

使用计算属性过滤一些不需要的，渲染函数如下

```js
(function anonymous(
) {
with(this){return _c('div',{attrs:{"id":"app"}},[_c('h1',[_v("v-for和v-if谁的优先级高？应该如何正确使用避免性能问题")]),_v(" "),_l((activeChildren),function(child){return _c('p',{key:child.id},[_v(_s(child.title))])})],2)}
})
```



##### 结论

  1.显然v-for优先于v-if被解析（把你是怎么知道的告诉面试官）

  2.如果同时出现，每次渲染都会先执行循环再判断条件，无论如何循环都不可避免，浪费了性能

  3.要避免出现这种情况，则在外层嵌套template，在这一层进行v-if判断，然后再内部进行v-for循环