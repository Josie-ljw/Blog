# 关于this指向的详细总结

> 在 ES5 中，其实 this 的指向，始终坚持一个原理：this 永远指向最后调用它的那个对象

### this的四种绑定规则

##### 默认绑定

> 规则：在非严格模式下，默认绑定的this指向全局对象，严格模式下this指向undefined

``` js
function foo() {
  console.log(this.a); // this指向全局对象
}
var a = 2;
foo(); // 2
function foo2() {
  "use strict"; // 严格模式this绑定到undefined
  console.log(this.a); 
}
foo2(); // TypeError:a undefined
```

``` js
function foo() {
  console.log(this.a); // foo函数不是严格模式 默认绑定全局对象
}
var a = 2;
function foo2(){
  "use strict";
  foo(); // 严格模式下调用其他函数，不影响默认绑定
}
foo2(); // 2
```
所以：对于默认绑定来说，决定this绑定对象的是<font color="red">函数体是否处于严格模式</font>，严格指向undefined，非严格指向全局对象。

##### 隐式绑定

> 规则：函数在调用位置，是否有上下文对象，如果有，那么this就会隐式绑定到这个对象上。

``` js 
function foo() {
 console.log(this.a);
}
var a = "Oops, global";
let obj2 = {
 a: 2,
 foo: foo
};
let obj1 = {
 a: 22,
 obj2: obj2
};
obj2.foo(); // 2 this指向调用函数的对象
obj1.obj2.foo(); // 2 this指向最后一层调用函数的对象
    
// 隐式绑定丢失
let bar = obj2.foo; // bar只是一个函数别名 是obj2.foo的一个引用
bar(); // "Oops, global" - 指向全局
```

**隐式绑定丢失：**
隐式绑定丢失的问题：实际上就是函数调用时，并没有上下文对象，只是对函数的引用，所以会导致隐式绑定丢失。
同样的问题，还发生在传入回调函数中，这种情况更加常见，并且隐蔽，类似：

``` js
test(obj2.foo); // 传入函数的引用，调用时也是没有上下文对象。
```

##### 显式绑定:

就像我们上面看到的，如果单纯使用隐式绑定肯定没有办法得到期望的绑定，幸好我们还可以在某个对象上强制调用函数，从而将this绑定在这个函数上。
> 规则：我们可以通过**apply、call、bind**将函数中的this绑定到指定对象上。

``` js
function foo() { 
    console.log(this.a); 
} 
let obj = { a: 2 }; 
foo.call(obj); // 2
```

##### new绑定

书中提到：在js中，实际上并不存在所谓的'构造函数'，只有对于函数的'构造调用'。

**new的时候会做哪些事情：**

* 创建一个全新的对象。这个新对象会被执[[Prototype]]连接。
* 这个新对象会绑定到函数调用的this。
* 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。

> 规则：使用构造调用的时候，this会自动绑定在new期间创建的对象上。

``` js
function foo(a) {
  this.a = a; // this绑定到bar上
}
let bar = new foo(2);
console.log(bar.a); // 2
```

#### this四种绑定规则的优先级

如果在某个调用位置应用了多条规则，如何确定哪条规则生效？

``` js
obj.foo.call(obj2); // this指向obj2 显式绑定比隐式绑定优先级高。
new obj.foo(); // this指向new新创建的对象 new绑定比隐式绑定优先级高。
```

显式绑定和隐式绑定无法直接比较(会报错),默认绑定是不应用其他规则之后的兜底绑定所以优先级最低，最后的结果是：
**显式绑定 > 隐式绑定 > 默认绑定
new绑定 > 隐式绑定 > 默认绑定**

### 怎么改变 this 的指向

改变 this 的指向我总结有以下几种方法：

* 使用 ES6 的箭头函数
* 在函数内部使用 _this = this
* 使用 apply、call、bind
* new 实例化一个对象

**举个例子**

``` js
var name = "windowsName";
    var a = {
        name : "Cherry",
        func1: function () {
            console.log(this.name)     
        },
        func2: function () {
            setTimeout(  function () {
                this.func1()
            },100);
        }
    };
    a.func2() // this.func1 is not a function
```
在不使用箭头函数的情况下，是会报错的，因为最后调用 setTimeout 的对象是 window，但是在 window 中并没有 func1 函数。

#### 箭头函数

> 众所周知，ES6 的箭头函数是可以避免 ES5 中使用 this 的坑的。箭头函数的 this 始终指向函数定义时的 this，而非执行时。，箭头函数需要记着这句话：“箭头函数中没有 this 绑定，必须通过查找作用域链来决定其值，如果箭头函数被非箭头函数包含，则 this 绑定的是最近一层**非箭头函数**的 this，否则，this 为 undefined”。

``` js
var name = "windowsName";
var a = {
   name : "Cherry",
   func1: function () {
       console.log(this.name)     
   },
   func2: function () {
       setTimeout( () => {
           this.func1()
       },100);
   }
};
a.func2()     // Cherry
```

#### 在函数内部使用 _this = this

> 我们是先将调用这个函数的对象保存在变量 _this 中，然后在函数中都使用这个 _this，这样 _this 就不会改变了。

``` js
var name = "windowsName";
var a = {
   name : "Cherry",
   func1: function () {
       console.log(this.name)     
   },
   func2: function () {
       var _this = this;
       setTimeout( function() {
           _this.func1()
       },100);
   }
};
a.func2()       // Cherry
```
这个例子中，在 func2 中，首先设置 var _this = this;，这里的 this 是调用 func2 的对象 a，为了防止在 func2 中的 setTimeout 被 window 调用而导致的在 setTimeout 中的 this 为 window。我们将 this(指向变量 a) 赋值给一个变量 _this，这样，在 func2 中我们使用 _this 就是指向对象 a 了。

#### 使用 apply、call、bind 
* apply

```js
var a = {
   name : "Cherry",

   func1: function () {
       console.log(this.name)
   },

   func2: function () {
       setTimeout(  function () {
           this.func1()
       }.apply(a),100);
   }

};

a.func2()            // Cherry
```

* call

```js
 var a = {
   name : "Cherry",

   func1: function () {
       console.log(this.name)
   },

   func2: function () {
       setTimeout(  function () {
           this.func1()
       }.call(a),100);
   }

};

a.func2()            // Cherry
```

* bind

```js
var a = {
   name : "Cherry",

   func1: function () {
       console.log(this.name)
   },

   func2: function () {
       setTimeout(  function () {
           this.func1()
       }.bind(a)(),100);
   }

};

a.func2()            // Cherry
```

### apply、call、bind 区别

> apply() 方法调用一个函数, 其具有一个指定的this值，以及作为一个数组（或类似数组的对象）提供的参数

语法：

```js
fun.apply(thisArg, [argsArray])
```

* thisArg：在 fun 函数运行时指定的 this 值。需要注意的是，指定的 this 值并不一定是该函数执行时真正的 this 值，如果这个函数处于非严格模式下，则指定为 null 或 undefined 时会自动指向全局对象（浏览器中就是window对象），同时值为原始值（数字，字符串，布尔值）的 this 会指向该原始值的自动包装对象。

* argsArray：一个数组或者类数组对象，其中的数组元素将作为单独的参数传给 fun 函数。如果该参数的值为null 或 undefined，则表示不需要传入任何参数。从ECMAScript 5 开始可以使用类数组对象。浏览器兼容性请参阅本文底部内容。

#### apply 和 call 的区别

```js
//call
fun.call(thisArg[, arg1[, arg2[, ...]]])
//apply
fun.apply(thisArg, [argsArray])
```

**所以 apply 和 call 的区别是 call 方法接受的是若干个参数列表，而 apply 接收的是一个包含多个参数的数组。**

```js
// apply
var a ={
   name : "Cherry",
   fn : function (a,b) {
       console.log( a + b)
   }
}

var b = a.fn;
b.apply(a,[1,2])     // 3

// call
var a ={
   name : "Cherry",
   fn : function (a,b) {
       console.log( a + b)
   }
}

var b = a.fn;
b.call(a,1,2)       // 3
```

### bind 和 apply、call 区别

> bind()方法创建一个新的函数, 当被调用时，将其this关键字设置为提供的值，在调用新函数时，在任何提供之前提供一个给定的参数序列。

```js
var a ={
   name : "Cherry",
   fn : function (a,b) {
       console.log( a + b)
   }
}

var b = a.fn;
b.bind(a,1,2)()           // 3 需要手动调用
```

### js中的函数调用

1. 作为一个函数调用
2. 函数作为方法调用
3. 使用构造函数调用
4. 作为函数方法调用函数（call，apply）

#### 作为一个函数调用

```js
    var name = "windowsName";
    function a() {
        var name = "Cherry";
        console.log(this.name);  // windowsName
        console.log("inner:" + this);  // inner: Window
    }
    a();
    cosnole.log("outer:" + this);  // outer: Window
```
在非严格模式默认是属于全局对象 window 的，在严格模式，就是 undefined。
**但这是一个全局的函数，很容易产生命名冲突，所以不建议这样使用。**

#### 函数作为方法调用

```js
    var name = "windowName";
    var a = {
        name: "Josie",
        fn: function() {
        console.log(this.name); // Josie
        }
    }
    a.fn();
```
this 永远指向最后调用它的那个对象

#### 使用构造函数调用函数

> 如果函数调用前使用了 new 关键字, 则是调用了构造函数。
这看起来就像创建了新的函数，但实际上 JavaScript 函数是重新创建的对象：

```js
    //构造函数
    function myFuntion(arg1,arg2) {
        this.firstName = arg1;
        this.lastName = arg2;
    }
    
    // 创建了一个新的函数
    var a = new myFuntion("super", "star");
    a.lastName; // star
```

**new 的过程**

```js
var a = new myFunction("Li","Cherry");

new myFunction{
    var obj = {};
    obj.__proto__ = myFunction.prototype;
    var result = myFunction.call(obj,"Li","Cherry");
    return typeof result === 'obj'? result : obj;
}
```

1. 创建一个空对象 obj;
2. 将新创建的空对象的隐式原型指向其构造函数的显示原型。
3. 使用 call 改变 this 的指向
4. 如果无返回值或者返回一个非对象值，则将 obj 返回作为新对象；如果返回值是一个新对象的话那么直接直接返回该对象。

**所以我们可以看到，在 new 的过程中，我们是使用 call 改变了 this 的指向。**

#### 作为函数方法调用函数

> 在 JavaScript 中, 函数是对象。
JavaScript 函数有它的属性和方法。call() 和 apply() 是预定义的函数方法。 两个方法可用于调用函数，两个方法的第一个参数必须是<font color="purple">对象本身</font>
在 JavaScript 严格模式(strict mode)下, 在调用函数时第一个参数会成为 this 的值， 即使该参数不是一个对象。在 JavaScript 非严格模式(non-strict mode)下, 如果第一个参数的值是 null 或 undefined, 它将使用全局对象替代。

```js
var name = "windowsName";

    var a = {
        name : "Cherry",

        func1: function () {
            console.log(this.name)     
        },

        func2: function () {
            setTimeout(  function () {
                this.func1()
            },100 );
        }

    };

    a.func2()     // this.func1 is not a function

```

还是那句话this 永远指向最后调用它的那个对象，那么我们就来找最后调用匿名函数的对象，这就很尴尬了，因为匿名函数名字啊，笑哭，所以我们是没有办法被其他对象调用匿名函数的。所以说 **匿名函数的 this 永远指向 window**。
如果这个时候你要问，那匿名函数都是怎么定义的，首先，我们通常写的匿名函数都是自执行的，就是在匿名函数后面加 () 让其自执行。其次就是虽然匿名函数不能被其他对象调用，但是可以被其他函数调用啊，比如例 7 中的 setTimeout。


参考文章：[this、apply、call、bind](https://juejin.im/post/59bfe84351882531b730bac2)



