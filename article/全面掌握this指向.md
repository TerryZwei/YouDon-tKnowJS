# 全面掌握javascript中的this

## 1. 为什么要用this？

`this`是`js`中最为复杂的机制之一。大部分开发者写了很久的`js`代码都可能没有主动使用过`this`，或者遇到就直接"忽略"，但其实`this`本质上提供了一种更加优雅的方式来传递对象引用，可以使代码更加简洁和易于复用，所以全面了解this是有必要的。



## 2. this的误解

在学习`this`之前，先来说说它的两个误区。第一个就是`this`并不是指代本身，这里是因为用了词法作用域才会出现这样的理解；第二个就是`this`的绑定和声明没有任何关系，它取决于调用时候的条件。



## 3. this的理解

`this`的绑定是在调用的时候确定下来的，和声明没有任何关系。



### 3.1. this绑定规则（4个）

`this`绑定规则有五个，分别是：默认绑定，隐式绑定，显式绑定，new绑定。



#### 3.1.1. 默认绑定

 独立函数调用，无法应用其他规则的时候使用的默认规则。这个规则根据会根据`严格模式`和`非严格模式`绑定不一样的值，严格模式`use strict`绑定的是`undefined`，而非严格模式绑定的是全局对象（浏览器就是`window`）。

```js
// 非严格模式
function foo() {
	console.log(this);
}
foo(); // window


// 严格模式
function bar() {
	'use strict'
	console.log(this);
}
bar(); // undefined
```



#### 3.1.2. 隐式绑定

第二个`this`绑定的规则就是`隐式绑定`，其实就是调用的位置是否有上下文对象。来看下面这个例子：

```js
function foo() {
	console.log(this.a);
}

var obj = {
	a: 1,
	foo: foo
};

obj.foo(); // 1
```

这里调用`foo`函数的方式是通过`obj.foo()`，使这个函数被调用的时候加上了`obj`这个上下文对象，`隐式绑定`这个规则会把函数中的this绑定到obj这个对象上，因此`this.a`和`obj.a`是一样的。



**多个层级引用**

大多数情况下，调用函数的层级都不是简单的一层，如果出现多层的话，函数中的`this`会绑定在哪一个对象上？例如：`obj1.obj2.obj3.foo()`，可以看下面的例子：

```js
function foo() {
	console.log(this.a);
}

var obj2 = {
	a: 2,
	foo: foo
};

var obj1 = {
	a: 1,
	obj2: obj2
};

// 多个对象引用函数的this会绑定在上一层或者最后一层的属性上
obj1.obj2.foo(); // 2
```

这个例子中的`foo`中的`this`绑定在`obj2`对象上，也就是说函数中的`this`绑定在上一层或者最后一层对象属性上。



**隐式丢失**

`隐式绑定`还有一个比较常见的问题就是`this`绑定时会丢失绑定对象，这样`this`会应用`默认绑定`规则，绑定到全局对象或者`undefined`上，这取决于是否严格模式（`use strict`）。

```js
function foo () {
	console.log(this.a);
}

var obj = {
	a: 1,
	foo: foo
};

var f = obj.foo; // 这里f是obj.foo的引用,引用的是foo函数本身
f(); // undefined; 
```

上面例子中的`f`是`obj.foo`的引用，也就是`foo`函数本身，当执行`f`函数时，相当于直接执行`f()`，`foo`函数中的`this`应用默认绑定规则，把当前函数内的`this`绑定到全局对象上（`window`）。

这里还有一个关于`setTimeout`经典的例子：

```js
function foo() {
  console.log(this.a);
}

var obj = {
  a: 'obj的a属性',
  foo: foo
}

var a = '全局变量a';

setTimeout(obj.foo,0); // '全局变量a'
```

这里传入`setTimeout`的其实是`obj.foo`引用的函数本身，也就是`f`函数，那么这个时候也是应用`默认绑定`规则，函数里的`this`会绑定到全局对象上。其实回调函数丢失`this`绑定是非常常见的，所以在处理回调函数的`this`绑定时候要注意。



#### 3.1.3. 显式绑定

显式绑定一般使用的就是`apply`、`call`、`bind`这几个方法，其中`apply`和`call`都是调用函数并且把它的`this`绑定在方法的第一个参数上，这两个方法的区别在于其他参数上。而`bind`也会绑定`this`到第一个参数对象上，但并不执行该函数。

```js
function foo() {
  console.log(this.a);
}

var obj = {
  a: 'obj的a属性'
};

foo.apply(obj); // 'obj的a属性'
```

这里使用`foo.apply(obj)`显式地绑定`foo`函数里面的`this`到`obj`对象上，`foo`函数里的`this.a`也就是`obj.a`。



**apply和call**

`apply`和`call`通过第一个参数影响函数里面的`this`绑定，但有时候第一个参数并不是普通的对象。

```js
function foo() {
  console.log(this.toFixed(2));
}

foo.apply(3); // 3.00
```

`apply`方法里面的`3`明显不是对象，那在`apply`执行后，函数里面的`this`会被转换成`new Number(3)`，所以最终执行的就是`new Number(3).toFixed(2)`输出结果是`3.00`。`apply`和`call`如果第一个参数传入原始值（数字，字符串，布尔值）来绑定`this`对象，那么`this`绑定的当前值是（`new Number()`、`new Boolean()`、`new String()`） 。

还有一种特殊情况就是，`apply`和`call`第一个参数传入`null`和`undefined`，那么函数中的`this`绑定被忽略，使用`默认绑定`规则，`this`绑定在全局对象或者`undefined`（这个取决于是否严格模式）。



**bind**

```js
function foo(param) {
  console.log(this.a, param);
}

var obj = {
  a: 'obj的a属性'
};

var f = foo.bind(obj);
f('f的参数'); // obj的a属性 f的参数
```

bind()会返回一个新的函数，并且会为函数的this绑定到第一个参数上。



#### 3.1.4. new绑定

在`js`中，构造函数其实是使用`new`操作符时调用的函数，构造函数并不属于某个类，实际上也不会实例化一个类，构造函数就是一个使用`new`操作符调用的普通函数。

```js
function Foo(a) {
  this.a = a;
}

var f = new Foo(1);
console.log(f.a); // 1
```

这里的`new`操作符调用`Foo()`函数时，内部会创建一个新的对象并把它绑定到`Foo()`调用的`this`上，如果这个函数没有其他返回值对象，那么函数就会自动返回这个新对象。



### 3.2. 箭头函数的this

