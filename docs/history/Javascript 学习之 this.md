## 1. this、作用域、闭包、对象

### QUESTION 1

```
var name = 'window';

var person1 = {
  name: 'person1',
  show1: function () { 
  	console.log(this.name) 
  },
  show2: () => console.log(this.name),
  show3: function () {
    return function () {
      console.log(this.name)
    }
  },
  show4: function () {
    return () => console.log(this.name)
  }
};

var person2 = { name: 'person2' };

// list outputs of functions below
person1.show1()
person1.show1.call(person2)

person1.show2()
person1.show2.call(person2)

person1.show3()()
person1.show3().call(person2)
person1.show3.call(person2)()

person1.show4()()
person1.show4().call(person2)
person1.show4.call(person2)()

```

正确答案选下：

```
person1.show1() // person1
person1.show1.call(person2) // person2

person1.show2() // window
person1.show2.call(person2) // window

person1.show3()() // window
person1.show3().call(person2) // person2
person1.show3.call(person2)() // window

person1.show4()() // person1
person1.show4().call(person2) // person1
person1.show4.call(person2)() // person2
```

所以，this总是指向调用该函数的对象。其中，在全局函数中，this等于window。

### QUESTION 2

```
var name = 'window'

function Person (name) {
  this.name = name;
  this.show1 = function () {
    console.log(this.name)
  }
  this.show2 = () => console.log(this.name)
  this.show3 = function () {
    return function () {
      console.log(this.name)
    }
  }
  this.show4 = function () {
    return () => console.log(this.name)
  }
}

// list outputs of functions below

var personA = new Person('personA')
var personB = new Person('personB')

personA.show1()
personA.show1.call(personB)

personA.show2()
personA.show2.call(personB)

personA.show3()()
personA.show3().call(personB)
personA.show3.call(personB)()

personA.show4()()
personA.show4().call(personB)
personA.show4.call(personB)()
```

正确答案选下：

```
personA.show1() // personA
personA.show1.call(personB) // personB

personA.show2() // personA
personA.show2.call(personB) // personA

personA.show3()() // window
personA.show3().call(personB) // personB
personA.show3.call(personB)() // window

personA.show4()() // personA
personA.show4().call(personB) // personA
personA.show4.call(personB)() // personB
```

答案与QUESTION1的答案千差万别。那么，构造函数创建对象，具体做了什么事呢？

使用 new 操作符调用构造函数，实际上会经历一下4个步骤：

* 创建一个新对象；
* 将构造函数的作用域赋给新对象（因此this就指向了这个新对象）；
* 执行构造函数中的代码（为这个新对象添加属性）；
* 返回新对象。

personA的函数的作用域链从构造函数产生的闭包开始，而person1的函数作用域仅是global，于是导致this指向的不同。



## 2.概念及理解

### 关于闭包

...每个函数都有自己的执行环境（execution context，也叫执行上下文），每个执行环境都有一个与之关联的变量对象，环境中定义的所有变量和函数都保存在这个对象中。

...当执行流进入一个函数时，函数的环境就会被推入一个环境栈中。当代码在环境中执行时，会创建一个作用域链，来保证对执行环境中的所有变量和函数的有序访问。函数执行之后，栈将环境弹出。

...函数内部定义的函数会将包含函数的活动对象添加到它的作用域链中。

...每个函数被调用时都会自动获取两个特殊变量：this和arguments。



### 关于箭头函数

一个箭头函数表达式的语法比一个函数表达式更短，并且不绑定自己的 this，arguments，super或 new.target。...箭头函数会捕获其所在上下文的 this 值，作为自己的 this 值。

也就是说，箭头函数没有自身的this，所以this只能根据作用域链往上层查找，直到找到一个绑定了this的函数作用域。