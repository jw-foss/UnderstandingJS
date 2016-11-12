   在学习JS的过程中，相信你对这一句话并不陌生：`在JS里，一切事物皆为对象`，今天就来谈谈自己对于JS面对对象的理解。

**"面对对象，实际上就是把你的需求抽象成一个对象，然后针对这个对象来进行分析。这个对象就被称作为一个类。而面对对象来编程最大的一个特点就是来封装这个类"** 

由于在JS当中是没有class(ES2015当中有class，但实际上是语法糖)这个说法的，这也使得JS异常的灵活，JS当中的关于类几乎都是通过一些特性的模仿来实现的。

`创建一个类`
>var Student = function(name, id, gender) {  
 var num = 1;  
function getNum(){ console.log(this.num) }  
//判断this是否是当前这个对象, 防止漏写new操作符带来的问题
if (this instanceof Student){  
  this.name = name;  
  this.id = id;  
  this.gender = gender;  
 } else {  
 return new Student(name, id, grade);  
}  
}  
Student.prototype.sayName = function() {  
  return this.name;  
}  

这样就创建了一个Student的类, 还为这个类上添加了一个公共的方法`sayName`  
这个时候就可以通过`var xiaoMing = new Student('XiaoMing', 1, 'boy')`  来创建一个Student的实例xiaoMing  
`xiaoMing instanceof Student // true`  

**"通过`new`关键字创建对象的实质是对新对象`this`的值不断的赋值，并将`prototype`指向类的`prototype`所指向的对象， 而在类的构造函数之外通过`.`语法定义的属性与方法是不会被添加到新的对象上去的，而类的原型`prototype`上定义的属性在新对象里面就能直接使用，因为新对象的`prototype`和类的`prototype`指向的是同一个对象"**

但是如果你想访问那些私有的属性或方法应该怎么做呢? 答案是通过闭包的方式来做到（关于闭包以后会说到）。

创建完类之后，就要接下来对继承进行一定的了解了，类与类之间也是有继承的关系的。继承大概分6种：

>1. 类式继承（原型链式）
2. 构造函数式继承
3. 组合继承
4. 原型式继承
5. 寄生式继承
6. 寄生组合式继承

在这里仅对类式继承和寄生组合式继承做分析， 其他4种方法请查阅[JavaScript高级程序设计](http://item.jd.com/10951037.html)`第六章`

`1. 类式继承:`
> function Father(){  
  this.faterMessage = 'I am the father' ;  
}  
Father.prototype.adult = function(){  
 console.log(this.faterMessage)  
}    
function Son() {  
 this.childMessage = 'I am the child';  
}  
Son.prototype = new Father();  
Son.prototype.child = function() {  
  console.log(this.childMessage);  
}  

**类的原型对象的作用就是为类的原型添加共有方法，但是类不能直接访问这些属性和方法，必须通过原型`prototype`来访问，但是实例化一个父类的时候，新创建的对象复制了父类的构造函数内的属性与方法，并且将原型`__proto__`指向了父类的原型对象也就是`prototype`，这样就子类就拥有了父类原型对象上的属性与方法，当我们将这个新创建的对象赋值给子类的原型的时候，子类的原型就可以访问到父类的原型属性和方法了。**

但是这种继承方法，有2个缺点：
1. 如果父类中的共有属性是引用类型，会被子类的实力更改所影响。
2. 由于继承是通过原型对父类的实例化实现的，因此在创建父类的时候是无法像父类传递参数的，因此也无法对父类构造函数里的属性进行初始化。

`4.寄生组合式继承:`

 2006年**Douglas Crockford**发表的一篇名为《JavaScript中原型式继承》的文章中写到：

> function inheritation(o) {  
//过渡函数对象  
 function F() {}  
F.prorotype = o;  
//返回该过渡对象的实例，这个实例继承了父对象  
return new F();  
}  

如何来理解这个函数呢，看下面的例子：

> var obj = {  
name: 'inheritation'  ,  
methods: ['prototype', 'combination']  
}  
var child1 = inheritation(obj);  
var child2 = inheritation(obj);  
child1.name = 'child1';  
child2.name = 'child2';  
child1.methods.push('parasitization');  
console.log(child1.name); //child1  
console.log(child2.name); //child2  
console.log(child2.methods); //['prototype', 'combination', 'parasitization']

同样的问题出现了，引用类型的属性是被父子共用的。在这个方法上，**Dogulars Crockford** 对其进行了拓展。

> function createChild(obj) {  
var o = new inheritation(obj);  
o.someFn = function(){};    
return o;  
}

这样一来，新创建的对象里， 不仅仅有父类的属性与方法，也有添加新的属性和方法。

寄生组合式继承，就是在上面这种情况下做出了一点改动。

> function inheritPrototype(child, father) {  
 //对父类的原型进行复制  
 var m = inheritation(father.prototype)  
//修正被修改的constructor  
 m.constructor = child;
 child.prototype = m;  
}

经过这样的修改之后，子类的原型就继承了父类的原型，而且没有执行父类的构造函数。由于篇幅原因就不在这里写测试代码了， 感兴趣的同学可以自行测试一下。

`多继承方式：`
如果有了解过[jQuery.js](jQuery.com)的同学应该知道，`jQuery`里面的extends方法用来扩展原型和插件，基本就是这样的原理。对`要复制的对象`的属性进行复制，当然`deep`或者`shallow`拷贝都是可以的，多继承方式就是将目标对象与需要继承的属性进行一个遍历的操作来达到复制效果。上代码：

> var extends = function(target) {  
  var i = 1,  
len = arguments.length,  
arg;  
for (; i < len; i++) {  
 arg = arguments[i];
 for(var prop in arg) {  
  target[prop] = arg[prop];  
  }  
}  
return target;  
}

这里做的仅仅是浅拷贝， 如果要进行深拷贝应该怎么做呢？ 很简单， 将要复制的属性的键值分别取出赋值即可，感兴趣的话可以自己去实现一下看看。
