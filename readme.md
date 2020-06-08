# 王慧 part1 模块二

## 简答题

### 1.描述引用计数的工作原理和优缺点。
答：引用计数的核心思想是设置引用数，判断当前引用数是否为0；即引用计数器，当引用关系改变时修改引用数字，一旦引用数字为0时立即回收

优点：1.发现垃圾时立即回收；
    2.最大限度减少程序暂停；
    
缺点：1.无法回收循环引用的对象；
2.时间开销大

### 2.描述标记整理算法的工作原理和优缺点。
答：先理解下标记清除算法分标记和清除二个阶段完成，首先遍历所有对象找标记活动对象（找到可达对象，并打上标记），接着遍历所有对象清除没有标记对象，最后，回收相应的空间；

标记整理可以看做是标记清除的增强，标记阶段的操作和标记清除一致，清除阶段会先执行整理，移动对象位置；

优点：减少碎片化空间

缺点：不会立即回收垃圾对象


### 3.描述V8中新生代存储区垃圾回收的流程。
答：V8将存储空间一分为二，小空间用于存储新生代对象；

新生代对象回收过程采用复制算法和标记整理法。
新生代内存区分为二个等大小空间，即使用空间（From）,空闲空间(To)；活动对象存储于From空间；标记整理后将活动对象拷贝至To;From与To交换空间完成释放；

### 4.描述增量标记算法在何时使用，及工作原理。
答：增量标记算法在老生代存储区垃圾回收时使用来提升效率。

 老生代区域的数据比较大，不适合一分为二（会造成空间的浪费），同时，复制数据会比较耗费时间，所以老生代区域垃圾回收不适合复制算法；

程序暂停执行的时候，遍历对象进行标记（标记的是老生代对象），遍历完直接可达对象后，继续程序的执行；程序暂停执行，继续标记对象（间接可达对象），标记对象和程序的执行交替进行；直到标记完所有对象后，完成清除；程序继续执行；
增量标记即标记过程分步完成，不是一次完成，一次完成会导致程序暂停时间较长，影响用户体验。

## 代码题1
基于以下代码完成下面的四个练习
```
const fp = require ('lodash/fp')
//数据
//horsepower 马力，dollar_value 价格，in_stock 库存
const cars =[
  {
    name: "Ferrari FF",
    horsepower: 660,
    dollar_value: 700000,
    in_stock:true
  } ,
  {
    name: "Spyker C12 Zagato",
    horsepower: 650,
    dollar_value: 648000,
    in_stock:false
  } ,
  {
    name: "Jaguar XKR-S",
    horsepower: 550,
    dollar_value: 132000,
    in_stock:false
  } ,
  {
    name: "Audi R8",
    horsepower: 525,
    dollar_value: 114200,
    in_stock:false
  } ,
  {
    name: "Aston Martin One-77",
    horsepower: 750,
    dollar_value: 1850000,
    in_stock:true
  } ,
  {
    name: "Pagani Huayra",
    horsepower:700,
    dollar_value: 1300000,
    in_stock:false
  }
]

```
### 练习1：
使用函数组合fp.flowRight ()重新实现下面这个函数
```
let isLastInStock=function (cars) { 
／／获取最后一条数据
    let last car= fp.last (cars)
    ／／获取最后一条数据的 in_stock 属性值
    return fp.prop ('in_stock',last car)
｝
```
答：
```
let isLastInStock = fp.flowRight(fp.prop('in_stock'),fp.last);
```
### 练习2：
使用fp.flowRight ()、fp.prop()和fp.first()获取第一个car的name
答：
```
let  firtCarName = fp.flowRight(fp.prop("name"),fp.first)(cars);
console.log('firtCarName:',firtCarName);
```

### 练习3：
使用帮助函数_average 
重构averageollarValue,使用函数组合的方式实现
```
let _average = function(xs) {
    return fp.feduce(fp.add, 0 , xs) / xs.length
}
//<-无须改动

let averageDollarValue = function (cars) {
  let dollar_values = fp.map(function(car){
      return car.dollar_value 
  },cars) ;
  
  return _average(dollar_values)
}
```

答：
```
let averageDollarValue1 = fp.flowRight(_average,fp.map(fp.prop('dollar_value')))
console.log('averageDollarValue1:',averageDollarValue1(cars));
```

### 练习4：
使用flowRight写一个sanitizeNames()函数，返回一个下划线链接的小写字符串，把数组中的name转换为这种形式：例如：sanitizeNames(["Hello World"]) =>["Hello_World"]

```
let _underscore = fp.replace(/\w+/g, '_') 
//无须改动，并在sanitizeNames中使用它
```

```
/*使用flowRight写一个sanitizeNames()函数，返回一个下划线链接的小写字符串，
把数组中的name转换为这种形式：例如：sanitizeNames(["Hello World"]) =>["hello_world"]
*/

let _underscore = fp.replace(/\s+/g, '_') ;

let sanitizeNames = fp.map(fp.flowRight(_underscore,fp.toLower,fp.prop('name')));
console.log('sanitizeNames:',sanitizeNames(cars));
```

PS:题目中的_underscore是不是拼错了，我在答案中按照我的理解重写了

## 代码题2
基于下面提供的代码，完成后续的四个练习
```
//support.js
class Container {
    static of (value) {
        return new Container(value)
    }
    
    constructor (value) {
        this._value = value
    }
    
    map (fn) {
        return Container.of(fn(this._value))
    }
}

class Maybe {
    static of (x) {
        return new Maybe(x)
    }
    
    isNothing () {
        return this._value === null || this._value === undefined
    }
    
    constructor (x) {
        this._value = x
    }
    
    map (fn) {
        return this.isNothing() ? this : Maybe.of(fn(this._value))
    }
}

module.exports = {
    Maybe,
    Container
}

```
### 练习1：
使用fp.add(x,y) 和fp.map(f,x) 创建一个能让functor里的值增加的函数ex1

```
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support');

let maybe = Maybe.of([5, 6, 1]);
let ex1 = //... 需要实现的位置
```
答：
```
let ex1 = function (x) { 
    return fp.map(fp.add(x),maybe._value);
}

console.log('result1:',ex1(1));
```
### 练习2：
实现一个函数ex2，能够使用fp.first获取列表的第一个元素
```
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support');

let xs = Container.of(['do', 'ray', 'me', 'fa', 'so', 'la', 'ti', 'do']);
let ex2 = //... 需要实现的位置
```
答：
```
let ex2 = function () {
  return fp.first(xs._value)
}

console.log('ex2 result:', ex2());

```

### 练习3：
实现一个函数ex3，使用safeProp和fp.first找到user的名字的首字母
```
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support');

let safeProp = fp.curry(function (x, o) {
    return Maybe.of(o[x])
});

let user = {
    id: 2,
    name: "Albert"
}

let ex3 = //... 需要实现的位置
```

答：
```
 let ex3 = function (atr,obj) {
   return fp.first(safeProp(atr,obj)._value)
 }

 console.log('ex3 result:',ex3('name',user));
```
### 练习4：
使用Maybe重写ex4,不要有if语句
```
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support');

let ex4 = function (n) {
   if (n) {
    return parseInt(n)
   }
};
```
答：
```
let ex4 = function (n) {
  return Maybe.of(n).map(parseInt)._value
}
```
