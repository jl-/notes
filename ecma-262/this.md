### 定义
`this` 是执行上下文的一个属性：
```js
activeExecutionContext = {
  VO: {...},
  this: thisValue
};
```
`this` 与上下文的可执行代码类型紧密相关，其值在进入上下文阶段就确定了，并且在执行代码阶段不能被改变。

---

### 全局代码中的 `this`
全局代码中的 `this` 非常简单，this 始终是全局对象自身，因此，可以间接获取引用
```js
// 显式定义全局对象的属性
this.a = 10; // global.a = 10
alert(a);    // 10
 
// 通过赋值给不受限的标识符来进行隐式定义
b = 20;
alert(this.b); // 20
 
// 通过变量声明来进行隐式定义
// 因为全局上下文中的变量对象就是全局对象本身
var c = 30;
alert(this.c); // 30
```

---

### 函数代码中的 `this`
- 函数代码中的 `this` 的值是进入执行上下文阶段确定的，并非静态绑定在函数上，其值可能每次都不一样。
- 一旦进入代码执行阶段，其值就维持不变了。也就是说，要给 `this` 赋一个新值是不可能的，因为 `this` 根本就不是一个变量

##### 影响 `this` 值的因素
首先，在通常的函数调用时，`this` 是由激活上下文代码的调用者（`caller`）决定的，即调用函数的父级上下文。并且 **`this` 的值是由调用表达式的形式决定的（换句话说就是，由调用函数的语法决定）**。

>了解并记住这点非常重要，这样才能在任何上下文中都能准确判断 this 的值。更确切地讲，调用表达式的形式（或者说，调用函数的方式）影响了 this 的值，而不是其他因素。

```js
function foo() {
    alert(this);
}
 
foo(); // global
 
alert(foo === foo.prototype.constructor); // true
 
// 然而，同样的函数，以另外一种调用方式的话，this的值就不同了
 
foo.prototype.constructor(); // foo.prototype
```
```js
var foo = {
  bar: function () {
    alert(this);
    alert(this === foo);
  }
};
 
foo.bar(); // foo, true
 
var exampleFunc = foo.bar;
 
alert(exampleFunc === foo.bar); // true
 
// 同样地，相同的函数以不同的调用方式，this的值也就不同了
 
exampleFunc(); // global, false
```

---

---

============= 那么，究竟调用函数的方式是如何影响 `this` 的值？为了完全弄懂其中的奥妙，首选需要了解一种内部类型 - 引用（Reference）类型 =============

### 引用类型
引用类型可以用伪代码表示为拥有两个属性的对象：`base`（即拥有属性的那个对象），和 `base` 中的 `propertyName` 。
```js
var valueOfReferenceType = {
    base: ,
    propertyName: 
};
```
引用类型的值只有可能是以下两种情况：
- 当处理一个标识符的时候
- 或者进行属性访问的时候

标示符的处理过程在第四章 作用域链中讨论；在这里我们只需要知道，使用这种处理方式的返回值总是一个引用类型的值（这对 `this` 来说很重要）。
