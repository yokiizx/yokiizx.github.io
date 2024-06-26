# 老生常谈 -- 原型链和继承


![](https://cdn.jsdelivr.net/gh/yokiizx/picgo@main/img/prototype.png)

## 正如上图

理解了上面的图，就理解了原型链。(如果你看不见图，用个梯子试试~)

一个函数创建的时候就会同时创建一个`prototype`属性和`constructor`属性。
其中`prototype`指向原型对象，实际上指向隐藏特性`[[prototype]]`；`constructor`指回构造函数。

如果把实例的原型看作是另一个类型的实例，那么原型对象的内部就会有一个指针指向另一个类型，同理下去，就会在实例和原型之间构造了一条原型链，直到 null 为止。

## 七大继承方式

### 原型链继承

这种继承是直接把子类的原型对象修改为父类的实例。

```js
function Super(father) {
  this.father = father
}
Super.prototype.getSuper = function() {
  return this.father
}
function Sub(son) {
  this.son = son
}
Sub.prototype = new Super()
// 解释：
// 1. Sub.prototype.__proto__ = Super.prototype ==> 其实就是修改了子类的原型对象指针，让原型链搜索的时候去Super的原型上去搜索.
// 2. Sub.proto.constructor = Super.prototype.constructor
// ...把其他变量/方法加到修改后的Sub.prototype上
```

直接原型链继承一般不会单独使用，有一些问题需要面对：

1. 如果父类中有引用类型数据，当子类实例对其进行修改的时候，会影响到所有子类的实例
2. 子类实例化的时候无法给父类构造器传参

> 一个注意点，如果用字面量直接修改 Sub.prototype，会导致之前的继承失效。

### 盗用构造函数继承

在子类构造函数中调用父类构造函数，解决了原型链继承的问题

```js
function Super(name) {
  this.name = name
}
// 可以给父类传参了，且不会访问到原型链上的属性
function Sub(age, name) {
  Super.call(this, name)
  this.age = age
}
```

盗用构造函数继承一般也不会单独使用，也有一些问题需要面对：

1. 访问不了父类的原型，因此方法必须在父类构造器中定义，无法实现方法的复用。

### 组合继承

巧妙的将原型链继承和盗用构造函数继承结合到一起：

1. 用盗用构造函数继承属性
2. 用原型链继承方法

```js
function Sub(name) {
  Super.call(this, name)
  this.age = age
}
Sub.prototype = new Super()
```

问题是：调用了两次父类构造函数，导致相同的属性在实例上和原型上同时出现。

### 原型式继承

借用临时构造函数来继承，这样就无自定义一个新的子类类型。
适用于在一个对象基础之上进行创建一个新对象。

```js
function Object(obj) {
  function F() {} // 创建临时构造函数
  F.prototype = obj
  return new F()
}
// 与 Obje.create(obj) 的效果相同
```

注意：obj 中属性包含的引用值，也会在所有实例间共享。

### 寄生式继承

主要是利用工厂模式的思想和寄生构造函数。

```js
function Obj(obj) {
  let newObj = Object.create(obj)
  newObj.func = function () {
    console.log('demo')
  }
  return newObj
}
```

工厂函数用原型式继承创建出新对象，然后在工厂内对新对象进行增强，最后返回这个新对象。
所以该继承也是适用于只关注对象，而不关注子类类型和构造函数的场景下。

同时寄生继承的缺点：给对象添加函数，导致函数难以复用。（这点与盗用构造函数继承一样）

### 寄生式组合继承

```js
function Demo(Super, Sub) {
  let super = Object.create(Super.prototype)        // 创建新对象
  super.constructor = Sub                    // 增强对象(修正constructor)
  Sub.prototype = super                      // 修改子类的原型
}
/* ---------- 更详细一点 ---------- */
function Demo(Super, Sub) {
  function F() {}                   // dummyFunciton 守卫函数,获取父类原型
  F.prototype = Super.prototype
  Sub.prototype  = new F()          // 子类继承父类
  Sub.prototype.constructor = Sub   // 修正构造函数
}
```

这下看的够仔细了吧，其实寄生组合继承和寄生式都是是基于原型式继承。  
寄生组合继承的优势就是中间借助一个临时的构造器，让原型链连接上了，这样避免了组合继承时调用两次父类构造器，让实例和原型链上有相同属性的问题。

### class 继承

使用 extends 关键字

```js
class A {

}
class B extends A {
  constructor() {
    super()
  }
}

// 构造函数的原型链指向父函数的原型对象
B.prototype.__proto__ === A.prototype;
// 构造函数继承自父函数
B.__proto__ === A;
```

> 注意：子类中如果没有调用`super(...args)`，是不允许使用`this`的。

## ES5 和 ES6 继承的区别

- es5 中，先创建了子类的实例，然后把从父类继承的方法添加到实例 this 上
- es6 中，子类自身是没有 this 的，通过调用`super()`获取到父类的 this，然后对 this 进行增强

> class 也可以继承普通函数

## 参考

- JavaScript 高级程序设计(第四版)

