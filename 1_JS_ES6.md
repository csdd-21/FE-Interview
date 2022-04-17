# **重要原理**

## **原型对象、原型链**

- **原型对象**
  
  - 原型对象即`prototype`，函数才有原型对象，实例不会有原型对象，原型对象`prototype`可以理解为存放所有实例公用的属性或方法
  
- **原型链**

  - JavaScript中每个对象都有私有属性`__proto__`指向它的构造函数的原型对象`prototype`，该原型对象也有自己的原型对象(也是通过`__proto__`访问)，层层向上直到`null`，`null`没有原型并作为这个原型链中的最后一个环节。 JavaScript中所有对象都是位于原型链顶端Object的实例

    `instance.__proto__ === instance.constructor.prototype`

    `Function.prototype.__proto__ === Object.prototype`

  - 对象中都有私有属性`__proto__`指向它的构造函数的原型对象`prototype`，函数对象除了有`__proto__`属性之外，还有`prototype`属性，当函数对象作为构造函数创建实例时，该`prototype`属性将被作为实例对象的`__proto__`指向的值
  - 当一个对象调用属性或方法但在自身不存在时，会通过私有属性`__proto__`指向原型对象`prototype`上去找，如果没找到，就去原型的原型找，依次类推，直到找到该属性或方法，找不到返回`undefined`，从而形成原型链

## **new**

- `new` 关键字会进行如下的操作：

  1. 创建一个空对象`{}`

  2. 为该对象添加`__proto__`属性并链接至它的构造函数的原型对象`prototype`

  3. 将`this`指向该新对象并执行构造函数

  4. 隐式`return this`（如果显式返回非基本数据类型会覆盖掉this）

     注：`new、Object.create`区别：`new`生成的对象会自动进行原型链链接，`Object.create`接收第一个参数作为新对象`__proto__`指向的原型对象

## **继承**

1. **原型链继承 — 重写prototype**

   - 实现：
     - 重写Son.prototype为一个Father实例
       `Son.prototype = new Father() ` 
       `Son.prototype.constructor = Son`

   - 特性：
     - Son原型链上的属性值修改都会在Son实例之间互相影响（引用值）
     - Son的prototype被重写，因此还需要重新为Son指定构造函数constructor
     - 在Son.prototype重写之前的原型链上的属性和方法都会丢失

   **原型链继承 — Object.setPrototypeOf()**

   - 实现：
     - 使用es6的Object.setPrototypeOf()来为Son设置原型对象

       `Object.setPrototypeOf(Son.prototype,new Father())`
     
   - 特性：
     - Son原型链上的属性值修改都会在Son实例之间互相影响（引用值）
     - 与之前重写Son.prototype不同，Son原本原型链上的constructor、属性、方法都不会丢失

2. **构造函数继承**

   - 实现：
     - 在Son的构造函数内部通过call、apply来执行Father的构造函数，这样后Father实例属性就变成了Son实例自身的属性，但是只拿到了Father的实例属性，拿不到Father原型链
       `Father.call(this,arg1,arg2,..)`或`Father.apply(this,[argsArray])`

   - 特性：
     - 解决了原型链继承的"Son原型链上的属性值修改都会在Son实例之间互相影响"问题
     - 拿不到Father原型上的属性或方法

3. **组合继承**

   - 实现：

     - 原型链+构造函数的组合继承

       `Son.prototype = new Father() `

       `Son.prototype.constructor = Son`
       
       `Father.call(this,arg1,arg2,..)`或`Father.apply(this,[argsArray])`

   - 特性：

     - Son的实例和原型链上都有Father的属性

4. **寄生组合继承 — 重写prototype**

   - 实现：

     - 原型链+构造函数的组合继承，基本思路是不通过调用父类构造函数给子类原型赋值，而是取得父类原型的一个副本

       `Father.call(this)`

       `Son.prototype = Object.create(Father.prototype) ` 

       `Son.prototype.constructor = Son` 

   - 特性：

     - 直接拿Father的原型对象，而不是Father实例
     - Son原本原型链上的constructor、属性、方法都丢失

   ```js
   // 来自红宝书 - Object.create这个api的本质就是下面这个函数：
   function object(o) {
       function F() { }
       F.prototype = o;
       return new F();
   }
   ```

   **寄生组合继承 — Object.setPrototypeOf()**

   - 实现：

     - 原型链+构造函数的组合继承

     ​       `Father.call(this) `

     ​       `Object.setPrototypeOf(Son.prototype, Father.prototype)`

   - 特性：

     - 直接拿Father的原型对象，而不是Father实例
     - Son原本原型链上的constructor、属性、方法都不会丢失

5. **类继承**

   - 实现：

     - 寄生组合继承的语法糖写法，使用extends关键字，且在Son类的构造函数里调用super()

       `class Son extends Father`

       `super()`

   - 特性：

     - extends相当于原型链连接
     - super相当于通过call、apply执行父类构造函数，super()必须在使用this前调用

## **class**

- **class**
  - `class`包含构造函数方法、实例方法、获取函数getter、设置函数setter、静态类方法（但不是必需的）
  - `class`可以看作为语法糖/构造函数的另一种写法，`class`不可变量提升（构造函数可）
  - `class`里定义的属性为实例上的属性，`class`里定义的方法为原型对象`prototype`的方法，要在类的原型对象上添加属性只能通过`类名.prototype.xx=''`定义
- **constructor**
  - 实例属性除了可定义在`constructor()`里的`this`上，也可以定义在类的最顶层（且不需要写this），但这只适用实例赋值不需要传参的情况
  - 一个类必须有`constructor()`方法，如果没有显式定义，一个空的`constructor()`方法会被默认添加。通过`new`命令生成对象实例时，自动调用该方法
  - 继承时子类必须在`constructor`方法中调用`super`方法，否则新建实例时会报错
- **static**
  - 静态属性/方法：通过`static`关键字定义，且只能通过`类名.xx`调用，实例无法调用
  - 静态方法里面的`this`指向这个类而不是类的实例，静态方法里面的`super`指向父类

## **this**

- **定义：**
  
  - `this`总是指向函数直接调用者（只有在调用的时候才能确认`this`），而箭头函数`this`绑定父作用域的上下文
  - 如果有`new`关键字，`this`指向`new`出来的实例
  - 可通过`call、apply、bind`改变`this`指向（无法改变箭头函数的`this`绑定指向，只能改变它的父函数的`this`）（实用场景：导入外部的js文件时进行bind，这样才能调得到vue实例上的data、computed方法等）
  
- **优先级：new绑定 > 显示绑定 > 隐式绑定 > 默认绑定**

  - new绑定：`this`指向`new`出来的实例

  - 显式绑定：`call、apply、bind`

  - 隐式绑定：作为对象的方法调用时，`this`指向该对象 ( 作为事件监听器`addEventListener`时,  `this`指向触发事件的元素本身（第二个参数回调函数不能写成箭头函数的形式）)

  - 默认绑定:   作为函数独立调用，严格模式下`this`指向`undefined`，非严格模式下指向`window`


- **隐式丢失**：

  函数丢失隐式绑定的对象，从而应用默认绑定（window或undefined）


- **call、apply、bind区别：**

  - `call：B.call(A, args1,args2,..)`

  - `apply：B.apply(A, argsArray)` 

  - `bind：B.bind(A, args1,args2,..)`

    注：call、apply的唯一区别只有传参方法，call接受多个参数，apply只接受一个参数数组。call、apply都是改变this指向且立即执行函数本身，而bind是返回原函数的拷贝、且拥有指定的this和初始参数

## **箭头函数、构造函数**

- **箭头函数：**
  
  - 箭头函数的`this`绑定了父作用域的上下文，箭头函数没有自己的`this`，通过`call()、apply()、bind()`方法调用时，会忽略掉第一个参数，因此只能传递参数，不能改变`this`指向
  
  - 不可以当作构造函数、没有`new、prototype、arguments`，但有`__proto__`、剩余参数、默认参数
  
  - 没有自己的`arguments`对象，但可访问父上下文的`arguments`
  
    注：vue2中帮我们把methods里面的this绑定到实例上（源码：遍历所有methods并分别调用bind），所以vue2不允许用箭头函数定义methods
  
- **构造函数：**

  - 构造函数一般用大写，普通函数也可以使用`new`关键字来当做构造函数使用
  - 构造函数使用`new`关键字调用时会进行原型链链接和将`this`指向`new`出来的实例，而普通函数调用时`this`指向调用者，如果没有调用者那就默认指向`window`
  
  - 构造函数默认返回新对象，而普通函数如何没有显示return出来就默认`return undefined`


## **闭包、垃圾回收机制、内存泄漏/溢出**

- **闭包：**

  - **定义：**

    - 一个函数和对其父作用域们局部变量的引用的组合叫做闭包，且注意闭包取的是父作用域中对应变量最终的值（不止是引用与其相邻的父函数里的变量才算，父的父的父都算，全局也算）

    - 闭包就是当一个函数即使是在它的词法作用域之外被调用时，也可以记住并访问它的词法作用域。一般函数的词法环境在函数调用结束后就被销毁，但是闭包会保存对创建时所在词法环境的引用（销毁前在堆里存放一个该变量），即便创建时所在的执行上下文被销毁，但创建时所在词法环境依然存在，以达到延长变量的生命周期的目的）

  - **好处：**
    - 闭包中的变量可被缓存、不被垃圾回收机制清除，也不会与其它环境变量发生命名冲突
    
  - **坏处：**
    - 滥用闭包会增大内存消耗甚至内存泄漏，**解决方法**：使用完后及时将其赋值为null来释放内存
    
  - **使用场景：**
  - 防抖、节流、函数柯里化、vuex中通过方法访问getters、setTimeout引用外部变量
  
- **垃圾回收机制**：根据不同浏览器，垃圾回收机制有两种：引用计数（chrome）、标记清除

  - 引用次数：引用+1、不再引用-1、值为0时 => 清除

  - 标记清除：标记符、进入环境、离开环境 => 清除

    注: `weakMap、weakSet`是弱引用，垃圾回收机制不需要考虑对它们的回收

- **内存泄漏和内存溢出的区别：**

  - 内存泄漏：占用的内存没有及时释放，内存泄漏多了就容易导致内存溢出（大对象）
  - 内存溢出：函数运行时需要的内存超出了计算机为其分配的内存，常见如死循环（循环递归调用但是没有判断让其return出去的语句）（尾递归函数->尾递归优化）

## **Promise、async await、实现异步**

- **Promise**

  - **定义：**

    - `Promise`构造函数，可以用`new Promise()`实例化一个promise对象用来处理异步操作，`Promise`接受一个函数参数，该函数参数里又有两个参数分别是`resolve`和`reject`
      - `resolve`函数作用是将promise对象的状态从`pending`变为`fulfilled`，以及保存`resolve('xx')`里的内容作为返回结果，结果可在`.then()`中获取
      - `reject`函数作用是将promise对象的状态从`pending`变为`rejected`，以及保存`reject('xx')`里的内容作为错误信息返回，抛出的错误可在`.then()`的第二个函数参数里或在`.catch()`获取

    - promise对象用于表示一个异步操作的最终状态(完成/失败)及其结果值

  - **特性：**

    - 当处于`pending`状态时，无法得知目前进展到哪一个阶段
    - promise只会执行一个`resolve`或者`reject`，其他的忽略掉
    - 如果不设置错误处理，promise内部抛出的错误，不会反应到外部（不会阻塞运行,但会报错uncought in promise）
    - 对于`Promise.race()`..等一些方法而言，promsie的结果可能会提前返回，但即使已经返回，如果还有未执行完的promise任务的话，仍然继续执行，这也是promise的缺点，无法取消`Promise`，一旦创建就会执行，无法中途取消

  - **状态**：（共3种）

    - `pending:` 初始状态，非`fulfilled`或`rejected`

    - `fulfilled:` 操作成功完成

    - `rejected:` 操作失败

    注：fulfilled与 rejected一起合称settled

  - **抛出错误方式：**

    - `return Promise.reject('xx')`
    - `throw new Error('xx')`
  
  - **链式调用：**
  
    - `then()、catch()、 finally()`都会返回一个新的promise对象，这也是能够链式调用原因
    - 在`.then()`里若返回任意一个非promise值，都会被包装为一个promise对象，即`return xx => Promise.resolve(xx)`
    - `.catch()`，无论链式调用多少层级，`.catch()`都能捕获到上层错误（错误有类似“冒泡”的性质，直到被捕获为止）
  
  - **实例方法**：
  
    - `.then()`，接收两个函数参数，第一个函数用来处理`resolve`返回的结果，第二个函数用来处理`reject`抛出的错误
    - `.catch()`，捕获`reject`抛出的错误，可以看作是.then()的第二个函数参数的语法糖（若错误在`.then()`的第二个函数参数里被捕获，那这里就不会再被捕获）
    - `.finally()`，无论`fulfilled`或`rejected`都会被调用，不接收参数
  
  - **构造函数方法**：
  
    - `.resolve()`  //Promise.resolve()相当于new Promise( ( resolve,reject)=>{ resolve('xx') } )的语法糖
  
    - `.reject()`    //Promise.reject()也是个语法糖，同上
  
    - `.all() `     //所有都成功或任意一个失败返回
  
    - `.allSettled()`   //所有的promise有结果后才返回
  
    - `.any()`    //一个成功或全部失败返回
  
    - `.race() `   //任意一个成功或失败就返回
  
      注：promise的结果即使已经返回，但如果还有未执行完的promise任务的话，仍然会执行只是结果被忽略，这也是promise的缺点


- **async await**


    - **定义：**


      - async，用来声明一个异步方法，返回一定是一个promise对象。await关键字只在async函数内有效。如果你在async函数体之外使用它，就会抛出语法错误SyntaxError
    
      - await，等待promise异步方法执行并返回结果（并不是把异步变为同步），await后面的代码为微任务，相当于promsie.then()（只有async里面的、await后面的代码会被阻塞，async外的代码是不会被阻塞的）。promise.then解决了回调地狱问题，async await则解决了.then链式调用过长问题
    
        （当异步结果为失败，需要用try..catch处理异常）


  - **与Generator比较：**
    - async await其实就是Generator函数语法糖。async函数就是将Generator函数的星号`*`替换成async，将yield替换成await，仅此而已（来自ryf一书）。但又有所不同：
      - 更好的语义：比起星号和yield，语义更清楚了。async表示函数里有异步操作，await表示等待紧跟在后面的表达式结果
      - 适用性更广：yield后面只能是promise或可迭代对象，跟基本数据类型时则会报错。而await后面不仅可以跟着promise对象、也可以是原始值。await后面紧跟原始值时不会报错、而是会自动将它转为resolved包裹的promise对象


- **实现异步方法**
  - promise、async、setTimeout、setInterval、生成器函数

##**迭代器对象、生成器函数**

- **迭代器：**

  - **特点：**
    - 迭代器是一个对象，迭代器是通过重复调用`next()`方法按顺序返回可迭代对象中的元素，返回值为具有`value、done`属性的一个对象，其中value为当前迭代的返回值，done为是否已经迭代完的布尔值，未终止迭代done为false，终止迭代后done为true

- **生成器：**

  - **特点：**

    - 生成器函数使用`function*`编写，生成器函数提供了一个强大的选择：它允许你定义一个包含自有迭代算法的函数， 同时它可以自动维护自己的状态

    - 首次调用时并不会马上执行它里面的语句，而是返回一个生成器（Generator）的迭代器对象（Iterator）且生成器状态为`<suspended>`。当这个迭代器的`next() `方法第一次被调用时，其内的语句会执行到第一个`yield`位置为止，并返回`yield`后紧跟的值。生成器函数在执行时能暂停，后面又能从暂停处继续执行。如果遇到`yield*`，则表示将执行权移交给下一个生成器函数或可迭代对象（即当前生成器暂停执行，后面的如果还有yield语句也不生效了）

      注：yield*1 => 会报错，因为1是非迭代对象，无法移交执行权

    - `next()`方法返回一个对象，这个对象包含两个属性：`value、done`，value属性表示本次yield表达式的返回值，done属性为布尔类型，表示生成器后续是否还有yield语句，即生成器函数是否已经执行完毕并返回

  - 代码：

    ```js
    // 调用`next()`方法时，如果传入了参数，那么这个参数会传给上一条执行的yield的左边变量
    function* gen_fn(){
        yield 10;
        x=yield 'foo';
        yield x;
    }
    var gen = gen_fn(); //首次调用生成器函数，返回一个生成器的迭代器对象
    gen.next()     // {value:10, done:false}
    gen.next()     // {value:'foo', done:false}
    gen.next(100)  // {value:100, done:false},参数传给上一条yield语句的左边变量
    gen.next()     // {value:undefined, done:true}
    ```

    ```js
    function* gen_fn() {
      yield* [1,2];
      yield* "34";
      yield* arguments;
    }
    var iterator = gen_fn(5,6);
    iterator.next()  // { value: 1, done: false }
    iterator.next()  // { value: 2, done: false }
    iterator.next()  // { value: "3", done: false }，字符串也可迭代
    iterator.next()  // { value: "4", done: false }
    iterator.next()  // { value: 5, done: false }
    iterator.next()  // { value: 6, done: false }
    iterator.next()  // { value: undefined, done: true } 
    ```

## **可迭代对象、可枚举属性**

- **可迭代对象：**
  
  - **定义：**
    - 一种数据结构只要部署了`Iterator`接口(原型上有`Symbol.iterator`属性)，我们就称这种数据结构是“可迭代的”（iterable），对象中的成员便可由`for..of`按次序遍历，为可迭代对象的有：`Array、Map、Set、String、 Arguments、NodeList`对象（Object、WeakSet、WeakMap不是可迭代的）  
    - 对象`Object`之所以没有默认部署Iterator接口，是因为对象的哪个属性先遍历，哪个属性后遍历是不确定的，需要开发者手动指定（即对象里的键值对是无序的）
  - **`for..of`：**
    
    - 遍历可迭代对象，包括`Array、Map、Set、String、Arguments、NodeList对象`
    - 遍历计算生成的数据结构：`Array、Set、Map`调用`keys()、values()、entries()`后返回对应的遍历器对象`ArrayIterator、SetIterator、MapIterator`，这些返回的Iterator对象都可以用`for..of`遍历
    - `for..of`可用`break`、`continue`和`return`来跳出循环，`forEach`不可
    - object没有内置迭代器，因此无法使用`keys()、values()、entries()`这些方法，但可通过Object内置对象来使用，即`Object.keys()、Object.values()、Object.entries()`
    
  - 代码：
  
    ```js
    //for..of遍历可迭代对象
    //for..of遍历Map
    let map = new Map().set('a', 1)
    for(let value of map )         => ['a', 1]        
    for(let [value,key] of map)    =>   a , 1
    
    //for..of遍历对象
    for (let key of Object.keys(obj)) 
        
    //for..of遍历Array、Set、Map调用keys()、values()、entries()后都的返回值
    for (let item of [1,2,3].entries())   => [0,1]、[1, 2]、[2, 3]
    ```
  
- **可枚举属性：**
  - **定义：**
    - 如果使用字面量创建对象时，对象的属性默认是可枚举的，如果使用`Object.defineProperty()`新增对象属性时，属性默认是不可枚举的，需要设置`{enumerable:true}`才是可枚举属性
  - 重要的对象api比较：
    - `for..in`：遍历对象自身的、和原型链上的除Symbol外的可枚举属性
    - `Object.getOwnPropertySymbols()`：遍历对象自身的所有Symbol属性
    - `Object.keys()`：遍历自身的可枚举属性
    - `Object.getOwnPropertyNames()`：遍历对象所有的可枚举或不可枚举的属性
    - `Object.assign()`： 忽略`enumerable`为`false`的属性，只拷贝对象自身的可枚举的属性

## **同步/异步任务、宏/微任务、事件循环**

- **同步、异步任务：**（在JavaScript中，所有的任务都可以分为）
  - 同步任务：立即执行的任务，进入主执行栈中立即执行

  - 异步任务：异步执行的任务，进入任务队列等待执行，异步任务又分为宏任务、微任务（若是`setTimeout`，会等到计时结束后再加到宏任务队列里等待执行，若是`promise.then`会等到promise有结果后推到微任务等待执行）

- **宏任务、微任务：**
  - 宏任务：`script、setTimeout/setInterval、UI render、setImmediate、I/O（Node.js）`
  - 微任务：`Promise.then、process.nextTick（Node.js）、MutaionObserver、Object.observe（已废弃；Proxy对象替代）`

- **一次完整的事件循环Event loop：**
  - 执行script中的所有同步代码/同步任务，直接进入主执行栈中立即执行，若遇到微任务放到微任务队列，若遇到宏任务放到宏任务队列（若是`setTimeout`会等到计时结束在放到宏任务队列里）
  - 执行完所有同步任务后主执行栈为空，查询是否有可执行的微任务，若有则执行微任务
  - 若需要则渲染`UI`
  - 然后开始下一个宏任务`Event loop`，依次循环

## **函数式编程、响应式编程**

- **函数式编程：**

  - 每个函数只做一件事情、只接受一个参数、只返回一个结果，不会污染其它、不会有其它副作用
  - 好处是：逻辑清晰易于调试，坏处：复杂逻辑的话会导致函数嵌套过深反而不好了

  函数柯里化、promise都可看做是函数式编程的一种体现

  解决嵌套过深，可以用链式调用，需要函数内部都返回this

- **响应式编程：**待补充

## **es6、commonjs模块**

- **区别：**
  - **写法**：es6使用import/export，commonjs使用module.exports/require（也可使用exports，本质上是module.exports简写）
  - **依赖**：es6建立模块依赖关系时在编译阶段，而commonjs在运行阶段
  - **导入**：es6导入的是变量的只读映射，因此如果是对导入变量修改（且它是基本数据类型），就会报错TypeError: Assignment to constant variable.。而commonjs导入的是值拷贝
  - **分包**：webpack对使用import导入的模块可以分包，但不支持用require导入的模块进行分包

# **基本结构**

## **对象**

[`Object.is()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is)

方法判断两个值是否为同一个值。

如，`NaN === NaN //false     Object.is(NaN) === Object.is(NaN ) //true`

[`Object.assign()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)

通过复制一个或多个对象来创建一个新的对象。— 对象浅拷贝（忽略对象的不可枚举属性，只拷贝可枚举属性）

[`Object.create()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)

创建一个新对象，使用现有的对象来提供新创建的对象的`__proto__` 

如，`Object.create( {'a':11}, {'b':{value:22} } )`

[`Object.defineProperty()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

给对象添加/修改一个属性并指定该属性的配置。— 且可使用get/set控制对象的访问、vue2响应式原理

[`Object.getPrototypeOf()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/GetPrototypeOf)

返回指定对象的原型对象。— 查爹

[`Object.setPrototypeOf()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf)

设置对象的原型（即内部 `[[Prototype]]` / `__proto__`属性）。— 换爹

[`Object.entries()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/entries)

返回给定对象自身可枚举属性的 `[key, value]` 数组。— 可以用来迭代

[`Object.values()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/values)

返回给定对象自身可枚举值的数组。

[`Object.keys()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)

返回一个包含所有自身可枚举属性名称的数组。 — 只遍历自身可枚举属性，`for..in`遍历自身和原型链上的

[`for..in`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...in)

以任意顺序遍历一个对象的除Symbol以外的可枚举属性。— 遍历自身，也遍历原型链上的

[`Object.getOwnPropertyNames()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyNames)

返回一个数组，它包含了指定对象所有的可枚举或不可枚举的属性名。

[`Object.getOwnPropertySymbols()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertySymbols)

返回一个数组，它包含了指定对象自身所有的符号属性。

## **内置对象**

略

## **函数对象**

略

## **数组**

[`Array.from()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/from)

从类数组对象或者可迭代对象中创建一个新的数组实例。— 数组浅拷贝 

[`Array.isArray()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/isArray)

用来判断某个变量是否是一个数组对象。

**以下是数组常用方法的形参、返回值的比较：**

- 记住数组方法的返回值、以及方法是否改变原数组，特别是`splice、slice、sort、forEach、map、reduce`

- 记住数组方法的形参，特别是`push、unshift、splice、concat、indexOf`

- 数组的第一个形参为函数参数，且该箭头函数里有3个参数`item、index、array`的所有数组方法有：

  `every、some、filter、forEach、map、find、findindex`

- reduce有2个参数，第1个为函数参数，第2个为初始值

  `arr.reduce( (preArr,curVal,curIndex,array) => {..},  initialValue )`

  其中`preArr`就是上一轮迭代返回值，若无返回值，那就是undefined

- keys()、values()、entries()

  ES6提供三个新的方法`keys()、values()、entries()`用于遍历数组，它们都返回一个遍历器对象，可以用`for...of`循环进行遍历，唯一的区别是`keys()`是对键名的遍历、`values()`是对键值的遍历，`entries()`是对键值对的遍历
  
- 哪些数组的方法是可能中断的、哪些是无法中断的？

|                                                              | 返回值                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`find`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/find) | 找到返回元素  没有undefined                                  |
| [`findindex`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex) | 找到返回位置  没有-1 （注:比较indexOf()）                    |
| [`includes`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/includes) | Boolean                                                      |
| [`pop`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/pop)、[`shift`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/shift) | 删除的元素                                                   |
| [`push`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/push)、[`unshift`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/unshift) | 新长度（可push、unshift多个值，不止一个也可以）              |
| [`splice()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/splice) | 包含开始下标，由被删除的元素组成的一个数组。如果只删除了一个元素，则返回只包含一个元素的数组。如果没有删除元素，则返回空数组（ 若未删除元素，新的元素会插入在之前（第一个下标的之前） |
| [`slice()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/slice) | 包含开始下标，从数组中删除的元素; 如果无元素为返回空数组     |
| [`concat()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/concat) | 合并多个数组后返回一个新数组（不限个数、不改变原数组）       |
| [`sort()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) | 排序后的原数组（在原数组上排序）                             |
| [`indexOf`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/indexOf) | index，不存在返回-1（接受2个参数，要查找的元素和开始查找的位置） |
| [`lastIndexOf`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/lastIndexOf) | 同上                                                         |
| [`every`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/every) | Boolean                                                      |
| [`some`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/some) | Boolean                                                      |
| [`filter`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) | 满足条件的所有元素                                           |
| [`forEach`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach) | undefined                                                    |
| [`map`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map) | 计算后新的Array                                              |
| [`reduce`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) | 函数累计处理的结果（共接收4个参数分别是`accu`累计器、`curVal`当前值、`curIndex`当前索引、`array`数组） |
| [`reduceRight`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/ReduceRight) | 函数累计处理的结果（与reduce的区别就是顺序：从右往左）       |
| [`flat`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/flat) | 扁平数组，接受一个数字参数表示扁平的层级，第一层为0          |

## **Set**

- Set是值的集合、永远唯一、不重复，Set里的值可以是数值、Symbol、函数等任意类型
  - 构造函数Set：接收一个可迭代对象初始化
  - 实例属性：`.size`获取长度
  - 实例方法：
    - 基本操作：`has()、add()、delete()、clear()` 
    - 迭代操作：`keys、values、entries、forEach、for..of`

  ```js
  数组去重 [...new Set([1,1,2,2,3])]  字符串去重 [...new Set('ababbc')].join('')
  
  Set实现数组的交集、并集、差集
  let a = [1, 2, 3];
  let b = [2, 3, 4];
  
  // 并集
  let union = new Set([...a, ...b]);  // Set {1, 2, 3, 4}
  // 交集
  let intersect = new Set(a.filter(v => b.includes(v)));  // set {2, 3}
  // （a 相对于 b 的）差集
  let difference = new Set(a.filter(v => !b.includes(v)));  // Set {1}
  ```

## **Map**

- Map用于保存键值对，每一个键值对唯一不重复

  - 构造函数Map：接收二维数组初始化
  
  - 实例属性：`.size`获取长度
  
  - 实例方法：
  
    - 基本操作：`has、get、set、delete、clear` 
    - 迭代操作：`keys、values、entries、forEach、for..of`

    注：Set、Map的方法其它一致，只有两点不一样，1.Set的添加是add、Map的添加是set，2.Map有get方法，Set没有
  
- Map与Object对比：

  | Map                                       | Object                                                       |
  | ----------------------------------------- | ------------------------------------------------------------ |
  | key可为任意值                             | key只能是字符串或Symbol                                      |
  | 有序，Map遍历顺序就是插入顺序             | 无序（一般是数字排前）                                       |
  | .size获取map对象里的键值对个数            | 手动计算`Object.keys(obj).length`                            |
  | 可迭代，可用for..of遍历                   | 不可迭代，不能用for..of，若要遍历应该用`for..in`或`Object.keys(obj).forEach` |
  | 不支持JSON（结果永远为`{}`，Set也不支持） | 支持JSON                                                     |
  
  - 少量的数据而言使用Object比Map占用更少的内存，而对于大量的数据且需对大量的数据进行增删改查Map更高效
  
  
  ```JS
  let mySet = new Set([1,2,3])
  let obj = {'a':1,'b':2}
  JSON.parse(JSON.stringify(mySet))   // {}，map使用JSON方法永远等于空对象{}
  JSON.parse(JSON.stringify(obj))   // {'a':1,'b':2}
  ```
  
- 注意，只有对同一个对象的引用，Map结构才将其视为同一个键

  ```js
  const map = new Map();
  
  map.set(['a'], 555);
  map.get(['a']) //undefined
  
  const b = ['a']
  map.set ( b, 555);
  map.get ( b ) //555，只有对同一个对象的引用，Map结构才将其视为同一个键
  ```

  ```js
  //（二维）数组传Map
  new Map([['a',1],['b',2]])
  //Map转数组
  [...new Map([['a',1],['b',2]])]
  
  //对象转Map
  const obj = {'a':1,'b':2}
  const map = new Map(Object.entries(obj))    
  //Map转对象
  const entries = new Map([['foo', 'bar'],['baz', 42]]);
  const obj = Object.fromEntries(entries);
  ```

## **WeakSet、WeakMap**

- **WeakSet**
  - `WeakSet`的成员只能是对象，而不能基本数据类型
  - `WeakSet`中的对象都是弱引用，即垃圾回收机制不考虑`WeakSet`对该对象的引用（弱引用随时会消失，ES6规定`WeakSet`不可遍历）
  - 只有`add、delete、has`方法，不可使用迭代方法
  - **WeakSet和Set：**
    - 与`Set`相比，`WeakSet`只能是对象的集合，而不能是任何类型的任意值
    - `WeakSet`持弱引用：集合中对象的引用为弱引用。 如果没有其他的对`WeakSet`中对象的引用，那么这些对象会被当成垃圾回收掉。 这也意味着`WeakSet`中没有存储当前对象的列表。 正因为这样，`WeakSet`是不可枚举的
  
- **WeakMap**
  - 是一组键值对的集合，其中的键是弱引用的。其键必须是对象（不能是原始数据类型），而值可以是任意
  - 相比之下，原生的`WeakMap`持有的是每个键对象的“弱引用”，这意味着在没有其他引用存在时垃圾回收能正确进行。原生`WeakMap`的结构是特殊且有效的，其用于映射的key只有在其没有被回收时才是有效的
  - `WeakMap`是弱引用，因此很适合用来跟踪对象引用，如vue3就是使用WeakMap来确定Effect的

## **Proxy**

- `Object.defineProperty `
  
  - **定义**：
    - 数据劫持，在一个已有对象上定义一个新属性，或修改一个现有属性，并返回此对象，可利用`getter、setter`函数来控制对象的访问
  - **特点**：
    - 无法检测数组元素变化，无法检测数组长度修改
    - 无法检测对象属性的添加或删除
    - 实现响应式时，必须遍历对象的每个属性，为每个属性都进行`get、set`拦截。且对象有嵌套属性时还需递归遍历为其绑定`get、set`从而实现响应式
  
- `Proxy    `

  - **定义**：

    - 用于创建一个对象的代理，不在局限某个属性，而是直接对整个对象进行代理，目标对象可以是数组、对象、函数、代理..等任意类型，Proxy可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写（注：Proxy没有prototype，因此也无法使用instanceof）

  - **特点**：
  
    - Object.defineProperty只get、set这2个拦截器，而Proxy有13个，几乎覆盖所有对数据的访问操作
    - 可以检测到数组元素变化、可以检测到对象属性的添加或删除
    - 针对整个对象，而不是对象的某个属性。但仍然是浅层的，因此若对象有嵌套属性时还需递归遍历为其绑定拦截器从而实现响应式
  
  - **参数：**
  
    - `target`：源对象
    - `handler`：代理操作/拦截器/拦截函数，对`proxy`对象进行拦截和操作
  
  - **handle：**（Proxy支持的拦截操作一览，一共13种）
  
    - get(target, propKey, receiver)
  
      拦截对象属性的读取，比如`proxy.foo`和`proxy['foo']`。
  
    - set(target, propKey, value, receiver)
  
      拦截对象属性的设置，比如`proxy.foo = v`或`proxy['foo'] = v`，返回一个布尔值。
  
    - has(target, propKey**)**
  
      拦截`propKey in proxy`的操作，返回一个布尔值。
  
    - deleteProperty(target**, **propKey**)**
  
      拦截`delete proxy[propKey]`的操作，返回一个布尔值。
  
    - ownKeys(target)
  
      拦截`Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`、`for...in`循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而`Object.keys()`的返回结果仅包括目标对象自身的可遍历属性。
  
    - getOwnPropertyDescriptor(target, propKey)
  
      拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象。
  
    - defineProperty(target, propKey, propDesc)
  
      拦截`Object.defineProperty(proxy, propKey, propDesc）`、`Object.defineProperties(proxy, propDescs)`，返回一个布尔值。
  
    - preventExtensions**(**target)
  
      拦截`Object.preventExtensions(proxy)`，返回一个布尔值。
  
    - getPrototypeOf(target**)**
  
      拦截`Object.getPrototypeOf(proxy)`，返回一个对象。
  
    - isExtensible(target)
  
      拦截`Object.isExtensible(proxy)`，返回一个布尔值。
  
    - setPrototypeOf(target, proto)
  
      拦截`Object.setPrototypeOf(proxy, proto)`，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
  
    - apply(target, object, args)
  
      拦截 Proxy 实例作为函数调用的操作，比如`proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`。
  
    - construct(target, args)
  
      拦截 Proxy 实例作为构造函数调用的操作，比如`new proxy(...args)`。

## **Reflect**

- `Reflect`
  - **参数：**
    - 部分Reflect方法里接收可选参数receiver，receiver可以理解为Reflect方法调用时的this值

  - **特性：**
    - `Reflect`是一个内置对象，相当于`Object`原生对象的一个副本，保留并优化原有的默认行为和方法

    - `Reflect`对象的方法与`Proxy`对象的方法一一对应，只要是`Proxy`对象的方法，就能在`Reflect`对象上找到对应的方法。这就让`Proxy`对象可以方便地调用对应的`Reflect`方法，完成默认行为，作为修改行为的基础。也就是说，不管`Proxy`怎么修改默认行为，你总可以在`Reflect`上获取默认行为（一句话说，为操作源对象提供了一个方便途径）

# **手写代码**

## **防抖、节流**

- **防抖**
  
  - 在规定时间里，事件处理函数只执行一次，若规定时间内多次触发，则重新计时
  
  ```js
  function debounce(fn,delay){
      let timer = null  
      return function() {  //借助闭包
          if(timer){ clearTimeout(timer) }   //规定时间内再次触发则重新计时
          timer = setTimeout(fn,delay) 
      }
  }
  
  function showTop  () {
      var scrollTop = document.body.scrollTop || document.documentElement.scrollTop;
  　　 console.log('滚动条位置：' + scrollTop);
  }
  window.onscroll = debounce(showTop,200) 
  ```
  
- **节流**
  
  - 在规定时间里，事件处理函数只执行一次，若在规定时间内多次触发函数，也只执行第一次，直到过了规定的时间，再次触发函数，函数才会再次执行
  
  ```js
  function throttle(fn,delay){
      let timer = null
      return function() {
         if(timer){ return false }   //规定时间内再次触发忽略
         timer = setTimeout(() => {	
              fn()
              timer = null;  //执行结束后赋值为null
          }, delay)
      }
  }
  
  function showTop  () {
     var scrollTop = document.body.scrollTop || document.documentElement.scrollTop;
  　　console.log('滚动条位置：' + scrollTop);
  }
  window.onscroll = throttle(showTop,200)
  ```
  
- **防抖和节流的比较**

  防抖和节流，都用于高频事件、短时间内连续触发的事件，以滚动事件为例，不同之处如下，
  - 防抖为规定时间内多次触发就重新计时，因此如果一直拖滚动条进行滚动，那就永远无法得到scrollTop值
  - 节流为规定时间内多次触发，只有第一次有效，其余忽略，因此如果一直拖着滚动条进行滚动，还是会以规定的时间间隔持续输出scrollTop的值

- **防抖和节流其它使用场景**
  
  - 搜索框input事件，例如要支持输入实时搜索可以使用节流方案（间隔一段时间就必须查询相关内容），或者实现输入间隔大于某个值（如500ms），就当做用户输入完成，然后开始搜索，具体使用哪种方案要看业务需求
  
  - 页面resize事件，常见于需要做页面适配的时候。需要根据最终呈现的页面情况进行dom渲染（这种情形一般是使用防抖，因为只需要判断最后一次的变化情况）

## **对象/数组的深/浅拷贝**

掌握对象、数组的所有浅拷贝、深拷贝方法，其实也等同于问为如何拼接、合并多个对象或数组，答案是一样的

- **对象**
  
  - 对象浅拷贝
  
    - `...`  展开运算符
  
    - `Object.assign`（忽略对象的不可枚举属性，只拷贝对象的可枚举属性）
  - 对象深拷贝

    - `JSON.parse(JSON.stringify())`（缺点：无法拷贝函数、Symbol、undefined的值，Map、Set使用JSON方法永远返回空对象{}，因此需要自己封装深拷贝函数）
    - 封装递归拷贝函数
- **数组**
  
  - 数组浅拷贝
  
    - `...`展开运算符
    - `Array.from()`
    - `concat`
    - `slice、splice`（splice插入元素）
    - `push、unshift`（因为这2个都可以接受多个参数）
  - 数组深拷贝
  
    - `JSON.parse(JSON.stringify())` 
    - 封装递归拷贝函数

```js
// 递归函数实现数组或对象的深拷贝（利用hash解决循环引用）
function recursive(obj,hash=new WeakMap()) {
    // 终止条件
    if (typeof obj !== 'object' || obj == null) return obj
    if (hash.has(obj)) return hash.get(obj)  
     
    let res = obj instanceof Array ? [] : {}
    hash.set(obj,res)
    
    for (let key in obj) {  //for..in遍历自身和原型链上的属性
        if (obj.hasOwnProperty(key)) {  //排除原型链上的属性，只遍历出自身的
            res[key] = typeof key == 'object' ? recursive() : obj[key]
        }
    }
    return res
}

// 拼接、合并数组（数组浅拷贝）
let arr_1 = [1,2,3]
let arr_2 = [4,5,6]
console.log('(1)', [...arr_1,...arr_2])
console.log('(2)', arr_1.push(...arr_2))
console.log('(3)', arr_1.unshift(...arr_2))
console.log('(4)', arr_1.splice(0,0,...arr_2))
console.log('(5)', arr_1.concat(arr_2))
console.log('(6)', arr_1.push.apply(arr_1,arr_2))
```

## **递归和扁平**

```js
// 递归
function trans2Tree(arr) {
    let res = [], parentArr = []
    for (let i = 0; i < arr.length; i++) {
        if (!arr[i].parent) {
            parentArr.push(arr[i])
        }
    }
    if (parentArr.length !== 0) {
        res = tree(parentArr, arr)
    } else {
        res = parentArr
    }
    function tree(parentArr, arr) {
        if (arr.length == 0) return parentArr
        parentArr.forEach(parent => {
            let childArr = []
            for (let i = 0; i < arr.length; i++) {
                if (arr[i].parent == parent.id) {
                    childArr.push(arr[i])
                }
            }
            parent.children = childArr
            if (childArr.length > 0) {
                tree(childArr, arr)
            }
        })
        return parentArr
    }
    return res
}

// 扁平，可用es6的flat()，或自己手写代码把嵌套的一层层递归出来
```

## **去重**

- **数组去重**


```js
let arr = [1,2,2,2,3,3]
let res = []

// Set（最快速的方法，不过只适用于数组里的元素都是基本数据类型）
res = Array.from(new Set(arr))  // new Set(arr) => Set(3) {1, 2, 3}
res = [...new Set(arr)]

// Map
const map = new Map()
arr.forEach( v => map.set(v, true) )
res = [...map.keys()]  //[1,2,3]，比较利用obj方法得到的返回值

// obj（同Map写法类似，但是元素值的类型会从Number变成String）
const obj = {}
arr.forEach( v => obj[v]=true )
res = Object.keys(obj)  //['1','2','3']，因为对象的key只允许字符串或Symbol

// filter（用indexOf判断返回值是不是当前下标）
res = arr.filter((v,index)=> arr.indexOf(v) === index)

// indexOf 或 lastIndexOf 返回-1时存入结果数组
arr.forEach( v => res.lastIndexOf(v) == -1 ? res.push(v) : '' )

// includes 返回false时存入数组
arr.forEach( v => res.includes(v) ? '' : res.push(v) )

// reduce+includes，把累加器结果不存在的值添加进去
res = arr.reduce((pre,cur)=> pre.includes(cur) ? pre: [...pre,cur], [])

// sort排序后比较相邻值
arr.sort((a,b)=>a-b) 
res.push(arr[0])    //先push一个进去
for (let i=1;i<arr.length;i++) {
  if (arr[i] !== arr[i-1]) {   //排序后比较相邻值
      res.push(arr[i]);
  }
}
```

## **排序**

```js
let arr = [3, 2, 10, 0, 5, 4, 9, 1, 5, 7, 6, 8, 1];

// 冒泡排序-时间O(N²)
function bubbleSort(arr) {
    for (let i = 0; i < arr.length; i++) {
        for (let j = 0; j < arr.length - 1; j++) {
            if (arr[j] > arr[j + 1]) {   //依次比较相邻，就像在不断冒泡
                [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]]
            }
        }
    }
    return arr
}

// 选择排序-时间O(N²)
function selectSort(arr) {
    for (let i = 0; i < arr.length; i++) {
        let index = i  //index表示最小数的索引
        for (let j = i; j < arr.length; j++) {  //从[i,arr.length-1]里选择出一个最小值
            if (arr[index] > arr[j]) {
                index = j
            }
        }
        [arr[index], arr[i]] = [arr[i], arr[index]]
    }
    return arr
}

// 插入排序-时间O(N²)
function insertSort(arr) {
    for (let i = 1; i < arr.length; i++) {   //从下标为1的位置开始遍历
        let j = i    //此时已排序数组为[0,i]，未排序数组[j=i,arr.length-1]
        while (j > 0 && arr[j-1] > arr[j]) {    // 抽出第j张卡，插入到[0,j]数组里的正确位置
            [arr[j], arr[j - 1]] = [arr[j - 1], arr[j]]
            j--
        }
    }
    return arr
}
console.log('插入', insertSort(arr_3));

// 快速排序-时间O(N*log2N)
function quickSort(arr) {
    if (arr.length < 2) return arr  //递归终止条件，无这个if判断会导致栈溢出
    let pivot = arr[0]
    let bigArr = [], smallArr = []
    for (let i = 1; i < arr.length; i++) {
        arr[i] < pivot ? smallArr.push(arr[i]) : bigArr.push(arr[i])  //arr[i]为基准数
    }
    return [...quickSort(smallArr), pivot, ...quickSort(bigArr)]
}

// 归并排序-O(N*log2N)
function merge(leftArr, rightArr) {
    let res = []
    while (leftArr.length > 0 && rightArr.length > 0) {
        if (leftArr[0] < rightArr[0]) {
            res.push(leftArr.shift())
        } else {
            res.push(rightArr.shift())
        }
    }
    return res.concat(leftArr, rightArr)
}
function mergeSort(arr) {
    if (arr.length < 2) return arr  //递归终止条件
    let mid = Math.floor(arr.length / 2)
    let leftArr = arr.slice(0, mid)
    let rightArr = arr.slice(mid, arr.length)
    return merge(mergeSort(leftArr), mergeSort(rightArr))  //每个数字分为一组后再顺序合并merge
}
```

## **正则**

- **表达式**（下面的reg为自定义的匹配规则，str为自定义字符串）
  - reg方法：
    - `reg.exec(str)`，返回包含第一个匹配信息的数组，包含两个额外的属性：index 和 input
    - `reg.test(str)`，匹配成功返回true，失败返回false
  - string匹配：
    - `str.search(reg, function(){})`，匹配成功返回首次匹配位置，失败返回-1
    - `str.match(reg, function(){})`，匹配成功返回匹配结果数组，失败返回null
    - `str.replace(reg, function(){})`，匹配后的新字符串
    - `str.split()`
  
- **基础**
  - 匹配位置：
    - `^`  表示匹配字符串的开始位置（例外：用在中括号中`[]`时，表示为取反，即不匹配括号中字符串）
    - `$`  表示匹配字符串的结束位置
  - 匹配次数：
    - `?`  表示匹配零次或一次
    - `+`  表示匹配一次到多次 (至少有一次)
    - `*`  表示匹配零次到多次
  - 括号：
    - `() ` 小括号表示匹配括号中全部字符
    - `[]` 中括号表示匹配括号中任意一个字符，描述范围，如`[0-9 a-z A-Z]`
    - `{}` 大括号用于限定匹配次数，如{n}表示匹配n个字符，{n,}表示至少匹配n个字符，{n,m}表示至少n,最多m
  - 通过`$`或`\`获取小括号匹配内容：
    - `$n` 以`$`加一个数字的形式表示第几个小括号里匹配的内容，在回调函数里（`function(){}`）使用
    - `\n` 以`\`加一个数字的形式表示第几个小括号里匹配的内容，在正则规则定义里（`//g`） 使用
  - 范围、以`\`开头：
    - `\w` 表示英文字母和数字，`\W ` 非字母和数字
    - `\d`  表示数字，`\D` 非数字
    - `\s` 空格，`\S` 非空格
  - 其它：
    - `\`  转义字符，如`\*`表示匹配*号
    - `|`  表示为`或`，两项中取一项
    - `.`  表示匹配单个字符 
    - `g` 全局匹配，`i` 不区分大小写

```js
常见的正则匹配：（代码见js文件）
    去掉首尾、中间的所有空格
    变量名转换为驼峰
    验证手机号码是否为11位、加密手机号码为'124****7890'的形式
    身份证号
    车牌号码
    校验账号、密码，规则必须是字母、数字、_组成，且长度为5-20
    邮箱(如2433378@qq.com)
    url
```

# **基础**

## **值分类**

- **基本数据类型**
  
  - `null、undefined、boolean、number、string、Symbol、BigInt`
  
    注：`Symbol`是一个内置对象，Symbol值作为对象属性名时，访问时不能用点运算符
    
    ​        `BigInt`是一个内置对象，可表示任意大整数，以数字+n结尾表示，可以保证大整数的精度问题
  
- **引用数据类型**
  
  - `object、array、map、set、weakmap、weakset、function..`

## **`null、undefined`区别**

- **null：**表示空值
  - 作为对象原型链终点
  - 当使用完闭包、或使用完一个较大对象时，可将其设为null释放内存

- **undefined：**表示"缺少值"，即此处应有值但还未定义
  - 变量已声明但未赋予初始值，`let a; console.log(a) => undefined `
  - 调用对象并不存在的属性，`let obj = {}; console.log(obj.a) => undefined`
  - 调用函数时，未给形参提供实参，`function fn(a){ console.log(a) }; fn(); => undefined`
  - 函数没有返回值时，默认返回undefined

- **null、undefined 相同点：**
  - 都为基本数据类型/原始数据类型

  - 转换为布尔值都是false
    - `!!null  //false`
    - `!!undefined  //false`

- **null、undefined 不同点：**
  - 类型不一样：
    - `typeof null   //object，注：null intanceof Object -> false`
    - `typeof undefined  //undefined`

  - 转化为数值时不一样：
    - `Number(10+null);  //10，null转为0`
    - `Number(10+undefined);  //NaN，undefined转为NaN`

## **`typeof、instanceof`区别**

- `typeof`：

  - 判断基础数据类型返回对应的基础数据类型

  - 判断函数返回`'function'`（判断String、Number..内置对象也返回'function'）

    typeof Array->'function'   typeof [] -> object

  - 判断除函数外的引用数据类型都是返回`'object'`  

  - 特殊的返回值：`typeof null => object`

- `instanceof`：

  - 构造函数的`prototype`属性是否出现在某个实例对象的原型链上

    注：`[] instanceof Array、[] instanceof Object => true`（所有引用类型都是Object实例）

## **`var、let、const`区别**

- `var`
  
  - 函数作用域，有变量提升
  
- `let、const`
  
  - 块级作用域、无变量提升、存在“暂存性死区”
  
  注：根据YDKJS一书其实let、const也有变量提升，只是“暂存性死区”规定它们两从块级作用域到声明的地方不可使用而已。“暂存性死区”也可以理解为只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量

## **`trusy、falsy`**

- `falsy`的所有情况：null | undefined | NaN | 0 | -0 | 0n |  "" | '' | ``（共9个）

## **默认参数、剩余参数、`arguments`参数**

- 默认参数：

  - 没有值或传入`undefined`时使用默认形参

- 剩余参数：

  - 将一个不定数量的参数表示为一个数组，必须写在函数形参的最后一个（否则报错）

- `arguments`：

  - 是一个对应于传递给函数（非箭头函数）的参数的类数组对象，`arguments`是类数组，只能使用index访问元素和length获取长度，其它数组的方法都不可使用
  - 除了有length属性、还有callee属性
  
  注：剩余参数、`arguments`区别：
  
  - 剩余参数只包含那些没有对应形参的实参，而`arguments`对象包含了传给函数的所有实参
  - 剩余参数是数组，能够使用数组上的所有方法，`arguments`是类数组，只能使用index访问元素和length获取长度，其它数组的方法都不可使用
  - `arguments`对象还有一些附加的属性（如`callee`属性）

## **创建对象、判断数组、创建函数**

- **创建对象方法：**

  - 字面量创建 `let obj = {}`
  - Object或自定义函数通过new创建 `let obj = new Object({})`
  - create函数 `let obj = Object.create(proto,[propertiesObject])`

- **判断是否数组：**

  - `Array.isArray([])`
  - `[] instanceof Array`
  - `[].constructor == Array`

  注：不能用`typeof [] -> object`，因为`typeof`是用来判断基本数据类型的，判断引用数据类型都只会返回object

- **创建函数：**

  - 函数声明法，`function fn(){}`   （只有这种方式会变量提升，即在这之前调用fn()不会报错）
  - 函数表达式法，`var fn = function Fn(){} `   // 这里只是fn变量的var提升且fn变量还未赋值，因此无法在之前执行fn()/Fn()
  - 构造函数法，`var fn = new Function()`（不推荐）
  - 箭头函数

## **柯里化函数**

- 是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数的新函数，bind的实现机制就是柯里化

##**`ES6`新增**

- 新增：
  - `let、const` 
  - `Map、Set、WeakSet、WeakMap`
  - `Proxy、Reflect`
  - `promise`、`async await`
  - `for..of`、迭代器和生成器
  - `...`展开运算符
  - 解构赋值（数组解构时有序、对象无序）
  - 默认参数、剩余参数、`arguments`参数
  - 箭头函数
  - `sessionStorage、localStorage`

## **attribute、property区别**

- `attribute`是`html`标签自定义的属性
- `property`就是dom元素在js中作为对象拥有的属性

## **增、删、改、查节点**

```js
// 创建
createDocumentFragment()  //创建DOM片段
createElement()           //创建具体的元素
createTextNode()          //创建文本节点

// 查找
getElementsByTagName()
getElementsByName() 
getElementById()

appendChild()      //添加
removeChild()      //移除
replaceChild()     //替换
insertBefore()     //插入
```

## **事件代理/委托、事件机制**

- 事件代理：

  - 也称事件委托，利用dom元素的事件冒泡，把子元素事件委托给父元素进行监听和触发，从而减少事件注册、节省内存占用、提高性能

- 事件机制：

  - 捕获 -> 目标阶段 -> 冒泡
  
  ```js
  <ul id="myLinks">
   <li id="goSomewhere">Go somewhere</li>
   <li id="doSomething">Do something</li>
   <li id="sayHi">Say hi</li>
  </ul> 
  let list = document.getElementById("myLinks");
  //addEventListener的第三个参数可以设置为冒泡或捕获，默认为冒泡
  list.addEventListener("click", (event) => {  
   let target = event.target;
   switch(target.id) {
     case "doSomething":
       document.title = "I changed the document's title";
       break;
     case "goSomewhere":
       location.href = "http:// www.wrox.com";
       break;
     case "sayHi":
       console.log("hi");
       break;
    }
  });
  ```

# **else**

- 作用域链


- 预编译阶段


- 手写一个promise

- 构造函数、普通函数、回调函数、自调用函数、递归函数分别是什么、之间有和区别

- 说出数组`[]`的原型链，`[] - Array.prototype - Object.prototype - null`

- 验证用户月份输入是否正确，不要简单粗暴的if(month<=12)，用正则匹配去做

- 为什么说js是单线程的

- array/set的互转、array/map互转、object/set的互转、object/map的互转

- promise网络请求超时时优化处理：(采用promise.race())

  ```js
  const promise = new Promise( (res,rej)=>{
    setTimeout(res, 5000, "hello world");
  });
  
  const abort = new Promise( (res,rej)=> {
    rej({
      name: "abort",
      message: "the promise is aborted",
      aborted: true
    });
  });
  
  Promise.race([promise, abort])
    .then(console.log)
    .catch(e => {
      console.log(e);
  });
  ```


- 进制缩写：

  0b 二进制，0x 16进制

- 如何实现能够链式调用的函数（return this）

- 发送网络请求时，都要对headers进行encodeURIComponent，服务器要拿headers参数时要dencodeURIComponent（待验证）

- async、defer、type='module'的区别

- 基本数据类型和引用数据类型有什么区别？

- 实现一个promise.all、promise的源码实现？

- 

