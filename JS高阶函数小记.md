# JS高阶函数小计
高阶函数英文叫Higher-order function。那么什么是高阶函数？

JavaScript的函数其实都指向某个变量。既然变量可以指向函数，函数的参数能接收变量，那么一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函数。

一个最简单的高阶函数：

function add(x, y, f) {
    return f(x) + f(y);
}






## map/reduce


### map
举例说明，比如我们有一个函数f(x)=x2，要把这个函数作用在一个数组[1, 2, 3, 4, 5, 6, 7, 8, 9]上，就可以用map实现如下：

map

*由于map()方法定义在JavaScript的Array中，我们调用Array的map()方法*，传入我们自己的函数，就得到了一个新的Array作为结果：
> map()是数组的方法，可以传入函数变量，产生新的数组。
'use strict';

function pow(x) {
    return x * x;
}
var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
var results = arr.map(pow); // [1, 4, 9, 16, 25, 36, 49, 64, 81]
console.log(results);

map()作为高阶函数，事实上它把运算规则抽象了，因此，我们不但可以计算简单的f(x)=x2，还可以计算任意复杂的函数，比如，把Array的所有数字转为字符串：

var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
arr.map(String); // ['1', '2', '3', '4', '5', '6', '7', '8', '9']
只需要一行代码。





### reduce
再看reduce的用法。Array的reduce()把一个函数作用在这个Array的[x1, x2, x3...]上，这个函数必须接收两个参数，reduce()把结果继续和序列的下一个元素做累积计算，其效果就是：

[x1, x2, x3, x4].reduce(f) = f(f(f(x1, x2), x3), x4)
比方说对一个Array求和，就可以用reduce实现：

var arr = [1, 3, 5, 7, 9];
arr.reduce(function (x, y) {
    return x + y;
}); // 25



## filter
filter也是一个常用的操作，它用于把Array的某些元素过滤掉，然后返回剩下的元素。

和map()类似，Array的filter()也接收一个函数。和map()不同的是，filter()把传入的函数依次作用于每个元素，然后根据返回值是true还是false决定保留还是丢弃该元素。
var arr = [1, 2, 4, 5, 6, 9, 10, 15];
var r = arr.filter(function (x) {
    return x % 2 !== 0;
});
r; // [1, 5, 9, 15]



### 回调函数
filter()接收的回调函数，其实可以有多个参数。通常我们仅使用第一个参数，表示Array的某个元素。回调函数还可以接收另外两个参数，表示元素的位置和数组本身：

var arr = ['A', 'B', 'C'];
var r = arr.filter(function (element, index, self) {
    console.log(element); // 依次打印'A', 'B', 'C'
    console.log(index); // 依次打印0, 1, 2
    console.log(self); // self就是变量arr
    return true;
});

网上找到的一个例子：
> 你有事去隔壁寝室找同学，发现人不在，你怎么办呢？
> 方法1，每隔几分钟再去趟隔壁寝室，看人在不
> 方法2，拜托与他同寝室的人，看到他回来时叫一下你

> 前者是轮询，后者是回调。

> 那你说，我直接在隔壁寝室等到同学回来可以吗？

> 可以啊，只不过这样原本你可以省下时间做其他事，现在必须浪费在等待上了。把原来的非阻塞的异步调用变成了阻塞的同步调用。

从这个例子上，回调函数是利用别人的结果做出判断。




## 闭包
### 函数作为返回值
高阶函数除了可以接受函数作为参数外，还可以把函数作为结果值返回。

我们来实现一个对Array的求和。通常情况下，求和的函数是这样定义的：

function sum(arr) {
    return arr.reduce(function (x, y) {
        return x + y;
    });
}

sum([1, 2, 3, 4, 5]); // 15
但是，如果不需要立刻求和，而是在后面的代码中，根据需要再计算怎么办？可以不返回求和的结果，而是返回求和的函数！

function lazy_sum(arr) {
    var sum = function () {
        return arr.reduce(function (x, y) {
            return x + y;
        });
    }
    return sum;
}
当我们调用lazy_sum()时，返回的并不是求和结果，而是求和函数：

var f = lazy_sum([1, 2, 3, 4, 5]); // function sum()
调用函数f时，才真正计算求和的结果：

f(); // 15
在这个例子中，我们在函数lazy_sum中又定义了函数sum，并且，内部函数sum可以引用外部函数lazy_sum的参数和局部变量，当lazy_sum返回函数sum时，相关参数和变量都保存在返回的函数中，这种称为“闭包（Closure）”的程序结构拥有极大的威力。

请再注意一点，当我们调用lazy_sum()时，每次调用都会返回一个新的函数，即使传入相同的参数：

var f1 = lazy_sum([1, 2, 3, 4, 5]);
var f2 = lazy_sum([1, 2, 3, 4, 5]);
f1 === f2; // false
f1()和f2()的调用结果互不影响。

> 这里谈一谈我自己的看法。前面中的lazy_sum(arr)函数并没有直接返回函数结果，返回的是函数sum，所以执行时需要调用f()，而不是直接使用f，因为返回的是函数，相当于没有执行完成。使用f()才是完成了函数的调用，计算出了结果。其中的闭包函数那一段只是存放了数值，并没有计算真正的结果。我们使用一个变量来监测一下。
```
var a = 1;
function lazy_sum(arr) {
    var sum = function () {
        return arr.reduce(function (x, y) {
			  a +=1;
            return x + y;
        });
    }
    return sum;
}

var f = lazy_sum([1,2,3,4])
(1)a =1
(2)f
(3)a =1
(4)f()
(5)a=4
```
> 这里后面采用的是（1）标记方法标记步骤。第一步开始a等于1，说明没有执行内部的匿名函数。（2）步执行f，并没有反应，他只是指向这个函数。（3）步，a=1从结果看，也说明没有执行函数。（4）步，执行f()；（5）a=4说明执行了。此时说明f（）调用了匿名函数。
> 如果想要返回函数，直接在最后return sum();这样返回的是函数结果。
> 依然可以测试。
```
var a = 1;
function lazy_sum(arr) {
    var sum = function () {
        return arr.reduce(function (x, y) {
			  a +=1;
            return x + y;
        });
    }
    return sum();
}

(1)a =1
(2)var f = lazy_sum([1,2,3,4])
(3)a =4
(4)f
(5)a=4
```
> 调用函数结束后，因为结果已经返回，所以直接计算结果。此时a已经等于4，改变结果。






## 闭包
注意到返回的函数在其定义内部引用了局部变量arr，所以，当一个函数返回了一个函数后，其内部的局部变量还被新函数引用，所以，闭包用起来简单，实现起来可不容易。

另一个需要注意的问题是，返回的函数并没有立刻执行，而是直到调用了f()才执行。我们来看一个例子：

function count() {
    var arr = [];
    for (var i=1; i<=3; i++) {
        arr.push(function () {
            return i * i;
        });
    }
    return arr;
}

var results = count();
var f1 = results[0];
var f2 = results[1];
var f3 = results[2];
这里arr.push(function(){})中其实是push了一个函数，并没有push其他东西。所以arr是一个函数组，有三个元素，元素都是函数。
所以这里的f1、f2、f3都是函数。直到最后执行才可以计算得出结果。
理论上讲，创建一个匿名函数并立刻执行可以这么写：
解决方法：函数立即执行。
function (x) { return x * x } (3);
但是由于JavaScript语法解析的问题，会报SyntaxError错误，因此需要用括号把整个函数定义括起来：

(function (x) { return x * x }) (3);

function count() {
    var arr = [];
    for (var i=1; i<=3; i++) {
        arr.push((function (n) {
            return function () {
                return n * n;
            }
        })(i));
    }
    return arr;
}
这是上面方式立即执行的方法。这里面arr.push的是结果。代码如下：
(function (n) {
            return function () {
                return n * n;
            }
        })(i)
将匿名函数使用（）包裹，从上面的立即执行的方式可以看出来，想要立即执行需要将函数体包裹起来，作为一个整体，后面的接一个（）表示参数。
[知乎解析]([如何才能通俗易懂的解释javascript里面的‘闭包’？ - 知乎](https://www.zhihu.com/question/34547104/answer/198016466))
> 所有的函数在运行时都会在其所在空间创建一个新的子平行空间，所有的参数，局部变量及在函数内创建的函数（不管是函数声明还是赋值给变量的函数表达式），都是在这个子平行空间内创建（出生）的，只有出生在这个平行空间内的代码可以访问到这个平行空间内的变量及函数。即使在这个空间内创建的函数被返回，并赋值给全局或者父平行空间的变量，它也还是出生在那个平行空间，它对变量的访问也还是从那个空间开始往更大的平行空间查找变量，直到全局变量。
> * 由于平行空间内的函数在运行时又会在其所在空间创建更小的平行空间。所以如果在平行空间内创建的函数还有可能运行，则函数所在的平行空间及所有父空间都不会被销毁。* 
当然不是！闭包有非常强大的功能。举个栗子：

在面向对象的程序设计语言里，比如Java和C++，要在对象内部封装一个私有变量，可以用private修饰一个成员变量。

在没有class机制，只有函数的语言里，*借助闭包，同样可以封装一个私有变量*。我们用JavaScript创建一个计数器：

'use strict';

function create_counter(initial) {
    var x = initial || 0;
    return {
        inc: function () {
            x += 1;
            return x;
        }
    }
}
这里return的是一个对象。里面包含了inc方法。
代码可以改写成：
function create_counter(initial) {
    var x = initial || 0;
    return function () {
            x += 1;
            return x;
    }
}





## 箭头函数
var fn = x => x * x;
箭头函数相当于匿名函数，并且简化了函数定义。箭头函数有两种格式，一种像上面的，只包含一个表达式，连{ ... }和return都省略掉了。还有一种可以包含多条语句，这时候就不能省略{ ... }和return：

x => {
    if (x > 0) {
        return x * x;
    }
    else {
        return - x * x;
    }
}
如果参数不是一个，就需要用括号()括起来：

// 两个参数:
(x, y) => x * x + y * y

// 无参数:
() => 3.14

// 可变参数:
(x, y, ...rest) => {
    var i, sum = x + y;
    for (i=0; i<rest.length; i++) {
        sum += rest[i];
    }
    return sum;
}



## this
箭头函数看上去是匿名函数的一种简写，但实际上，箭头函数和匿名函数有个明显的区别：箭头函数内部的this是词法作用域，由上下文确定。

回顾前面的例子，由于JavaScript函数对this绑定的错误处理，下面的例子无法得到预期结果：

var obj = {
    birth: 1990,
    getAge: function () {
        var b = this.birth; // 1990
        var fn = function () {
            return new Date().getFullYear() - this.birth; // this指向window或undefined
        };
        return fn();
    }
};
现在，箭头函数完全修复了this的指向，this总是指向词法作用域，也就是外层调用者obj：

var obj = {
    birth: 1990,
    getAge: function () {
        var b = this.birth; // 1990
        var fn = () => new Date().getFullYear() - this.birth; // this指向obj对象
        return fn();
    }
};
obj.getAge(); // 25





## generator
generator跟函数很像，定义如下：

function* foo(x) {
    yield x + 1;
    yield x + 2;
    return x + 3;
}
generator和函数不同的是，generator由function*定义（注意多出的*号），并且，除了return语句，还可以用yield返回多次。
函数只能返回一次，所以必须返回一个Array。但是，如果换成generator，就可以一次返回一个数，不断返回多次。用generator改写如下：

function* fib(max) {
    var
        t,
        a = 0,
        b = 1,
        n = 0;
    while (n < max) {
        yield a;
        [a, b] = [b, a + b];
        n ++;
    }
    return;
}
直接调用试试：

fib(5); // fib {[[GeneratorStatus]]: "suspended", [[GeneratorReceiver]]: Window}
直接调用一个generator和调用函数不一样，fib(5)仅仅是创建了一个generator对象，还没有去执行它。

调用generator对象有两个方法，一是不断地调用generator对象的next()方法：

var f = fib(5);
f.next(); // {value: 0, done: false}
f.next(); // {value: 1, done: false}
f.next(); // {value: 1, done: false}
f.next(); // {value: 2, done: false}
f.next(); // {value: 3, done: false}
f.next(); // {value: undefined, done: true}
next()方法会执行generator的代码，然后，每次遇到yield x;就返回一个对象{value: x, done: true/false}，然后“暂停”。返回的value就是yield的返回值，done表示这个generator是否已经执行结束了。如果done为true，则value就是return的返回值。

第二个方法是直接用for ... of循环迭代generator对象，这种方式不需要我们自己判断done
generator和普通函数相比，有什么用？

因为generator可以在执行过程中多次返回，所以它看上去就像一个可以记住执行状态的函数，利用这一点，写一个generator就可以实现需要用面向对象才能实现的功能。
generator还有另一个巨大的好处，就是把异步回调代码变成“同步”代码。这个好处要等到后面学了AJAX以后才能体会到。
