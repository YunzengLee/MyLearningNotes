## 1 什么是 JS

一门世界上最流行的脚本语言

Java与JavaScript没有关系

## 2 快速入门

### 1 引入javascript

1. 内部标签

   ```html
   <script>
   // js代码
   </script>
   ```

   

2. 外部引入

   abc.js

   ```javascript
   alert('hello world');
   ```

   text.html

   ```html
   <script src="abc.js"></script>
   ```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

<!--    script标签内写js代码,该标签也可以放在body里-->
<!--    方式1-->
    <script src="js/qj.js"></script>
<!--    方式2-->
    <script>
        alert('hello world!');
    </script>

<!--    不用显式定义type也默认是js-->
    <script type="text/javascript"></script>
</head>
<body>

</body>
</html>
```



### 2 语法入门

```html
<script>
        alert('hello world!');
        // 1 定义变量
        var num =1;
        var name = "name";
        // 2 条件控制
        if(num>1){
            alert("true");
        }else if(num<1 && num != 0){
            
        }
        //js严格区分大小写
        //console.log();在浏览器的控制台打印变量
        
    </script>
```

### 3 数据类型

数值、文本、图形、音频、视频。。。

- number：js不区分小数和整数

  ```javascript
  123//整数123
  123.1//浮点数
  1.123e2//科学计数法
  -99//负数
  NaN//not a number
  Infinity//表示无限大
  
  ```

- 字符串

  ```javascript
  'abc'
  "abc"
  ```

- 布尔值

  ```javascript
  true
  false
  ```

- 逻辑运算

  ```javascript
  && 与
  || 或
  ! 非
  
  ```

- 比较运算符

  ```javascript
  = 赋值
  == 等于，类型不一样，值一样，也为true
  === 绝对等于（类型一样值一样才为true）
  //坚持使用===
  ```

  **须知：**NaN与所有都不相等，包括自己；只能通过isNaN（NaN）来判断这个数是否是NaN

  浮点数问题：

  ```javascript
  1/3===1-2/3
  false
  //避免使用浮点数计算，有精度问题
  //常用做法是：
  Math.abs(1/3-(1-2/3))<0.000001
  ```

- null和undefined

  - null是空
  - 后者是未定义

- 数组

  ```javascript
  var arr = [1,2,3,"hello",null,true];
  new Array(1,2,3,"hello",true);
  ```

- 对象是大括号，数组是中括号

  ```javascript
  var person = {
      name: "qin",
      age: 12,
      tags: ["22","sss","ss","hh"]
  }
  person.name;//访问
  ```

### 4 严格检查模式

```javascript
'use strict';//使用严格检查模式，必须写在第一行。若IDEA报错，要设置支持ES6语法
let i = 1; //局部变量建议使用let而不是var定义
```

## 3 数据类型

### 1 字符串

1. 正常字符串使用单或双引号包裹

2. 注意转移字符用 \ 

   ```javascript
   以下都在字符串包裹内使用
   \'
   \n
   \t
   \u##### //unicode字符
   \x##  Ascii字符
   ```

3. 多行字符串编写

   ```javascript
   var name= `hellos
   sdadd
   efefe
   deede`
   ```

4. 模板字符串

   ```javascript
   var name = 'qin';
   var msg = '你好,$(name)';
   ```

5. 字符串长度

   ```javascript
   console.log(str.length);
   console.log(str[0]);
   console.log(str.indexOf('t'));
   console.log(str.toUpperCase());
   console.log(str.subString(1,2));//包含第一个不包含第2个
   console.log(str.subString(1));//从第一个到最后
   ```

6. 字符串的可变性：不可变

### 2 数组

可以包含任意数据类型。

下标越界就会显示undefined，可以通过下标取值和赋值。

arr.length可以修改（会改变数组长度），改小会丢失元素，改大会在后面出现undefined。

indexOf（元素）：通过元素获取下标。

**slice()**：截取一部分返回新数组。

push（元素，元素）：往里面（尾部）加若干个元素，

pop（） ：弹出尾部的元素。

unshift(元素): 往首部加元素

shift（）：从首部弹出元素。

arr.sort():排序

arr.reserse():反转

arr1.concat(arr2):在arr1的基础上添加arr2的元素，返回新数组，原数组不改变

arr.join("-"):把数组的元素用符号拼接起来



### 3 对象

若干个键值对。

```javascript
var person = {
    name: "qin",
    age: 12,
    tags: ["22","sss","ss","hh"]
}
console.log(person.age);
person.name = "111";//赋值
//动态删减属性
delete person.name;
//动态添加
person.email = "123.com";
//判断属性是否属于对象
if ('age' in person)
//判断属性是否是对象自身拥有,有些属性如toString是自动继承来的，不是自身的
person.hasOwnProperty('age')
```

### 4 流程控制

if判断

```javascript
if(){
   
   }else if(){
            
            }else{
    
}
```

循环

```javascript
do{
    
}while()
while(){
      
      }
for(var i = 0;i<100;i++){
    
}
//数组循环遍历
var nums = [1,2,3]
for(var idx in nums){//idx是下标
    
}
nums.forEach(function(value){
    console.log(value);
})
```

### 5 Map和Set

```javascript
var names = ["tom",'Jack','eric'];
var map = new Map([['Tom',100],['Jack',101],['Eric',102]]);
console.log(map.get('Tom'));
map.set('ad',123);//新增或修改
map.delete('ad');
```

```javascript
var set = new Set([1,2,3,1,1]);//自动去重
set.add(2);
set.delete(2);
console.log(3 in set);
console.log(set.has(3));
```

```javascript
//遍历
for(var value of set){ //of用于数组，map和set的元素的遍历，in用于数组中下标的遍历
    console.log(value);
}
```



### 6 iterator

```javascript
for(var value of set){ //of用于数组，map和set的元素的遍历
    console.log(value);
}
for(var value of map){ //of用于数组，map和set的元素的遍历
    console.log(value);
}
for(var value of arr){ //of用于数组，map和set的元素的遍历
    console.log(value);
}
```

## 4 函数

### 1 定义函数

```javascript
//方式一
function abs(x){
    if (x>=0){
        return x;
    }else{
        return -x;
    }
}
```

一旦执行return，函数结束并返回结果，如果没有return，函数执行完后也会返回结果，结果是undefined。

```javascript
//方式二
var abs = function(x){ //等号右边是一个匿名函数，相当于py中的lambda。
    if (x>=0){
        return x;
    }else{
        return -x;
    }
}
var res = abs(-10);
```

参数问题：尽管定义了函数，但在调用时可以传入多个参数，也可以不传参（不会报错）。这就需要在函数内判断参数的合法性，如下，可以判断出没有传入参数的情况和参数不是数字的情况。

```javascript
var abs = function(x){ //等号右边是一个匿名函数，相当于py中的lambda。
    if(typeof x!== 'number'){
        throw 'Not a number!';//抛出异常
    }
    if (x>=0){
        return x;
    }else{
        return -x;
    }
}
```

传递多个参数：

函数会取第一个值为参数，其他传入的参数不处理，但是提供了一个arguments关键字来获取这些参数。arguments 是一个由传入的所有参数组成的数组。

```javascript
var abs = function(x){ //等号右边是一个匿名函数，相当于py中的lambda。
    console.log('x=='+x);
    for(var i of arguments){
        console.log(i);
    }
    if (x>=0){
        return x;
    }else{
        return -x;
    }
}
```

问题：arguments会包含所有参数，而我们需要获取多出来的参数：

rest：ES6新特性，获取除了已经定义的参数之外的所有多余参数。

```javascript
var abs = function(x,...rest){ //等号右边是一个匿名函数，相当于py中的lambda。
    //rest必须写在最后面，用...标识
    console.log(rest);
    if (x>=0){
        return x;
    }else{
        return -x;
    }
}
```

### 2 变量作用域

```javascript
//函数体外无法使用在函数内声明的变量
//函数嵌套定义的情况下，内部函数可以访问外部函数成员。
//内部函数变量和外部函数变量如果重名，则从自身开始，由内向外查找。

//全局对象window
var x = 'xx';
alert(x);
alert(window.x);//默认所有全局变量都绑定在window这个对象上。
window.alert(x);//alert也是window的函数
var myalert = window.alert;//在js中，函数名也可以作为对象来赋值，类似于python
```

规范：由于所有全局变量都绑定在window对象上，如果不同的js文件有同名的全局变量，就会冲突，如何处理？

```javascript
var KuangApp={};//定义一个对象，把自己的全局变量都给这个对象的属性上，降低全局名称冲突的问题
KuangApp.name="qin";
KuangApp.add=function(){
};

```

> 局部作用域let

```javascript
function aaa(){
    for(var i =0;i<10;i++){
        
    }
    console.log(i);//问题：i依然可以使用
}
```

```javascript
function aaa(){
    for(let i =0;i<10;i++){
        
    }
    console.log(i);//问题解决，这里会报错，无法使用for循环中定义的变量
}
```

建议使用let定义局部变量。

#### 常量const

在ES6之前，所有用大写字母定义的都是常量，建议不要修改（但可以改）。

在ES6引入关键字const

```javascript
const PI= '3.14';//该值无法改变
```

### 3 方法

```javascript

var yun = {
    name:'li',
    birth:2020,
    //方法的定义
    getAge: function(){
        var now = new Date().getFullYear();
        return now - this.birth;//this代表调用这个函数的对象。
    }
}
yun.name;
yun.getAge();
```

> apply

在js中指定this的指向。

```javascript
function getAgeold(){
        var now = new Date().getFullYear();
        return now - this.birth;
var yun = {
    name:'li',
    birth:2020,
    //方法的定义
    getAge:getAgeold
    }
}
yun.getAge();
getAgeold.apply(yun,[]);//代表指向yun这个对象，参数为空
```

## 5 内部对象

```javascript
typeof []
"object"
typeof {}
"object"
typeod 123

typeof 123
"number"
typeof '123'
"string"

typeof Math.abs
"function"
typeof undefined
"undefined"
```



### 1 Date

```javascript
var d = new Date();
undefined
console.log(d);
breadcrumbs.ts:117 Wed Jun 17 2020 16:36:03 GMT+0800 (中国标准时间)

d.getFullYear();//年
2020
d.getMonth();//范围是0-11
d.getDate();//日
17
d.getDay();//星期几
3
d.getHours();
d.getMinutes();
d.getSeconds();
d.getTime();//时间戳，全世界唯一
d.toLocaleDateString()
"2020/6/17"
d.toLocaleString()
"2020/6/17 下午4:36:03"
```

### 2 JSON对象

- 轻量级数据交换格式
- 层次结构清晰简洁
- 传输效率高

js支持的对象都可以用json来表示。

格式：

- 对象都用｛｝
- 数组用[]
- 键值对使用key：value

```javascript
var user = {
    name:'qin',
    age:3,
    sex:'男'
}
//对象转JSON,本质是字符串
var jsonuser = JSON.stringify(use);
//字符串转对象
var u = JSON.parse(jsonuser);
```

### 3 Ajax

- 原生的js写法，xhr异步请求
- jQuery封装好的方法，$("#name").ajax("")
- axios请求

## 6 面向对象编程

### 1 什么是面向对象

```javascript
var user = {
    name:'qin',
    age:3,
    sex:'男',
    run: function(){
    console.log(this.name+' running...')
    }
}

var xiaoming ={
    name:'xiaoming'
}

xiaoming.__proto__=user;
//这样，xiaoming变量就有了run方法。
```

#### class继承

class关键字是在ES6引入的，很多浏览器可能还不支持。

1. 定义一个类

```javascript
//原来的方式
function Student(name){
    this.name = name;
}
//给student添加一个方法
Student.prototype.hello=function(){
    alert('hello');
}

//使用class关键字。 ES6
class Student{
    constructor(name){
        this.name = name;
    }
    hello(){
        alert('hello');
    }
}

var xiaom = new Student('xiaom');
xiaom.hello();
```

2. 继承

   ```javascript
   class Student{
       constructor(name){
           this.name = name;
       }
       hello(){
           alert('hello');
       }
   }
   class xiaoStudent extends Student{
       constructor(name,age){
           super(name);
           this.age=age;
       }
       myAlert(){
           alert('我是小学生');
       }
   }
   ```

   每个对象都有一个prototype属性指向该对象的原型对象

## 7 操作BOM对象

> 浏览器

js和浏览器的关系：

js的诞生就是为了在浏览器中运行

BOM：浏览器对象模型

内核：

- IE：6-11
- Chrome
- Safari
- FireFox
- Opera

三方（使用以上内核）

- QQ
- 360

> window对象，代表浏览器窗口

```javascript
window.innerHeight
526
window.innerWidth
394
window.outerHeight
680
window.outerWidth
1280
```

> navigator

window的一个属性，封装了浏览器信息

```javascript
navigator.appName
"Netscape"
navigator.appVersion
"5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"

navigator.userAgent
"Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"
navigator.platform
"Win32"
```

大多数时候不使用navigator对象，因为会被人为修改。

不建议使用这些属性来判断和编写代码。

> screen

代表屏幕

```javascript
screen.width
1280  //这个的单位是px
screen.height
720//获取屏幕的大小
```

> location（重要）

代表当前的url信息。

```javascript
host: "www.jetbrains.com"
hostname: "www.jetbrains.com"
href: "https://www.jetbrains.com/idea/download/?utm_source=product&utm_medium=link&utm_campaign=IC&utm_content=2019.3#section=windows"
origin: "https://www.jetbrains.com"
pathname: "/idea/download/"
port: ""
protocol: "https:"
reload: ƒ reload()//这个代表刷新网页
//设置新的地址,该代码一运行就会跳转到设置的url
location.assign('https://bolg.xx.com');
```

> document 内容：DOM

代表当前页面

```javascript
document.title='百度'
"百度"
document.title='百度一下'
"百度一下"
var al =  document.getElementById("app");//获取具体的文档树节点，就能动态增删节点
document.cookie //可以获取cookie
"BAIDUID=E8F0E91FC3253B5AC2B3F1D73BB82CD7:FG=1; PSTM=1587204822; BIDUPSID=43893C7F0B3F4552CD05DB0940BF5EB4; "
```

一些网站可以恶意获取cookie上传到他的服务器。为了解决这个问题，服务器端可以设置cookie：httpOnly

> history （不建议使用）

代表浏览器的历史记录。

```javascript
history.back();//就是浏览器后退
history.forward();//浏览器前进
```

## 8 操作DOM（重要）

DOM：文档对象模型，就是window.document对象

整个浏览器网页就是一个DOM树形结构

- 更新：更新DOM节点
- 遍历：得到DOM节点
- 删除：删除DOM节点
- 添加：新增新的DOM节点



### 1 获得DOM节点

DOM节点就是html中的一个个标签

```html
<div>
    <h1></h1>
    <p id="h1"></p>
    <p class="h1"></p>
</div>
<script>
    var h1= document.getElementByTagName('div');
    var h2 = document.getElementById('h1');
    var h3 = document.getElementByClassName('h1');
    var h4 = h1.children;//获取父节点下的所有子节点
    //h1.firstChild
    //h1.lastChild
    
</script>
</body>
```

这是原生代码，以后尽量使用jQuery

### 2 更新节点

```javascript
var h2 = document.getElementById('id1');
h2.innerText='123';//修改本文的值
h2.innerHTML('<h1>哈哈</h1>');//嵌入超文本，可以解析html文本标签
h2.style.color = 'red'; //设置颜色
h2.style.fontSize = '20px'; 
h2.style.padding = '20em';
```

### 3 删除

步骤：

- 获取父节点
- 通过父节点删除自己

```javascript
var h1 = document.getElementById('p1');
var f = h1.parentElement;
f.removeChild(h1);
var c = f.children;
f.removeChild(c[0]);
//删除节点时，children属性是在动态变化的
```

### 4 插入节点

获得了某个DOM节点，假设这个节点是空的，用innerHTML就可以增加元素了，若已经存在元素了，就会覆盖原来的。

所以一般用追加操作。

```html
<body>
<p id="js"></p>
<div id="list">
    <h1></h1>
    <p id="h1"></p>
    <p class="h1"></p>
    <p></p>
    <p></p>
    <p></p>
</div>

<script>
    var list = document.getElementById('list');
    var js = document.getElementById('js');
    list.appendChild(js);//追加节点,这样原来的标签会移动到list里面
    
    //通过js创建新节点
    var newP = document.createElement('p');//创建一个p标签
    newP.id = 'newp';
    newP.innerText('文本');
    list.appendChild(newP);
    
    //创建特殊标签，比如script标签
    var s = document.createElement('script');
    s.setAttribute('type','text/javascript');
   
</script>
</body>
```

还可以修改样式style：

```javascript
//创建一个style标签
    var style = document,createElement('style');
    style.setAttribute('type','text/css');
    style.innerHTML='body{background-color: chartreuse;}';
    document.getElementByTagName('head')[0].appendChild(style);
    
```

但是操作繁琐，所以有了jQuery。



还可以这样插入：

```javascript
var list = document.getElementById('list');
var ee = document.getElementById('ee');
var li = document.getElementById('li');
list.insertBefore(li,ee);//把li节点插入到list中，位于ee节点之前
list.replaceChild();//替换
```

## 9 操作表单（用于输入验证）

表单：form，DOM树

- 文本框 text
- 下拉框 select
- 单选框 radio
- 多选框 checkbox
- 密码框 password
- 。。。

表单的目的是提交信息，js的目的是获取表单信息做一些操作。

```html
<form action="#" method="post">
    <p>
        <span>用户名：</span><input type="text" id="username">
    </p>
    <!--多选框的值就是定义好的value值-->
    <p>
        <span>性别：</span>
        <input type="radio" name="sex" value="man" id="boy"> 男
        <input type="radio" name="sex" value="woman" id="girl"> 女
    </p>
    <input type="submit"> <!--提交按钮-->
</form>

<script>
    var input = document.getElementById('username');
    var valtext = input.value;//获取文本框的值
    input.value='123';//修改值
    
    var boy_radio = document.getElementById('boy');
    var ifcheck = boy_radio.checked;//如果选中则为true，否则为false
    boy_radio.checked = true;//可以通过代码设置为选中状态（操作表单）
    boy_radio.value;//该值为man
</script>
```



### 提交表单

```html
<head>
<script src="https://cdn.bootcss.com/blueimp-md5/2.10.0/js/md5.min.js"></script>
</head>

<body>
    <form method="post" action="#">
    <p>
        <span>用户名：</span><input type="text" id="username" name="username">
    </p>
    <p>
        <span>密码：</span><input type="password" id="password" name="pwd">
    </p>
<!--    <input type="submit"> &lt;!&ndash;提交按钮&ndash;&gt;-->
<!--    给按钮绑定点击事件test()-->
    <button type="submit" onclick="test()">提交</button>
</form>

<script>
    function test(){
        alert(1);
        var name = document.getElementById('username');
        var pd = document.getElementById('password');
        //加密
        pd.value = md5(pd.value);
        console.log(name.value);   
    }
    </script>
</body>
```

改为表单绑定提交，而不是按钮绑定提交（如下）。这样的话，test函数最后一句必须返回一个true才能提交成功，返回false会阻止提交。

```html

<form method="post" action="#" onsubmit='return test()'>
    <p>
        <span>用户名：</span><input type="text" id="username" name="username">
    </p>
    <p>
        <span>密码：</span><input type="password" id="password" name="pwd">
    </p>
<!--    <input type="submit"> &lt;!&ndash;提交按钮&ndash;&gt;-->
<!--    绑定事件test()-->
    <button type="submit">提交</button>
</form>
```

还可以进一步修改来增加安全性：

```html
<p>
    <span>密码：</span><input type="password" id="input-psd">
</p>
<input type="hidden" id="md5-psd" name="password">

<script>
	function test(){
        var md5_psd=document.getElementById('md5-psd');
        var input_psd=document.getElementById('input-psd');
        md5_psd.value = md5(inout_psd.value);
    }
</script>
```

## 10 jQuery

一个封装了大量js函数的库。[文档](https://jquery.cuishifeng.cn/)

获取jQuery：

- 下载js文件，script的src使用js文件目录
- 找一些cdn链接，放到script的src中。

```html
<script src="https://code.jquery.com/jquery-3.5.1.js" integrity="sha256-QWo7LDvxbWT2tbbQ97B53yJnYU3WhH/C8ycbRAkjPDc=" crossorigin="anonymous"></script>
```

The `integrity` and `crossorigin` attributes are used for [Subresource Integrity (SRI) checking](https://www.w3.org/TR/SRI/). This allows browsers to ensure that resources hosted on third-party servers have not been tampered with. Use of SRI is recommended as a best-practice, whenever libraries are loaded from a third-party source. Read more at [srihash.org](https://www.srihash.org/)

jQuery的使用公式：$(选择器).action()

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://code.jquery.com/jquery-3.5.1.js" integrity="sha256-QWo7LDvxbWT2tbbQ97B53yJnYU3WhH/C8ycbRAkjPDc=" crossorigin="anonymous"></script>

</head>
<body>
<a href="#" id="test-j">点我</a>

<script>

    //选择器就是css选择器
    $("#test-j").click(function(){
        alert("aa");
    })
</script>
</body>
</html>
```

### 选择器

选择某个节点

```javascript
//选择器
document.getElementById();
document.getElementByTagName();
document.getElementByClassName();
//jQuery选择器
$('标签')
$('#id')
$('.classname')
```

### 事件

- 鼠标事件
- 键盘事件
- 其他事件

```javascript
$('p').mousedown()//按下
$('p').mouseleave()//离开
$('p').mousemove()//移动
。。。
```

使用举例：

```html
mouse: <span id="mouseMove"></span>
<div id="divmove">
    在这里移动鼠标试试
</div>

<script>
    //当网页元素加载完毕之后加载事件
    $(document).ready(function(){
        //网页加载完成后执行的代码写在这里面

    });

    $(function(){
        //网页加载完成后执行的代码写在这里面,是上面代码的另一种简单写法
        $('#divmove').mousemove(function(e){
            $('#mouseMove').text('x:'+e.pageX+'y:'+e.pageY);
        })
    })
</script>
```

### 操作DOM元素

节点文本操作

```html
<ul id="test-ul">
    <li class="js">javascript</li>
    <li name="py">python</li>
</ul>

<script>
    $('#test-ul li[name=py]').text();//拿到name属性为py的li标签里面的值 
    $('#test-ul li[name=py]').text('123');//往里面设置值
    $('p').html();//获得值
    $('p').html('<p>hahah</p>');//设置值
</script>
```

css操作

```javascript
$("p").css({ "color": "#ff0011", "background": "blue" });
```

元素显示和隐藏：本质：display：none

```javascript
$("p").show();//显示
$("p").hidden();//隐藏
```

其他：

```javascript
$(window).height();
$(window).width();
```

