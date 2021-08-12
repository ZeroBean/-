# Javascript难以理解的点



## 关于this的指向问题

1. 在类中的表现

   1.1 类的本质就是函数

   类 Class--->容器/作用域/模块壳子

   ```javascript
   class Test {
       
       constructor(){
           
       }
       
   	say(){
   		console.log("hello!")
   	}
       
       static do(){
           //该方法只在类中存在，实例上调用不生效
       }
   }
   const test = new Test()
   
   //立即执行函数
   const Test = (function(){
       function Test2(){ //construtor
   
       }
   
       Test.prototype.say = function(){
           console.log("hello!")
       }
       Test.do = function(){
   
       }
       //把这个函数添加到window
       window.Test = Test
   })()
   
   const test2 = new Test()
   ```

   1.2类中this溯源

   类中的构造函数和构造实例的溯源顺序

   ```javascript
   class Test(){
       test(){
           console.log(this)
       }
   }
   //this->{},此时的this指向的是实例化后的对象
   const test = new Test()
   test.test()
   ```

   ```javascript
   class Test(){
       //非静态方法->new->this
       //再构造函数中的this的属性，会绑定到他的实例对象
       constructor(){
           this.test = function(){
               console.log('none-static:' + this)
           }
       }
       //静态方法（类原型上的静态属性）在定义的时候就放到Test.prototype(...)
       //new this->{自己的对象实例}->__proto__->Test.prototype->__proto__->....->Object.prototype
       
       //定义的时候将test放到Test.prototype{}里面
       test(){
           console.log('static:' + this)
       }
   }
   //this->'none-static'
   const test = new Test()
   test.test()
   
   ```

   ```
   
   //关于对象的__proto__
   const TestA = Object.create(null)//此时已是Object顶端
   const TestB = {}//此时构造一个空对象实例，他是有__proto__的  
   ```

   关于继承

   ```
   class Father{
   	//1.2
   	//no this binding
   	//new ->this->{}->age属性
   	constructor(){
   		this.age = 44//此时会绑定到实例化对象上
   	}
   	swim(){
   		console.log("go Swimming")
   	}
   }
   
   class Son extends Father{
   	//1.3 此时必须添加super(),
   	
   	constructor(){
   		//首先调用father的constructor
   		//生成this绑定--> father 中的this--> son 的实例
   		//相当于 this-> new Father() ->{}
   		//所以super()一定要再绑定this之前
   		super()
   		this.hobby = 'traval'
   		console.log(this.age)
   	}
   	study(){
   		console.log(this)//1.1此时的Son类中并没有swim属性，但是会沿着__proto__进行溯源
   		this.swim()
   	}
   }
   
   const son = new Son()
   son.study()// Go swimming
   ```

   1.3 call apply bind的指向问题

   ```
   var obj = {
   	a:1
   }
   
   var obj2 = {
   	a:100
   }
   
   var a = 2
   function test(b,c){
   	//this 默认指向全局对象window
   	console.log(this.a,b,c)
   }
   test()//a=2
   test.call(obj)//a=1
   test.apply(obj)//a=1
   test.call(obj,3,4)//1,3,4
   test.apply(obj,[3,4])//1,3,4
   
   //bind不是立即执行而是返回一个函数，只会生效一次
   var test1 = test.bind(obj,3,4)
   test1()//1,3,4
   
   var test2 = test1.bind(obj2,3,4)
   test2()//还是1，3，4
   
   //以上相当于  只会生效一次!!!!!后续不生效
   var t = test.bind(obj,3,4).bind(obj2,3,4)
   t()
   ```

   1.4 箭头函数中的this

   箭头函数this-->外层作用域的指向(外层的函数不能是箭头函数)，直到非箭头函数的作用域的this指向

   ```
   
   //严格模式下普通函数会this指向undefined
   const test = ()=>{
   	console.log(this)
   }
   test()//指向window
   ```

   ```
   var obj = {
   	a:1
   }
   var a=2
   const test = ()=>{
   	console.log(this.a)
   }
   test()//--2--
   
   //箭头函数是忽略任何形式的this指向改变
   //即静态this指向
   test.call(obj)//--2--
   test.apply(obj)//--2--
   var test1 = test.bind(obj)
   test1()//--2--
   
   // 箭头函数一定不是一个构造器，一定不能new
   new test()
   ```

   ```
   var obj = {
   	a:1
   }
   obj.test = ()=>{
   	//箭头函数中的this不是谁绑定指向谁
   	console.log(obj)
   	console.log(this)//this指向的是window 
   }
   obj.test()
   ```

   ```
   var obj = {
   	a:1
   }
   //1.1
   obj.test = function(){
   	var t = ()=>{
   		console.log(this)//this-->obj
   	}
   	t()
   }
   //1.2
   obj.test = function(){
   	setTimeour(()=>{
   		console.log(this)//this-->obj
   	},0)
   }
   
   //1.3
   obj.test = fuction(){
   	var t1 = ()=>{
   		var t2 = ()=>{
   			console.log(this)//this-->obj
   		}
   		t2()
   	}
   	t1()
   }
   
   //1.4
   obj.test = fuction(){
   	// t1是箭头函数 this-->obj
   	// t1是普通函数 this-->window
   	var t1 = function (){
   		var t2 = ()=>{
   			console.log(this)//this-->window
   		}
   		t2()
   	}
   	t1()
   }
   obj.test()
   ```

   

总结：

this的指向的基本原则：谁调用this的宿主，this就指向谁

对象方法内部this总是指向最近的引用



嵌套的this指向

内部的t指向最近的引用，而最近指向就是window（尽管函数声明在内部）

```
var obj2 = {
	a:1,
	b:2,
	test3: function(){
		function t(){//这是一个孤立的函数，不要看层次误导了this指向
			console.log(this)
		}
	}
	t()
}
obj2.test3()//this->window
```

## Object.defineProperty

Object.defineProperty可以拦截get，set值

```
var obj3 = {}

Object.defineProperty(obj3,'a',{
	get: function(){
		console.log('I am A')
		console.log(this)//this->obj3
	}
})

obj3.a
```

## new的过程this指向

new 的过程

1.this->{}

2.{a:1}

3.{a:1,b:2}

4.return this(隐式的返回)

------

1.构造函数里默认隐式返回this，或者手动返回this 这个this指向的新对象构造都是成功的

2.如果手动返回了一个新对象，this指向的新对象的构造是失败的

3.如果你手动返回一个新对象，this->对象被忽略了

```
function Test(){
	this.a = 1;
	this.b = 2
	console.log(this)//this->实例化的对象
	return{
		c:3,
		d:4
	}
}
var test = new Test()
console.log(test)//{c:3,d:4}
```



## 事件处理函数的this绑定

1.处理函数内部的this指向被绑定的DOM元素

## Promise的深入理解

1.为什么Promise执行是同步的，而.then是异步呢？

​	promise.then()如果使用了同步就会阻塞整个顺序流，相当于同步程序了，那就没有了promise的使用必要

​	Promise存在使得异步程序同步化，并且不阻塞promise意外的任何程序

## Prototype知识点

1.Prototype->函数的一个属性：对象{}

2.proto-> 对象Object的一个属性：对象{}

3.对象的proto保存着该对象的构造函数的prototype

```js
function Test(){}

console.log(Test.prototype)
const test = new Test()
console.log(test.__proto__)

//test.__proto__ === Test.prototype
```

## 前端路由

### 前端路由模式

1.hash路由

​	带有#标志， 主要通过监听url中的hash变化来进行路由跳转

​	优势：兼容性更好

​	使用了原生的监听事件hashchange，监听内容：1.点击a标签，改变浏览器地址，2.浏览器的前进后退，3.window.location来改变浏览器地址

2.history模式

​	没有#标志

​	实现原理，遍历所有的a标签，阻止a标签默认事件，加入点击回调，读取href属性，通过pushstate改变浏览器location.pathname属性，然后手动执行 `popstate` 事件的回调函数，去匹配相应的路由

### 前端路由做了什么

​	对比传统的DOM出网页，Vue，React单页应用进行了组件拆分，页面跳转变为不同组件的渲染，通过切换浏览器地址路径来匹配相对于页面的组件，如何控制这一个过程，即是前端路由但是的缘由

## 原型与原型链

1.什么是原型？

​	使用原型可以实现其他语言中的类的设计模式，基本概念为实例化，继承，多态

​	实例化：好比我们们有一张造车的图纸，通过图纸我们可以实实在在的造出一台车实现我们想要的功能

​	继承：好比如该毕业我们没有车，但是父母有车，我们可以借用父母的车

1.1.查询原型

```
Object.getPrototypeOf()
```

1.2没有原型的对象

```
const me = Object.create(null,{
	name:"ZZN"
})
//此时me查不到__proto__属性
```

1.3原型方法与对象方法的优先级

很简单自己有车就不需要开父亲的车

1.4函数有多个长辈

构造函数对应的是prototype，而实例化对象对应的是--proto--

1.5更改一个对象的原型

```
Object.setPrototypeOf(子，父)
```

1.6构造函数

​	我们可以通过Prototype找到原型，那原型也可以通过constructor找到原型下游

1.7通过对象沿着原型链找到构造函数即可创建新的都西昂

```js
function User(name){
    this.name = name;
}
let john = new User("John")
//创建一个构造函数
function createObject(obj,...ags){
    const constructor = Object.getPrototypeof(obj).constructor
    return new constructor(..ags)
}
let peter = createObject(john,"peter")
```

## Javascript数据类型

数据类型分为两类：

1. 基础数据类型：Undefined,Null,Boolean,String,Number(ES6新增Symbool，BigInt)
2. 引用类型：Object，Array，Function

数据检测：

​	typeof

### 基础类型

#### Undefined类型

- ​	当数据已经声明了，但是没有初始值，此时变量的值未undefined

  ```js
  var message
  console.log(message===undefined)//true
  ```

- 当直接使用没有声明的变量是会报错

  ```js
  alert(age)//报错
  ```

- 注意！！！未声明的变量和声明了但是没有初始化的变量使用typeof都会返回undefined

  ```js
  var message
  alert(typeof(message))//undefined
  alert(typeof(age))//age未声明，但是也会是undefined
  ```

#### Null类型

- Null值表示的是一个空对象指针

  ```js
  console.log(typeof(Null))//返回一个Object
  ```

- null和undefined是相等的

总结：

​	在声明普通变量时，会隐式的初始化未undefined，在声明对象的时候，尽可能初始化未null，表示未来会将该变量作为对象来使用

#### Boolean类型

- Boolean只有两个字面值true和false

- Boolean的类型转换

  | 数据类型  | 转换true的值 | 转换false的值 |
  | --------- | ------------ | ------------- |
  | Boolean   | true         | flase         |
  | String    | 非空字符串   | ""//空字符串  |
  | Number    | 非0数字      | 0和NaN        |
  | Object    | 任何对象     | null          |
  | Undefined | 不适用       | undefined     |

  

#### Number类型

- 支持8进制和16进制

- 8进制第一位必须时0，超出范围，前导0会被忽略，转化为十进制；16进制必须0x开头

  ```
  //8进制
  
  var a = 070//转化为56
  var b = 08//转化为8
  
  //16进制
  
  var c = 0xA //转化为10
  ```

##### 浮点数

- JS会将本身是整数的浮点数转化为整数

  ```
  var message = 10.0//解析为10
  ```

- 可以使用科学计数法

- 经典的0.1+0.2！=0.3

  因为浮点数是使用IEE754标准，具体请参照IEE754标准进行计算

##### NaN

- NaN全称(Not a Number),表示为一个应该返回数值的操作数，但是未返回数值

- 涉及NAN的操作都会返回NaN

```js
console.log(NaN + 10)//NaN
```

​	NaN不等于任何值，甚至是他本身

```js
console.log(NaN==NaN)//false
```

##### 数值转换

- Number可用于任何类型转换

```js
//Number()
var number1 = Number("hello!")//NaN,表示不能数字化
var number2 = Number("") //0
var number3 = Number("000011")//11，忽略前导0
var number4 = Number(true)//1
```

- parseInt转换字符串为数值

```js
var num1 = parseInt("1234blue")//1234,忽略后续的字符串
var num2 = parseInt("")//NaN
var num3 = parseInt("0xA")//10，十六进制数
var num4 = parseInt(22.5)//22,小数点不是有效数字字符，忽略后续字符
var num5 = parseInt("070") //70
```

- parseFloat只解析十进制数，而且只认第一个小数点，后续的小数点被忽略，能解析程整数都会转化为整数

#### String

##### 	数值转换

- ​	toString() 数字，boolean，对象，字符串都有，但是null和undefined没有这个方法

  ```js
  //对于toString的传参
   var num = 10
   console.log(num.toString(2))//表示转化为二进制数，1010
   console.log(num.toString(8))//表示转化为八进制数，12
  ```

- String(),在位置转换值是否包含null和undefined的时候使用

  ```js
  var value1;
  var value2 = null;
  console.log(value1)//"undefined"
  console.log(value)//"null"
  ```

## 引用类型

引用类型其中包括Object，Array，function，Date，RegExp

### Array

