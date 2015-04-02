### 变量对象
变量对象（简称为 VO）是与某个执行上下文相关的一个特殊对象，并储存了以下数据：
- 变量（var, VariableDeclaration）
- 函数声明（FunctionDeclaration, 缩写为FD）
- 函数形参

简单举例，可以用 ECMAScript 的对象来表示变量对象：
```js
VO = {};
```
VO 是执行上下文的一个属性：
```js
activeExecutionContext = {
    VO: {
        // 上下文中的数据 (变量声明（var）, 函数声明（FD), 函数形参（function arguments）)
    }
};
```
只有全局上下文中的变量对象可以通过 VO 的属性名间接访问（因为在全局上下文中，全局对象自身就是变量对象，稍候会详细介绍）。
在其他上下文中是不能直接访问 VO 对象的，因为它只是内部机制的一个实现（抽象的）。
当我们声明一个变量或函数时，就等于是在 VO 对象上添加了一个相应键/值的属性。

```js
var a = 10;
 
function test(x) {
    var b = 20;
};
test(30);
```
对应的变量对象：
```js
// 全局上下文中的变量对象
VO(globalContext) = {
    a: 10,
    test: <reference to function>
};
// “test” 函数上下文中的变量对象
VO(test functionContext) = {
    x: 30,
    b: 20
};
```

---

```js
AbstractVO (变量实例化过程中的通用行为)
 
  ║
  ╠══> GlobalContextVO
  ║        (VO === this === global)
  ║
  ╚══> FunctionContextVO
           (VVO === AO, <arguments> object and <formal parameters> are added)
```

---

### 全局上下文中的变量对象
> 全局对象是一个在进入任何执行上下文之前就创建的对象，此对象以单例的形式存在，它的属性在程序任何地方都可以访问，其生命周期随着程序的结束而终止。

全局对象创建时，Math、String、Date、parseInt 等属性也会同时被初始化，同样也可以附加其它对象作为属性，其中包括可以引用全局对象自身的属性。比如，BOM 中，全局对象上的 window 属性就指向了全局对象自身（但是，并非所有的实现都是如此）：
```js
global = {
    Math: <...>,
    String: <...>
    ...
    ...
    window: global
};
```

**全局上下文中的变量对象就是全局对象自身**
> 准确理解“全局上下文中的变量对象就是全局对象自身”是非常必要的，正是由于如此，在全局上下文中声明一个变量时，我们可以通过全局对象的属性间接访问到这个变量：
```js
a in window /// false
var a = 'found';
window.a // 'found'
```
---

### 函数上下文中的变量对象

在函数上下文中，变量对象（VO）不能直接被访问到，此时活动对象（Activation Object，简称 AO）扮演着 VO 的角色。
> 活动对象在进入函数上下文的时候被创建，同时伴随着 arguments 属性的初始化，该属性是 Arguments 对象的值：

```js
AO = {
    arguments:
};
```


---

### 处理上下文代码的几个阶段
>- 进入执行上下文
- 执行代码

>变量对象的修改和这两个阶段密切相关

###### 1. 计入执行上下文
当进入执行上下文时（在代码执行前），VO 就会被下列属性填充：
- 函数的所有形参（如果是在函数执行上下文中）
每个形参都对应变量对象中的一个属性，该属性由形参名和对应的实参值构成，如果没有传递实参，那么该属性值就为 undefined
- 所有函数声明（FunctionDeclaration, FD）
每个函数声明都对应变量对象中的一个属性，这个属性由一个函数对象的名称和值构成，如果变量对象中存在相同的属性名，则完全替换该属性。
- 所有变量声明（var, VariableDeclaration）
每个变量声明都对应变量对象中的一个属性，该属性的键/值是变量名和 undefined，**如果变量名与已经声明的形参或函数相同，则变量声明不会干扰已经存在的这类属性。**
- 函数表达式不会影响 VO

**注意：填充VO的顺序是: 函数的形参 -> 函数声明 -> 变量声明。
当变量声明遇到VO中已经有同名的时候，不会影响已经存在的属性**

```js
function test(a, b) {
    var c = 10;
    function d() {}
    var e = function _e() {};
    (function x() {});
}
 
test(10); // call
```
当进入 test 的执行上下文，并传递了实参 10，AO 对象如下：
```js
AO(test) = {
    a: 10,
    b: undefined,
    c: undefined,
    d: <reference to FunctionDeclaration "d">
    e: undefined
};
```
注意：AO 并不包含函数 x，这是因为 x 不是函数声明，而是一个函数表达式（FunctionExpression，简称为 FE），函数表达式不会影响 VO。
###### 2. 执行代码
此时，AO/VO 的属性已经填充好了。（尽管，大部分属性都还没有赋予真正的值，都只是初始化时候的 `undefined` 值）

继续以上一例子，到了执行代码阶段，AO/VO 就会修改为如下形式：
```js
AO['c'] = 10;
AO['e'] = <reference to FunctionExpression "_e">;
```
再次注意，函数表达式 _e 仍在内存中，它被保存在声明的变量 e 中。但函数表达式 x 却不在 AO/VO 中，如果尝试在其定义前或者定义后调用 x 函数，这时会发生“x未定义”的错误。未保存在变量中的函数表达式只能在其内部或通过递归才能被调用


---

### 关于变量
**使用 var 是声明变量的唯一方式**

如下赋值语句：
```js
a = 10;
```
仅仅是在全局对象时创建了新的属性（而不是变量）。“不是变量”并不是意味着它无法改变，而是指它不符合 ECMAScript 规范中的变量概念，所以它“不是变量”（它之所以能成为全局对象的属性，完全是因为 VO(globalContext) === global，大家还记得这个吧？）。

#### 两者区别：
```js
alert(a); // undefined
alert(b); // "b" is not defined
 
b = 10;
var a = 20;
```
进入执行上下文：
```js
VO = {
    a: undefined
};
```
我们看到，这个阶段并没有任何 b，因为它不是变量，b 在执行代码阶段才出现。（但是，在我们这个例子中也不会出现，因为在 b 出现前就发生了错误）

将上述代码稍作改动：
```js
alert(a); // undefined, we know why
 
b = 10;
alert(b); // 10, created at code execution
 
var a = 20;
alert(a); // 20, modified at code execution
```

**关于变量还有非常重要的一点：与简单属性不同的是，变量是不能删除的`{DontDelete} | [[Configurable]]`，这意味着要想通过 delete 操作符来删除一个变量是不可能的。**