---
layout: post
title: "bind实现原理及在React中的应用"
date: 2018-02-05 
description: "bind基本使用和实现原理(polyfill)"
tag: JavaScript 
---   


> 58前端面试题: ES5的bind如何实现?

作为一个前端开发者, 我们对 `call`, `apply`应该不陌生了, 他们都可以绑定函数执行上下文, 然而它们有一个问题就是, 绑定作用域的时候就会调用这个函数, 不能很好的重复利用; 而ES5新增的`bind`函数可以让我们绑定作用域之后返回这个函数, 这样我们就可以在多个地方调用这个绑定指定作用域的函数了, 下面我们来聊一聊 bind 的基本使用和实现

## 基本语法
```text
func.bind(context[, arg1[, arg2[, ...]]])  
```
***参数***
* context: 要绑定的函数在运行时的this指向, 当使用`new操作符`调用绑定函数时，该参数无效
* arg1, arg2...: 调用this时传入的参数, 在调用绑定后的函数时, 作为`前置参数`使用


***返回值***
* 返回由指定的this值和初始化参数改造的原函数拷贝(返回一个新函数)

下面我们来看一个bind的应用

### bind应用: React事件处理器绑定this

在React项目中我们可以使用 bind来绑定this, 解决在事件函数中this不存在的问题
```text
import React, { Component } from 'react'

@HocXxx
export default class Test extends Component {

  constructor(props) {
    super(props)
    // 1.
    this.handleClick = this.handleClick.bind(this)
  }

  handleClick(e) {
    // 2.
    console.log(this)
  }

  render() {
    return (
      <div className="test" onClick={this.handleClick}>
        点击测试
      </div>
    )
  }
}
```

位置2 处, 我们打印出了 handleClick事件处理函数中的 this, 此时的this正确的指向这个Test组件实例, 如果我们不使用位置 1处的 `bind` 绑定this, 在 handleClick中的this将会是undefined, 这个时候非常不方便我们使用了(当然react绑定事件还可以直接使用箭头函数, 不过这个不在本文的探讨范围之内)

当然, bind的应用场景还有很多, 在此不列举出所有.

### 知根知底, 探索 bind实现原理
bind是 ES5新增的, 在一些老旧浏览器不支持, 那么我们能不能在已有call和apply基础上实现一个 bind函数呢?
当然可以!
下面来研究一下 bind 的一个实现, 这个也是一个58前端工程师的面试题

***在实现 bind 之前, 我们首先要明白我们的基本需求***
* 1.检测bind是否已经存在, 不存在才创建 bind
* 2.给所有的函数拓展一个 bind 方法
* 3.bind绑定时候传入 context, args等, 需要处理参数
* 4.返回一个新函数
* 5.调用新函数的时候, 需要让新函数在指定context下执行
* 6.新函数需要拷贝源函数的原型
* 7.新函数调用时传入的参数, 要和源函数的参数合并

***针对上面的需求, 我们可以一一制定实现思路***
* 1.bind函数是所有函数都可以调用的, 所以要从原型上检测是否存在 bind
* 2.函数也是一个对象, 它的构造器是 Function类型, 要让所有的函数都具有bind方法, 我们应该使用 Function.proptype.bind = func来创建  bind
* 3.对于调用bind传入的参数,我们在函数中可以使用arguments(伪数组)来获取, 利用数组的原型方法slice处理arguments对象
* 4.我们需要构造一个新函数 
* 5.使用apply可以让我们绑定this,但是需求只能在调用新函数的时候才执行, 所以我们返回的新函数的返回值必须是一个调用apply的函数, 而不是直接用apply调用
* 6.函数的原型拷贝, 为了不因为引用类型的修改相互影响, 所以我们可以使用一个空函数, 然后创建一个实例来解决 prototype的指向问题
* 7.新函数调用时传入的参数可能也是多个, 我们也可以使用 arguments和数组原型的slice处理, 最后把两次调用的参数合并到数组中


下面是一个 polyfill对 bind的实现 (https://raw.githubusercontent.com/inexorabletash/polyfill/master/polyfill.js)

```text

// https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Function/bind
if (!Function.prototype.bind) {
  Function.prototype.bind = function (o) {
    if (typeof this !== 'function') { throw TypeError("Bind must be called on a function"); }

    var args = Array.prototype.slice.call(arguments, 1),
        _self = this,
        nop = function() {},
        bound = function () {
          return _self.apply(this instanceof nop ? this : o,
                            args.concat(Array.prototype.slice.call(arguments)));
        };

    // Function.prototype.__proto__ === Object.prototype   ---> true, Object.prototype__proto__ = null
    if (this.prototype)
      nop.prototype = this.prototype;
    bound.prototype = new nop();
    return bound;
  };
}

```

核心分析: 在bind函数实现中, 为了延迟调用apply,我们需要返回一个函数, 在函数中调用 apply; 给  Function.prototype拓展bind方法

ps: 代码虽少, 可五脏俱全, 很考验js基础知识, 只有我们的基础很扎实才可能答出这道题
ps: 以上并非浏览器的官方实现, 根据ECMA-262标准, 还有很多细节问题

参考文献:

[ECMA-262-bind](https://www.ecma-international.org/ecma-262/5.1/#sec-15.3.4.5)

[github-polyfill](https://raw.githubusercontent.com/inexorabletash/polyfill/master/polyfill.js)

JavaScript高级程序设计





转载请注明原地址，[本文首发于http://blog.nodejs.tech/](http://blog.nodejs.tech) 谢谢！
