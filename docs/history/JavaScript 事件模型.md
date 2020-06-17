## 1.冒泡型和捕获型

```
addEventListener(event, fn, useCapture)
//useCapture： 布尔值，默认false
```

- **冒泡型事件**(useCapture 为 false)： 事件按照从最特定的事件目标到最不特定的事件目标(document对象)的顺序触发，即从该节点到各个父亲节点依次向上传播。
  - IE 5.5: div -> body -> document
  - IE 6.0: div -> body -> html -> document
  - ​Mozilla 1.0: div -> body -> html -> document -> window


- **捕获型事件**(useCapture 为 true)：事件从最不精确的对象(document 对象)开始触发，然后到最精确(被指定的节点)。


- 传统绑定事件，采用的是**冒泡**方式。

  elem.onclick = doSomething

- **阻止传播**，`e.stopPropagation()`，阻止事件传播到下一个节点。

  ​

## 2. 举个栗子

```
<div id='one'>
  <div id='two'>
    <div id='three'>
      <div id='four'></div>
    </div>
  </div>
</div>

<script>
	var one = document.getElementById('one');
    var two = document.getElementById('two');
    var three = document.getElementById('three');
    var four = document.getElementById('four');
    
    //false为冒泡获取[目标元素先触发],
    //true为捕获获取[父级元素先触发]
    var useCapture = true; 
    
    one.addEventListener('click', function() {
        console.log('one');
    }, useCapture);
    
    two.addEventListener('click', function(e) {
        // e.stopPropagation();
        console.log('two');
    }, useCapture);

    three.addEventListener('click', function() {
        console.log('three');
    }, useCapture);

    four.addEventListener('click', function(e) {
        console.log('four');
    }, useCapture);       
    
</script>
```

设置不同的useCapture值，输出结果如下：

    1. false //冒泡

    点击four div
    输出结果：four three two one       
    
    2. true  //捕获
    
    点击four div
    输出结果： one two three four
