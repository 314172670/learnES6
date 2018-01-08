## ES6——let 和const

### let和var ###

- let命令所在的代码块内(指的是花括号{}包含的)有效 

- var命令所有数组a的成员里面的i，指向的都是同一个i，导致运行时输出的是最后一轮的i的值，也就是 10

		var a = [];
		for (var i = 0; i < 10; i++) {
		  a[i] = function () {
		    console.log(i);
		  };
		}
		a[6](); // 10

- let则每一次循环的i其实都是一个新的变量，所以最后输出的是6，JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量i时，就在上一轮循环的基础上进行计算

		var a = [];
		for (let i = 0; i < 10; i++) {
		  a[i] = function () {
		    console.log(i);
		  };
		}
		a[6](); // 6

- for循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域

		for (let i = 0; i < 3; i++) {
		  let i = 'abc';
		  console.log(i);
		}
		// abc
		// abc
		// abc
	>上面代码正确运行，输出了 3 次abc。这表明函数内部的变量i与循环变量i不在同一个作用域，有各自单独的作用域。

- let命令改变了语法行为，它所声明的变量一定要在声明后使用，否则报错

- 只要块级作用域内存在let命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。

- 如果一个变量根本没有被声明，使用typeof反而不会报错

- let不允许在相同作用域内，重复声明同一个变量

- 用来计数的循环变量泄露为全局变量，变量i只用来控制循环，但是循环结束后，它并没有消失，泄露成了全局变量。(使用块级作用域的原因之一)

		var s = 'hello';
		
		for (var i = 0; i < s.length; i++) {
		  console.log(s[i]);
		}
		
		console.log(i); // 5

- let中外层作用域无法读取内层作用域的变量，内层作用域可以定义外层作用域的同名变量

- 块级作用域之中，函数声明语句的行为类似于let，在块级作用域之外不可引用。

- 在浏览器的 ES6 环境中，块级作用域内声明的函数，行为类似于var声明的变量。

- 返回内部最后执行的表达式的值,在块级作用域之前加上do，使它变为do表达式，然后就会返回内部最后执行的表达式的值。

		let x = do {
		  let t = f();
		  t * t + 1;
		};
上面代码中，变量x会得到整个块级作用域的返回值（t * t + 1）。

		
- 考虑到环境导致的行为差异太大，应该避免在块级作用域内声明函数。如果确实需要，也应该写成函数表达式，而不是函数声明语句。

		// 函数声明语句
		{
		  let a = 'secret';
		  function f() {
		    return a;
		  }
		}
		
		// 函数表达式
		{
		  let a = 'secret';
		  let f = function () {
		    return a;
		  };
		}

>另外，还有一个需要注意的地方。ES6 的块级作用域允许声明函数的规则，只在使用大括号的情况下成立，如果没有使用大括号，就会报错。


### const ###

- const声明一个只读的常量。一旦声明，常量的值就不能改变。

- 一旦声明变量，就必须立即初始化

- 只在声明所在的块级作用域内有效

- 只能在声明的位置后面使用。也与let一样不可重复声明

- const实际上保证的,变量指向的那个内存地址不得改动

- const只能保证这个指针是固定的，至于它指向的数据结构是不是可变的，就完全不能控制了
常量foo储存的是一个地址，这个地址指向一个对象
- 可以为其添加新属性

- 对象冻结，应该使用Object.freeze方法，严格模式时还会报错添加新属性不起作用
除了将对象本身冻结，对象的属性也应该冻结。下面是一个将对象彻底冻结的函数:

		var constantize = (obj) => {
		  Object.freeze(obj);
		  Object.keys(obj).forEach( (key, i) => {
		    if ( typeof obj[key] === 'object' ) {
		      constantize( obj[key] );
		    }
		  });
		};

###顶层对象的属性

顶层对象，在浏览器环境指的是window对象，在 Node 指的是global对象。ES5 之中，顶层对象的属性与全局变量是等价的。

- var命令和function命令声明的全局变量，依旧是顶层对象的属性；另一方面规定，let命令、const命令、class命令声明的全局变量，不属于顶层对象的属性。


- 可以在所有情况下，都取到顶层对象。下面是两种勉强可以使用的方法。

		// 方法一
		(typeof window !== 'undefined'
		   ? window
		   : (typeof process === 'object' &&
		      typeof require === 'function' &&
		      typeof global === 'object')
		     ? global
		     : this);
		
		// 方法二
		var getGlobal = function () {
		  if (typeof self !== 'undefined') { return self; }
		  if (typeof window !== 'undefined') { return window; }
		  if (typeof global !== 'undefined') { return global; }
		  throw new Error('unable to locate global object');
		};

- 垫片库system.global模拟了这个提案，可以在所有环境拿到global。

		// CommonJS 的写法
		require('system.global/shim')();
		
		// ES6 模块的写法
		import shim from 'system.global/shim'; shim();

	>上面代码可以保证各种环境里面，global对象都是存在的。
	
		// CommonJS 的写法
		var global = require('system.global')();
		
		// ES6 模块的写法
		import getGlobal from 'system.global';
		const global = getGlobal();

	>上面代码将顶层对象放入变量global。

