### 1. 执行上下文

&nbsp;&nbsp;&nbsp;&nbsp;执行上下文就是当前 JavaScript 代码被解析和执行时所在环境的抽象概念， JavaScript 中运行任何的代码都是在执行上下文中运行。函数的每次运行都会生成，并且都是独一无二的，函数运行完就会销毁。

#### 1.1 执行上下文类型：

&nbsp;&nbsp;&nbsp;&nbsp;执行上下文总共有三种类型：

- `全局执行上下文`： 这是默认的、最基础的执行上下文。不在任何函数中的代码都位于全局执行上下文中。
- `函数执行上下文`：每次调用函数时，都会为该函数创建一个新的执行上下文。一个程序中可以存在任意数量的函数执行上下文。每当一个新的执行上下文被创建，它都会按照特定的顺序执行一系列步骤，具体过程将在本文后面讨论。
- `Eval执行上下文`： 运行在 `eval` 函数中的代码也获得了自己的执行上下文，但由于 Javascript 开发人员不常用 eval 函数，所以在这里不再讨论。

#### 1.2 执行栈

&nbsp;&nbsp;&nbsp;&nbsp;在具体谈执行上下文时，我们先来简单了解下执行栈，`执行栈`在其他编程语言中也被叫做`调用栈`，具有 `LIFO`（`后进先出`）结构，用于存储在代码执行期间创建的所有`执行上下文`。
&nbsp;&nbsp;&nbsp;&nbsp;当 `JavaScript` 引擎首次读取你的脚本时，它会创建一个`全局执行上下文`并将其推入当前的`执行栈`。每当发生一个函数调用，引擎都会为该函数创建一个`新的执行上下文`并将其推到当前执行栈的`顶端`。引擎会运行执行上下文在执行栈顶端的函数，当此函数运行完成后，其对应的执行上下文将会从执行栈中弹出，栈顶指针下移，上下文控制权将移到当前执行栈的下一个执行上下文。

&nbsp;&nbsp;&nbsp;&nbsp;让我们通过下面的代码示例来理解这一点：

```
function fn1() {
  fn2();
  console.log('fn1 函数执行上下文');
}

function fn2() {
  console.log('fn2 函数执行上下文');
}

fn1();
console.log('Global 全局执行上下文');
```

![](https://user-gold-cdn.xitu.io/2020/7/10/1733688843530eed?w=1172&h=646&f=png&s=55207)
&nbsp;&nbsp;&nbsp;&nbsp;当上述代码在浏览器中加载时，JavaScript 引擎会创建一个`全局执行上下文`并且将它推入当前的`执行栈`。当调用 `fn1`函数时，`JavaScript` 引擎为该函数创建了一个`新的执行上下文`并将其推到`当前执行栈的顶端`。
当在 `fn1` 函数中调用 `fn2` 函数时，Javascript 引擎为该函数创建了一个`新的执行上下文`并将其推到`当前执行栈的顶端`。当 `fn2` 函数执行完成后，它的执行上下文从当前执行栈中弹出，上下文控制权将移到当前执行栈的下一个执行上下文，即 `fn1` 函数的执行上下文。
当 `fn1` 函数执行完成后，它的执行上下文从当前执行栈中弹出，上下文控制权将移到`全局执行上下文`。一旦所有代码执行完毕，Javascript 引擎把全局执行上下文从执行栈中移除。

#### 1.3 全局执行上下文

&nbsp;&nbsp;&nbsp;&nbsp;我们已经看到了 JavaScript 引擎如何管理执行上下文，现在就让我们来理解 JavaScript 引擎是如何`创建` `执行`上下文的。

&nbsp;&nbsp;&nbsp;&nbsp;`全局执行上下文`（`Global Execution Context`）包含至少两个东西——一个`全局对象`和 `this` 变量。`this` 会引用全局对象，在浏览器执行 JavaScript 则全局对象是 `window`，在 Node 环境执行则全局对象是 `global`。看下面代码

```
console.log('name: ', name) // name: undefined
console.log('handle: ', handle) // handle: undefined
console.log('getUser :', getUser) // getUser: ƒ getUser () {}

var name = 'Tyler'
var handle = '@tylermcginnis'

function getUser () {
  return {
    name: name,
    handle: handle
  }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;每个执行上下文都有两个独立的阶段——创建（Creation）阶段和执行（Execution）阶段，每个阶段都有它特有的职责，可以看到当代码还没执行到定义变量和函数位置的时候，变量和函数已经被声明了，只是变量的初始值为`undefined`，而函数在未执行的时候已经在内存中开辟了空间并将引用给了函数的声明。

执行上下文分两个阶段：`创建阶段`和`执行阶段`

##### 1.3.1 创建阶段

- 1 创建作用域链
- 1 创建全局对象（`window` or `global`）
- 2 创建 this 对象（`window` or `global`）
- 3 创建作用域链
- 4 在内存中放置所有函数声明，变量声明并默认赋值为 `undefined`。（变量提升，块级作用域(`const, let`)不会发生变量提升，而是`绑定`在`暂时性死区`）

> `暂时性死区`的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。看下面例子。<br>

```
var name = 'Tyler'
var handle = '@tylermcginnis'

function getUser () {
  console.log(name)
  let name = "Tyler"
  return {
    name: name,
    handle: handle
  }
}
getUser() // ReferenceError: Cannot access 'name' before initialization
```

##### 1.3.2 执行阶段

```
var name = 'Tyler'
var handle = '@tylermcginnis'

function getUser () {
  return {
    name: name,
    handle: handle
  }
}
console.log('name: ', name) // name: Tyler
console.log('handle: ', handle) // handle: @tylermcginnis
console.log('getUser :', getUser) // getUser: ƒ getUser () {}
```

&nbsp;&nbsp;&nbsp;&nbsp;JavaScript 引擎才会开始一行一行的执行代码，并为执行到的变量赋值

#### 1.4 函数执行上下文

&nbsp;&nbsp;&nbsp;&nbsp;`函数执行上下文`与`全局执行上下文`几乎完全相同，在函数调用的时候创建，不同点是他不会创建全局对象，而是会创建一个参数对象（`arguments`），就不做过多介绍了。

### 2 作用域链

#### 2.1 词法作用域

&nbsp;&nbsp;&nbsp;&nbsp;JavaScript 中的变量都是有词法作用域的。词法作用域是由你在写代码时将变量和块级作用域写在哪里所决定的，因此当词法分析器处理代码时会保持作用于不变（欺骗词法除外），不会因为函数调用的位置发生改变。

```
function foo() {
    console.log(a); // 1
}
function bar() {
    var a = 2;
    foo();
}
var a = 1;
bar()
```

#### 2.2 作用域链创建和标识符访问过程

&nbsp;&nbsp;&nbsp;&nbsp;每一个函数都被表示为对象，进一步说，他是一个函数实例。函数对象正如其他对象那样，拥有你可以访问的属性，也包含了不能被访问的属性，比如内部属性[[Scope]]。[[Scope]]包含了一个函数被创建的作用域中对象的集合。这个集合被称为函数作用域链，它决定了哪些数据可以被访问。此函数作用域链中的每个对象被称为一个可变对象，以键值对的形式存在。当一个函数创建后，它的作用域链被可变对象填充，这些对象代表创建此函数的环境中可访问的数据。看下面的全局函数

```
function add(sum1, sum2) {
    var sum = sum1 + sum2;
    return sum;
}
```

&nbsp;&nbsp;&nbsp;&nbsp;当 add 函数被创建后，它的作用域被填入了一个单独的可变对象，此全局对象代表了全局范围内定义的可变对象（包含 this，window，document，...）

![](https://user-gold-cdn.xitu.io/2020/7/10/1733788c1d5d52ff?w=1552&h=584&f=png&s=82607)

&nbsp;&nbsp;&nbsp;&nbsp;add 函数的作用域链将会在运行时用到。

```
var total = add(1, 2)
```

&nbsp;&nbsp;&nbsp;&nbsp;运行 add 函数时建立一个内部对象，称为上文提到的执行上下文。该执行上下文定义了函数运行是的环境，并且每次调用 add 函数都会创建一个独一无二的执行上下文，当函数执行完毕，执行上下文被销毁。
一个执行上下文有自己的作用域链，用于字符串解析，当执行上下文被创建时，他的作用域链被初始化，连同运行函数的[[Scope]]属性中所包含的对象，这些值按照他们在函数中出现的顺序，被复制到执行上下文的作用域链中，这项工作一旦完成，一个被称作“激活对象”的新对象就位执行上下文创建好了，此激活对象最为函数执行时的一个可变对象，包含访问所有局部变量，命名参数，参数集合（arguments），和 this 的接口，然后被推到作用域顶端。当作用域链被销毁时，激活对象也一起销毁，如下图：

![](https://user-gold-cdn.xitu.io/2020/7/10/17337987bb943f87?w=1554&h=900&f=png&s=138752)
&nbsp;&nbsp;&nbsp;&nbsp;在函数运行过程中，每遇到一个变量，标识符识别过程要决定从哪里获取或存储数据。这个搜索执行上下文的作用域链，查找同名的标识符。搜索宫锁从运行函数的激活对象即作用域链的顶端开始。如果找到了就使用这个具有指定标识符的变量；否则搜索工作将进入作用域链的下个对象。一直到标识符被找到为止，若完成了搜索工作还没找到，则认为该标识符是未定义的。函数运行时每个标识符都要经过这样的搜索过程，正是这样的搜索过程影响了代码运行的性能，标识符子啊作用域链中的深度越深则越耗性能（优化的 JavaScript 引擎除外）。

> 优化过的 JavaScript 引擎，入 Safari 的 Nitro 引擎，企图分析代码来确定哪些变量应该在任意时刻被访问，来加快标识符的识别过程。这些引擎企图避开传统作用域链查找，取代以标识符索引的方式进行快速查找。但是当涉及欺骗词法作用域后，此优化就不起作用了。引擎需要切回慢速的基于哈希表的标识符识别方法，更像传统的作用域链搜索。

#### 2.3 欺骗词法之 with

&nbsp;&nbsp;&nbsp;&nbsp;一般来说执行上下文的作用域链是不会被改变的，但是当使用`eval`、try-catch 的子句`catch`和`with`，下面我们来看下 with

```
function withFunction() {
  var b = "b"
  with (document) {
    getElementById("id").onclick = function() {
      console.log("do something", b)
    }
  }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;当代码执行到 with 表达式的时候，执行上下文的作用域链被临时改变了，一个新的可变对象被创建，它包含指定对象的所有属性，并插入到作用域链的顶端，这意味着执行函数的所有局部变量都被推入到第二作用域链对象中，所以访问代价更高了。

![](https://user-gold-cdn.xitu.io/2020/7/10/17337bb5f908ffab?w=1526&h=994&f=png&s=170639)

### 闭包

&nbsp;&nbsp;&nbsp;&nbsp;函数和对其周围状态（lexical environment，词法环境）的引用捆绑在一起构成闭包（closure）。也就是说，闭包可以让你从内部函数访问外部函数作用域。[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)

下面看一个经典案例：

```
function foo() {
    var a = 2;
    function bar() {
        console.log(a);
    }
    return bar
}
foo()() // 2
```

&nbsp;&nbsp;&nbsp;&nbsp;函数 bar()的词法作用域能够访问 foo()内部作用域，然后将 bar()函数本身当做一个值类型传递，foo()执行后，通常期望 foo()的整个内部作用域被销毁，单闭包的神奇之处可以阻止这件事情发生，事实上内部作用域依然存在，是 bar()本身在使用。
