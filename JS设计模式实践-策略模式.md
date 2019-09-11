# JS设计模式实践-策略模式
## 概念
设计模式的两个目的：
1. 将易变的代码和不易变的分开。
2. 尽可能地复用代码。
策略模式也是基于这两个目的，模式的实现可描述为：
1. 将一系列算法封装为一个个策略。
2. 将不易变的代码抽离出来。

## 实现过程
首先看一个例子。

```javascript
// index.js
useAward('ticket', params);

function useAward(type, params) {
    switch(type) {
        case 'ticket':
            ticket(params);
            break;
        case 'card':
            card(params);
            break;
        default:
            break;
    }
}
function ticket(params){}
function card(params){}
```

这份代码已经将不同的处理用函数的方式替代了，如果将封装的函数视为“策略”，那么我已经实现了策略模式的第一步——将一系列算法封装为一个个策略。但是这份代码还有一个缺点：如果新增处理函数，需要修改`useAward()`。
为了使得`useAward()`能够保持不变，可以尝试将所有策略函数放在一个字面量对象中，函数名作为key，函数作为value。
```javascript
const useAwardHandlers = {
    ticket() {},
    card() {},
    // .......
}
```
那么，`useAward()`就可以改造为通过入参去调用`useAwardHandlers`对应的属性了。
```javascript
function useAward(handler, params = {}) {
    if (typeof useAwardHandlers[handler] === 'function') {
        return useAwardHandlers[handler].call(this, params)
    } else {
        throw new Error('function is not defined in useAwardHandlers')
    }
}

useAward('ticket', params)
```

最好将`useAward()`和策略方法都抽离在一个模块。`useAwardHandlers`可以作为模块的私有变量，`useAward()`则暴露给外部调用。
```javascript
// useAwardHandlers.js
const useAwardHandlers = {
    ticket() {},
    card() {},
    // .......
}

export default function(handler, params = {}) {
    if (typeof useAwardHandlers[handler] === 'function') {
        return useAwardHandlers[handler].call(this, params)
    } else {
        throw new Error('function is not defined in useAwardHandlers')
    }
}
```

使用的时候如下。
```javascript
// index.js
import useAward from 'useAwardHandlers'

useAward.call(this, 'ticket', params)
```

值得注意的是`this`的指向。为了使得`this`指向调用方所在上下文的`this`，要通过`call()`层层改变函数内`this`的指向。

## JS中的策略模式
经历了上面的实现过程，可以总结为两个步骤：
1. 将一系列算法封装为一个个策略，用一个Map的结构（对象字面量）组织起来。
2. 对外暴露一个调用策略的方法，该方法直接从Map取出对应的策略执行。
以上步骤生成的模块，以后增加策略，也只是修改Map，不改调用策略的方法，属于代码中“不易变”的部分，这样就达到了将易变的代码和不易变的分开的目的。

## 在vue中使用策略模式
我日常使用vue开发，以上代码可以放在mixin。
```
// useAwardHandlers.js
const useAwardHandlers = {
    ticket() {},
    card() {},
    // .......
}

export default {
    methods: {
        useAward() {
            if (typeof useAwardHandlers[handler] === 'function') {
                return useAwardHandlers[handler].call(this, params)
            } else {
                throw new Error('function is not defined in useAwardHandlers')
            }
        }
    }
}
```

mixin之后，`useAward()`挂载到了vue实例，直接通过vue(`this`)实例调用。
```
// index.vue
this.useAward('ticket', params);
```
值得注意的是，mixin的方法，调用`useAward()`不在需要`call()`了，因为在vue实例化的时候，会在遍历methods时使用`call()`将函数的`this`绑定到vue实例。

## 更多的例子
### 校验方法抽象为一个个策略
```javascript
// validator.js
const validator = {
    city() {},
    street() {},
    houseNum() {},
    name(){},
    // ...
}

export default {
    methods: {
        validate(params, silent = false) {
            return Object.getOwnPropertyNames(params).every(key => {
                if (typeof validator[key] === 'function') {
                    return validator[key].call(this, params[key], silent)
                } else {
                    return true
                }
            })
        }
    }
}

// index.vue
this.validate({
  city: data1,
  street: data2,
  houseNum: data3,
})
```
