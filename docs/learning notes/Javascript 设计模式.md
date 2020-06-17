

## 前提

- 面向对象
  - 继承，子类继承父类
  - 封装，数据的局限和保护
  - 多态，统一接口不同实现
- 面向对象举例

```
class jQuery {
  constructor(selecor) {
    let slice = Array.prototype.slice;
    let dom = slice.call(document.querySelectorAll(selecor))
    let len = dom ? dom.length : 0;
    for(let i = 0; i <= len - 1; i ++) {
      this[i] = dom[i];
    }
    this.length = len;
    this.selecor = selecor || '';
  }

  append(node) {}

  addClass(name) {}
  
  html(data) {}

  // ...
}

window.$ = function(selecor) {
  // 工厂模式
  return new jQuery(selecor);
}
```

- UML 类图
  - UML：统一建模语言(Unified Modeling Language)
  - 类图：属性，方法
  - 关系
    - 泛化，表示继承
    - 关联，表示引用



## 设计原则

### 何为设计

- <<UNIX / LINUX 设计哲学>>
- 准则
  1. 小即是美
  2. 让每个程序只做好一件事
  3. 快速建立原型
  4. 舍弃高效率而取可移植性
  5. 采用纯文本来存储数据
  6. 充分利用软件的杠杆效应(软件复用)
  7. 使用 shell 脚本来提高杠杆效应和可移植性
  8. 避免强制性的用户界面
  9. 让每个程序都称为过滤器

### 设计原则

- SOLID 五大设计原则
  - S - 单一指责原则
  - O - 开放封闭原则
  - L - 里氏置换原则
  - I - 接口独立原则
  - D - 依赖倒置原则
- 单一指责原则
  - 一个程序只做好一件事情
  - 如果功能过于复杂就拆分，每个部分保持独立
- 开放封闭原则
  - 对扩展开发，对修改封闭
  - 增加需求时，扩展新代码，而非修改已有代码
  - 这是软件设计的终极目标
- 里氏置换原则
  - 子类能覆盖父类
  - 父类能出现的地方子类就能出现
- 接口独立原则
  - 保持接口的单一独立，避免出现“胖”接口
  - typescript 有接口
- 依赖倒置原则
  - 面向接口（抽象）编程，依赖于抽象而非具体
  - 使用方只关注接口，而不关注具体类的实现

## 设计模式 

### 工厂模式

- 介绍
  - 将 new 操作单独封装
  - 遇得 new 时，就要考虑是否使用工厂模式
- UML类图

- 代码

```
class Product  {
  constructor(name) {
    this.name = name;
  }

  init() {}

  fn1() {}

  fn2() {}
}

class Creator  {
  create(name) {
    return new Product(name);
  }
}

export default Creator;
```

- 场景

  - jQuery - $('p')
  - React.createElement

  ```
  class Vnode(tag, attrs, children) { /*... */ }
  React.createElement = function(tag, attrs, children) {
  	//...
  	return new Vnode(tag, attrs, children);
  }
  ```

  - Vue 异步组件

- 设计原则验证

  - 构造函数和创建者分离

  - 符合开放封闭原则

    

### 单例模式

- 介绍
  - 系统中被唯一使用
  - 一个类只能有一个实例
  - 需要用到 java/typescript 的特性 (private)，或通过 IIFE 和闭包实现
- 代码

```
class SingleObject {
  login() {
    console.log('login');
  }
};

SingleObject.getInstance = (function() {
  let instance = null;
  return function() {
    if (!instance) {
      instance = new SingleObject();
    }
  
    return instance;
  };
})();
```

- 场景
  - jQuery 只有一个全局函数 $
  - 登陆窗
  - vuex 中的 store
- 设计原则验证
  - 符合单一指责原则，实力化唯一对象



### 适配器模式

- 介绍
  - 旧接口与使用者格式不兼容
  - 中间加一个适配器转换接口

- UML 类图
- 代码

```
// 旧接口
class Adaptee {
  specificRequst() {
    return '日本插头';
  }
}

// 适配器
export class Target {
  constructor() {
    this.adaptee = new Adaptee();
  }

  request() {
    const info = this.adaptee.specificRequst();

    return `${info} - 转换器 - 中国插头`;
  }
}

// 使用者
export class Client {
  constructor(target) {
    this.target = target;
  }

  do() {
    const info = this.target.request();
    console.log('适配器模式：', info);
  }
}
```

- 场景
  - Vue.computed()
- 设计原则验证
  - 将旧接口和使用者进行分离
  - 符合开发封闭原则



### 装饰器模式

- 介绍
  - 为对象添加新功能
  - 不改变其原有的结构和功能
- UML 类图
- 代码

```
class Circle {
  draw() {
    console.log('draw a circle');
  }
}

class Decorator {
  constructor(circle) {
    this.circle = circle;
  }

  draw() {
    this.circle.draw();
    this.setRedBoarder();
  }
  
  setRedBoarder() {
    console.log('set red border');
  }
}
```



- 场景

  - ES7 装饰器

  ```
  // 装饰类
  @testDec
  class Demo { /*...*/ }
  
  function testDec(target) {
  	target.isDec = true;
  }
  
  console.log(Demo.isDec);
  
  // 装饰属性
  // ...
  ```
  
  ```
// 装饰方法
  // ...
  function readonly(target, name, decriptor) {
  	decriptor.writable = false;	
  	return decriptor;
  }
  
  // ...
  function log(target, name, decriptor) {
  	var oldVal = decriptor.value;
  	decriptor.value = function() {
  		console.log(`Calling ${name}`);
  		return  oldVal.apply(this, arguments);
  	}
  
  	return decriptor;
  }
  ```
  
  - core-decorator

- 设计原则验证



### 代理模式

- 介绍
  - 使用者无权访问目标对象
  - 中间加代理，通过代理做授权和控制
- UML 类图
- 代码

```
class RealImage {
  constructor(filename) {
    this.filename = filename;
    this.loadFromDisk()
  }

  loadFromDisk() {
    console.log('loading...' + this.filename);
  }

  display() {
    console.log('display: ' + this.filename);
  }
}

class ProxyImage {
  constructor(filename) {
    this.realImage = new RealImage(filename);
  }

  display() {
    this.realImage.display();
  }
}

```

- 场景

  - 网页事件代理
  - JQuery $.proxy
  - ES6 Proxy

  ```
  // 明星
  const star = {
    name: '蔡徐坤',
    age: 23,
    phone: 1861234567
  };
  
  // 经纪人
  const agent =  new Proxy(star , {
    get: function(target, key) {
      if(key === 'phone') {
        return 'agent phone: 1351234567'
      }
      if (key === 'price') {
        return 8000;
      }
      return target[key];
    },
  
    set: function(target, key, val) {
      if (key === 'customPrice') {
        if (val < 6000) {
          throw new Error('价格太低');
        } else {
          target[key] = val;
          return true;
        }
      }
    }
  });
  ```

- 设计原则验证

  - 代理类与目标类分离，隔离开目标类和使用者
  - 符合开发封闭原则



### 外观模式

- 介绍
  - 为子系统的一组统一接口提供一个高层接口
  - 使用者使用这个高层接口
- UML 类图
- 代码
- 场景
- 设计原则验证
  - 不符合单一职责原则和开放封闭原则，故谨慎使用，不可滥用



### 观察者模式

- 介绍
  - 发布 & 订阅
  - 一对多
- UML 类图
- 代码

```

```

- 场景
- 设计原则验证

### 迭代器模式

- 介绍
- UML 类图
- 代码
- 场景
- 设计原则验证





### 状态模式

- 介绍
- UML 类图
- 代码
- 场景
- 设计原则验证





### 职责链模式

- 介绍
- UML 类图
- 代码
- 场景
- 设计原则验证