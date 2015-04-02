### 对象
> 对象是属性的集合，并有另一个对象或者 null 作为其`prototype` (原型), 由内部属性 `[[Prototype]]` 指向。(`[[Prototype]]`是内部属性，在脚本中无法访问；在 Firefox, Safari, Chrome 的实现中，支持为 `__proto__`，默认指向构造函数的原型对象)，

```js
var foo = {
  x: 10,
  y: 20
};
```
![](images/basic-object.png)
---
### 原型链 (prototype chain)
> - 原型对象也是简单对象，可以拥有自己的原型。以此递推，形成原型链。原型链终止于 `__proto__ === null`
> - 原型链是一个用来实现继承和共享属性的有限对象链

属性\方法 查找过程：


```js
var a = {
  x: 10,
  calculate: function (z) {
    return this.x + this.y + z
  }
};
var b = {
  y: 20,
  __proto__: a
};
var c = {
  y: 30,
  __proto__: a
};
// call the inherited method
b.calculate(30); // 60
c.calculate(40); // 80
```
![](images/prototype-chain.png)

---
###### 原文：[Javascript. The core.](http://dmitrysoshnikov.com/ecmascript/javascript-the-core/) by _[Dmitry Soshnikov](http://dmitrysoshnikov.com/)

###### 参考译文: 

- [JavaScript核心](http://dmitrysoshnikov.com/ecmascript/javascript-the-core/)  by _[JeremyWei](http://weizhifeng.net/tech.html)_
