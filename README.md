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
