---
layout: post
title: JS中的this以及Call与Apply
categories: Js
---

## 一. this 关键字

this 是JS中一个十分常见的关键字，那么 this 指向的是谁呢？

this 的指向不是在函数定义的时候确定的，只有函数执行的时候才能确定 this 到底指向谁，this 指向的是那个调用它的对象。

下面分情况分析一下 this 在不同情况下的指向。

1. 在 普通函数 中

```javascript
function test(){
  console.log(this);
}
test();// Window
```

上面的代码 this 没有调用的对象，默认为Window，但在严格模式下的默认值为undefined

```javascript
function test(){
  "use strict";
  console.log(this);
}
test();// undefined
```

当我们设置了方法的调用对象后，this 将指向调用对象。

```javascript
function test(){
  "use strict";
  console.log(this);
}
window.test();// Window
```

同理也能理解对象方法的this。

```javascript
var name="window";
var person = {
  name:"hhLiu",
  sayName:function(){
    console.log(this.name);
  }
};
person.sayName();// hhLiu
window.person.sayName();// hhLiu

var wFun = person.sayName;
window.wFun();// window
```

上面例子中前两个输出均为hhliu，我们可以总结出，this 指向的是上一级调用它的对象。而最后一个例子印证了this的指向只有在函数执行时才能确定，尽管wFun和person.sayName是同一个函数，但是由于wFun的调用对象是window，因此会导致this的指向不同。

2. 在 构造函数 中

```javascript
function Person(name){
  this.name = name;
  this.sayName = function(){
    console.log(this.name);
  }
}
var person = new Person("hhliu");
person.sayName();// hhliu
```

当我们把一个函数当做构造函数来使用，即使用 new 操作符的时候，函数中的 this 将指向新建的对象。

3. 在 回调函数 中

```javascript
var test = {
  id:"test",
  sayId:function(){
    console.log(this.id);
  }
}

$("#abc").click(test.sayId); // abc
```

回调函数实际上是将test.sayId方法在$("#abc")对象上执行，因此调用者也就不再是 test 对象而是实际调用的对象。

4. 在 闭包 中
每个函数被调用时，其活动对象都会自动取得两个特殊变量：this和arguments。内部函数在搜索这两个变量时，只会搜索到其活动对象为止，因此永远不可能直接访问外部函数中的这两个变量。  《Javascript高级程序设计》

也就是说 闭包不能访问到其外部函数的this。

```javascript
var out = {
  name:"out",
  sayName:function(){
    var closure = function(){
      console.log(this.name);
    }
    return closure;
  }
}
out.sayName()();// window
```

如果我们想要获取闭包的外部函数中的this的话，可以用一个变量存储一下，闭包内去访问变量即可；

```javascript
var out = {
  name:"out",
  sayName:function(){
    var that = this;
    var closure = function(){
      console.log(that.name);
    }
    return closure;
  }
}
out.sayName()();// out
```

## 二. Call与Apply

Call 与 Apply 实现的功能相同，都是为了动态改变this而出现的，如果一个对象没有某个方法，我们可以使用call或apply来“借用”其它对象的方法。

```javascript
var cat = {
	name : "kitty"
};
var person = {
	name : "hhliu",
	sayName : function() {
		console.log(this.name)
	},
	plusNum : function(a, b) {
		console.log(a + b)
	}
};
person.sayName.call(cat); // Kitty
person.plusNum.call(cat, 1, 5); // 6
person.plusNum.apply(cat, [ 1, 5 ]);// 6
```

这里cat不会说自己的名字也不会作加法，但我们使用call和apply可以使cat借用人的方法。

call和apply的区别在于call接受的是连续参数，而apply接受的是数组参数。

另外利用call和apply可以实现继承

```javascript
function Parent(username) {
	this.username = username;
	this.hello = function() {
		alert(this.username);
	};
}

function Child(username, password) {
	Parent.call(this, username);
	// 这里的this传到Parent中的this、这里的username传到Parent的参数

	this.password = password;
	this.world = function() {
		alert(this.password);
	};
}

var parent = new Parent("zhangsan");
var child = new Child("lisi", "123456");
parent.hello();
child.hello();
child.world();
```