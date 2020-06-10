# Javascript 面向对象编程（一）：封装
为了解决从原型对象生成实例的问题，Javascript提供了一个构造函数（Constructor）模式。

所谓"构造函数"，其实就是一个普通函数，但是内部使用了this变量。对构造函数使用new运算符，就能生成实例，并且this变量会绑定在实例对象上。

比如，猫的原型对象现在可以这样写，
```
　　function Cat(name,color){

　　　　this.name=name;

　　　　this.color=color;

　　}
```
我们现在就可以生成实例对象了。
```
　　var cat1 = new Cat("大毛","黄色");

　　var cat2 = new Cat("二毛","黑色");

　　alert(cat1.name); // 大毛

　　alert(cat1.color); // 黄色
```
这时cat1和cat2会自动含有一个constructor属性，指向它们的构造函数。
```
　　alert(cat1.constructor == Cat); //true

　　alert(cat2.constructor == Cat); //true
```
Javascript还提供了一个instanceof运算符，验证原型对象与实例对象之间的关系。
```
　　alert(cat1 instanceof Cat); //true

　　alert(cat2 instanceof Cat); //true
```
### 构造函数模式的问题
构造函数方法很好用，但是存在一个浪费内存的问题。

请看，我们现在为Cat对象添加一个不变的属性type（种类），再添加一个方法eat（吃）。那么，原型对象Cat就变成了下面这样：
```
　　function Cat(name,color){

　　　　this.name = name;

　　　　this.color = color;

　　　　this.type = "猫科动物";

　　　　this.eat = function(){alert("吃老鼠");};

　　}
```
还是采用同样的方法，生成实例：
```
　　var cat1 = new Cat("大毛","黄色");

　　var cat2 = new Cat ("二毛","黑色");

　　alert(cat1.type); // 猫科动物

　　cat1.eat(); // 吃老鼠
```
表面上好像没什么问题，但是实际上这样做，有一个很大的弊端。那就是对于每一个实例对象，type属性和eat()方法都是一模一样的内容，每一次生成一个实例，都必须为重复的内容，多占用一些内存。这样既不环保，也缺乏效率。
```
　　alert(cat1.eat == cat2.eat); //false
```
能不能让type属性和eat()方法在内存中只生成一次，然后所有实例都指向那个内存地址呢？回答是可以的。
### Prototype模式

Javascript规定，每一个构造函数都有一个prototype属性，指向另一个对象。这个对象的所有属性和方法，都会被构造函数的实例继承。

这意味着，我们可以把那些不变的属性和方法，直接定义在prototype对象上。
```
　　function Cat(name,color){

　　　　this.name = name;

　　　　this.color = color;

　　}

　　Cat.prototype.type = "猫科动物";

　　Cat.prototype.eat = function(){alert("吃老鼠")};
```
然后，生成实例。
```
　　var cat1 = new Cat("大毛","黄色");

　　var cat2 = new Cat("二毛","黑色");

　　alert(cat1.type); // 猫科动物

　　cat1.eat(); // 吃老鼠
```
这时所有实例的type属性和eat()方法，其实都是同一个内存地址，指向prototype对象，因此就提高了运行效率。
```
　　alert(cat1.eat == cat2.eat); //true
  ```
  * isPrototypeOf()

这个方法用来判断，某个proptotype对象和某个实例之间的关系。
```
　　alert(Cat.prototype.isPrototypeOf(cat1)); //true

　　alert(Cat.prototype.isPrototypeOf(cat2)); //true
```
 * hasOwnProperty()

每个实例对象都有一个hasOwnProperty()方法，用来判断某一个属性到底是本地属性，还是继承自prototype对象的属性。
```
　　alert(cat1.hasOwnProperty("name")); // true

　　alert(cat1.hasOwnProperty("type")); // false
```
 * in运算符

in运算符可以用来判断，某个实例是否含有某个属性，不管是不是本地属性。
```
　　alert("name" in cat1); // true

　　alert("type" in cat1); // true
```
in运算符还可以用来遍历某个对象的所有属性。
```
　　for(var prop in cat1) { alert("cat1["+prop+"]="+cat1[prop]); }
  ```
  创建自定义对象（实例化类）的代码：
  ```
var zhang = new Person("ZhangSan", "man");
console.log(zhang.getName()); // "ZhangSan"
var chun = new Person("ChunHua", "woman");
console.log(chun.getName()); // "ChunHua"
当代码var zhang = new Person("ZhangSan", "man")执行时，其实内部做了如下几件事情：
```
创建一个空白对象（new Object()）。
* 拷贝Person.prototype中的属性（键值对）到这个空对象中（我们前面提到，内部实现时不是拷贝而是一个隐藏的链接）。
* 将这个对象通过this关键字传递到构造函数中并执行构造函数。
* 将这个对象赋值给变量zhang
为了证明prototype模版并不是被拷贝到实例化的对象中，而是一种链接的方式，请看如下代码：
```
function Person(name, sex) {
    this.name = name;
    this.sex = sex;
}
Person.prototype.age = 20;
var zhang = new Person("ZhangSan", "man");
console.log(zhang.age); // 20
// 覆盖prototype中的age属性
zhang.age = 19;
console.log(zhang.age); // 19
delete zhang.age;
// 在删除实例属性age后，此属性值又从prototype中获取
console.log(zhang.age); // 20
```
这种在JavaScript内部实现的隐藏的prototype链接，是JavaScript赖以生存的温润土壤， 也是模拟实现继承的基础。
## js继承
### 原型式继承与类式继承
类式继承是在子类型构造函数的内部调用超类型的构造函数。
严格的类式继承并不是很常见，一般都是组合着用：
```
function Super(){
    this.colors=["red","blue"];
}
 
function Sub(){
    Super.call(this);
}
```
原型式继承是借助已有的对象创建新的对象，将子类的原型指向父类，就相当于加入了父类这条原型链
### 原型链继承
为了让子类继承父类的属性（也包括方法），首先需要定义一个构造函数。然后，将父类的新实例赋值给构造函数的原型。代码如下：
```
<script>
    function Parent(){
        this.name = 'mike';
    }

    function Child(){
        this.age = 12;
    }
    Child.prototype = new Parent();//Child继承Parent，通过原型，形成链条

    var test = new Child();
    alert(test.age);
    alert(test.name);//得到被继承的属性
    //继续原型链继承
    function Brother(){   //brother构造
        this.weight = 60;
    }
    Brother.prototype = new Child();//继续原型链继承
    var brother = new Brother();
    alert(brother.name);//继承了Parent和Child,弹出mike
    alert(brother.age);//弹出12
</script>
```
以上原型链继承还缺少一环，那就是Object，所有的构造函数都继承自Object。而继承Object是自动完成的，并不需要我们自己手动继承，那么他们的从属关系是怎样的呢？

确定原型和实例的关系
<br>可以通过两种方式来确定原型和实例之间的关系。操作符instanceof和isPrototypeof()方法：
 ```
 alert(brother instanceof Object)//true
alert(test instanceof Brother);//false,test 是brother的超类
alert(brother instanceof Child);//true
alert(brother instanceof Parent);//true
```
只要是原型链中出现过的原型，都可以说是该原型链派生的实例的原型，因此，isPrototypeof()方法也会返回true

在js中，被继承的函数称为超类型（父类，基类也行），继承的函数称为子类型（子类，派生类）。使用原型继承主要由两个问题：
一是字面量重写原型会中断关系，使用引用类型的原型，并且子类型还无法给超类型传递参数。

伪类解决引用共享和超类型无法传参的问题，我们可以采用“借用构造函数”技术
### 借用构造函数（类式继承）
 ```
 <script>
    function Parent(age){
        this.name = ['mike','jack','smith'];
        this.age = age;
    }

    function Child(age){
        Parent.call(this,age);
    }
    var test = new Child(21);
    alert(test.age);//21
    alert(test.name);//mike,jack,smith
    test.name.push('bill');
    alert(test.name);//mike,jack,smith,bill
</script>
```
借用构造函数虽然解决了刚才两种问题，但没有原型，则复用无从谈起，所以我们需要原型链+借用构造函数的模式，这种模式称为组合继承
###组合继承
```
<script>
    function Parent(age){
        this.name = ['mike','jack','smith'];
        this.age = age;
    }
    Parent.prototype.run = function () {
        return this.name  + ' are both' + this.age;
    };
    function Child(age){
        Parent.call(this,age);//对象冒充，给超类型传参
    }
    Child.prototype = new Parent();//原型链继承
    var test = new Child(21);//写new Parent(21)也行
    alert(test.run());//mike,jack,smith are both21
</script>
```
组合式继承是比较常用的一种继承方法，其背后的思路是 使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。这样，既通过在原型上定义方法实现了函数复用，又保证每个实例都有它自己的属性。

call()的用法：调用一个对象的一个方法，以另一个对象替换当前对象。

call([thisObj[,arg1[, arg2[, [,.argN]]]]]) 
### 原型式继承
这种继承借助原型并基于已有的对象创建新对象，同时还不用创建自定义类型的方式称为原型式继承
```
<script>
     function obj(o){
         function F(){}
         F.prototype = o;
         return new F();
     }
    var box = {
        name : 'trigkit4',
        arr : ['brother','sister','baba']
    };
    var b1 = obj(box);
    alert(b1.name);//trigkit4

    b1.name = 'mike';
    alert(b1.name);//mike

    alert(b1.arr);//brother,sister,baba
    b1.arr.push('parents');
    alert(b1.arr);//brother,sister,baba,parents

    var b2 = obj(box);
    alert(b2.name);//trigkit4
    alert(b2.arr);//brother,sister,baba,parents
</script>
```
原型式继承首先在obj()函数内部创建一个临时性的构造函数 ，然后将传入的对象作为这个构造函数的原型，最后返回这个临时类型的一个新实例。
### 寄生式继承
这种继承方式是把原型式+工厂模式结合起来，目的是为了封装创建的过程。
```<script>
    function create(o){
        var f= obj(o);
        f.run = function () {
            return this.arr;//同样，会共享引用
        };
        return f;
    }
</script>
```
### 组合式继承的小问题
组合式继承是js最常用的继承模式，但组合继承的超类型在使用过程中会被调用两次；一次是创建子类型的时候，另一次是在子类型构造函数的内部

那么寄生组合继承，解决了两次调用的问题。
### 寄生组合式继承
```
<script>
    function obj(o){
        function F(){}
        F.prototype = o;
        return new F();
    }
    function create(parent,test){
        var f = obj(parent.prototype);//创建对象
        f.constructor = test;//增强对象
    }

    function Parent(name){
        this.name = name;
        this.arr = ['brother','sister','parents'];
    }

    Parent.prototype.run = function () {
        return this.name;
    };

    function Child(name,age){
        Parent.call(this,name);
        this.age =age;
    }

    inheritPrototype(Parent,Child);//通过这里实现继承

    var test = new Child('trigkit4',21);
    test.arr.push('nephew');
    alert(test.arr);//
    alert(test.run());//只共享了方法

    var test2 = new Child('jack',22);
    alert(test2.arr);//引用问题解决
</script>
```
全局函数apply和call可以用来改变函数中this的指向，如下：
```
 // 定义一个全局函数
    function foo() {
        console.log(this.fruit);
    }
    
    // 定义一个全局变量
    var fruit = "apple";
    // 自定义一个对象
    var pack = {
        fruit: "orange"
    };
    
    // 等价于window.foo();
    foo.apply(window);  // "apple",此时this等于window
    // 此时foo中的this === pack
    foo.apply(pack);    // "orange"
    ```
