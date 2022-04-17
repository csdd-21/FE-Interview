# ts面试题

## ts特定、和js区别

弱语言、强语言：定义变量时指定类型，静态语言、动态语言：运行时是否允许改变变量类型

js：弱语言、动态语言

ts：强语言、静态语言，支持js、es6，支持面向对象编程的概念，如**接口、类、抽象类、泛型、继承、多态**..，ts会先编译成js文件后在浏览器运行，缺点：编译时间长

## 常用命令

tsc --init 生成配置文件

tsc  xx.ts   //运行

tsc --watch   //修改实时自动编译

tsc --outFile xx.js x1.ts x2.ts ..  //多个ts合并为1个js

1：number

## 类型基础类型

null、undefined、boolean、string、number、symbol

array、object

tuple、enum、void、never、unknown、any（js没有的）

**特别：**

array：string[ ]、number[ ]

tuple：[string, number, number]    //['abc', 1, 2]

unknown：只能赋值给any或unknown

never：抛出异常或无返回值时可定义

## 参数

默认值： x =

可选类型： x?

联合类型： |

交叉类型：&

## rest参数

允许函数传递可变数量的参数，规则：1个函数只能有1个rest参数，只能出现在函数列表的最后一个，参数必须是数组类型

## 枚举

用途：带名字的常量，创建一组有区别的用例

分类：**数字枚举**：自增长、反向映射（可用index获取值），**字符串枚举**：不可，**异构枚举**：前两者混合

## 泛型

用<T>表示类型，T可以自定义，支持当前和未来的数据类型，提高复用性

同any区别：若传入和返回的类型应该相同，any作为类型会丢失信息，因为any可以返回任何类型的值，而<T>可限制返回和传入的值一致，保持准确性

## 类型断言、类型推论

类型断言：as：对变量指明一个更具体的值，只在编译时起作用，写法：

​                           <类型>变量

​                           变量 as 类型 （在tsx中只能使用这种方式）

类型推论：在设置默认参数值、决定函数返回值时进行类型推论，即自动类型注解

typescript中的as：类型断言

## !/?./!!/??作用

!   非空类型断言，确定有值，跳过ts在编译时对它的检测

?. 可选连，不存在时返回undefined，存在时返回值

!!  转为布尔值

?? 有值赋值，无值赋默认值

类型缩小

typeof、===、instanceof、in..

## 接口

**定义：**对值的结构进行类型检查、为类型命名和代码定义契约，接口类型可以是对象、函数、类..

**特点：**接口可以定义为对象、数组、函数、类等

​            可设置为可选参数？、联合参数|、只读参数readonly

​            可以实现继承、以及implements实现接口

readonly与const：最简单判断该用readonly还是const的方法是看要把它做为变量使用还是做为一个属性。 做为变量使用的话用const，若做为属性则使用readonly

类型检查器不会去检查属性的顺序，只要相应的属性存在并且类型也是对的就可以

## type、interface区别

相同点：两者都可以用来描述对象、函数

不同点：1.type可以描述基本数据类型、联合/交叉类型、元祖，interface不可

​               2.interface有implements、extends（接口继承、类继承），type不可

​               3.interface可以声明合并，type不可

## 类

**继承：**extends，在派生类的构造函数中必须使用super()执行基类的构造函数，且必须在访问this属性之前调用

**修饰器：**public、private、protected、readonly，区别：**public**外部可访问，且在ts中成员都默认为public，**private**只有父类可访问，**protected**父类和派生类都可访问，**readonly**只读、必须在声明或构造函数里被初始化（ts中默认都是public）

**存取器**：可通过get xx/set xx来获取或改变对象成员的值，从而有效控制对对象成员的访问

**静态属性**：static、属性存在在类本身而不是类的实例上，用static xx定义，用类名.xx访问

**抽象类：**abstract、一般作为派生类的基类、不会被直接实例化，抽象方法与interface接口方法类似，只定义方法签名不包含具体实现，要求必须在派生类中定义抽象方法的实现

## 方法重写

在派生类中定义与基类相同的方法，会被方法重写/方法覆盖

避免构造函数的重写：super()，避免普通函数的重写：super.xx()，两者都必须写在最前

## 函数重载

根据传入参数类型的不同返回不同

function pickCard(x: {suit: string; card: number; }[]): number;   //重载1

function pickCard(x: number): {suit: string; card: number; };     //重载2

function pickCard(x): any {}   //不是重载列表，而是函数方法主体

## 模块

新的ts版本里规定“内部模块”现在称做“命名空间”， “外部模块”现在则简称为“模块”，即module xx =》替换为namespace xx{ }，模块有自己的作用域，因此需要export+import后供外部访问

**export**导出/import { xx } from导入，**export default**导出/一个模块只能有一个export default，import xx from导入

为了支持CommonJS和AMD语法，ts提供了export **=**导出，import xx = require('')导入

## 命名空间

命名空间可以将代码包裹起来，只通过export暴露需要在外部访问到的对象，避免命名冲突

和模块的区别：命名空间主要避免命名冲突，模块主要代码复用，一个模块里可能有多个命名空间

## 声明合并

可以合并接口、可以合并命名空间，但不可以合并类

如果想复用类，可以使用extends、mixins

## 混入mixins

类的复用除了可以用面对对象继承方式extends，还可以用混入mixins

类A implements 类B，类C ，表示将B,C混入A中，在这里类被当成了接口

## else

static、readonly

extends

super

abstract

type、interface、implements

public、private、protected

get xx、set xx 存取器 

封装

泛型推断、自动指定泛型、多个泛型、T extends

 解释下Typescript的装饰器是什么？

Declare关键字是干嘛用的、TypeScript Declare关键字?

tsconfig.json文件有什么作用？

什么是Typescript映射文件？

说说TS的模块解析策略，什么是相对导入？什么是非相对导入

函数签名

可索引类型接口

数字索引、字符串索引是什么？区别？

never和void的区别

交叉类型、联合类型的区别

# node、计算机网路面试题

## node特点

基于v8、js，提供js运行环境，单线程、异步I/O避免js阻塞

## 错误优先回调、回调地域、避免回调地域

错误优先回调：node的所有异步api的回调函数callback的第一个参数都是err

## 事件循环eventloop

事件循环把同步任务全部进栈然后出栈,把异步任务拿开,等满足条件后,进入事件队列, 任务栈执行完毕 出栈 然后检查事件队列 加入任务栈执行
事件队列的任务有两种 宏任务 微任务 事件队列 有宏任务队列(ajax,settimeout,dom监听,uirendering)和 微任务队列 (promise) 先执行微任务队列 然后执行宏任务队列 宏任务队列执行一个完 就会去看有没有微任务队列可执行
async的await 语句相当于promise的立即执行部分 await 后面的相当于.then 加入 微任务队列 不管是啥都进入微任务 await 函数也是一样

## 模块加载机制、模块缓存

require('./xx')：先在当前路径下找是否有xx.js文件，没有就找xx文件夹，找xx文件夹里是否有index.js，没有index.js文件就在package.json里的main属性找入口文件，找不到main入口文件就报错

require('xx')：从当前目录下的node_modules开始找，一直找到全局的node_modules文件夹，其余和上面一致

## import/require、export/exports、exports/module.exports区别

**import/require：**

import：（js）es6、异步

require：（node）commonjs、同步、会阻塞

**exports/module.exports：**

node中，每个模块都会有1个module对象，且在文件里默认exports=module.exports，文件最后会return module.exports

## Buffer、fs、http

## 实现一个自定义的stream（写一个读取文件方法）

## webSocket

## 异步

promise对象表示一个异步操作的最终结果

3种状态：pending、fullfilled、rejected，若已被fullfilled、rejected则可表示已被settled

构造函数：new Promise()

静态方法：.all()、.allSettled()、.any()、.resolve()、.reject()

## v8的垃圾回收机制、内存泄漏

标记清除、引用计数

标记清除：当变量进入执行环境时将其标记为“进入环境”。当变量离开环境时将其标记为“离开环境”

引用计数：跟踪记录每个值被引用的次数，当声明了一个变量，若被引用则引用次数+1，若不再被引用减1，当这个引用次数变成0时说明没有办法再访问这个值了，便可将其所占的内存空间回收

内存泄漏：避免滥用闭包

## XSS、CSRF攻击

xss：跨站脚本攻击

防御xss攻击：1.对输入和url参数进行过滤

​                         2.对输出进行编码，如<转成%lt，>转成&gt

​                         3.设置cookie为httpolny

csrf：跨站请求伪造，攻击者盗取用户cookie后以用户身份访问服务器

防御csrf：1.使用post，不要get

​                  2.加入验证码

​                  3.服务器请求时在请求头上加上token

## 进程、线程

用户运行一个程序时会产生1个或多个进程，进程相当于是线程的容器，线程是程序的基本运行单位，计算器会为进程分配资源，这些资源在线程间共享

程序与进程对应关系：1对n

进程与线程对应关系：1对n

## npm -g、-S、-D区别，devDependencies、dependencies有区别

**npm -g**：安装在系统全局位置，npm -S：安装生产环境下需要的module，注册到package.json里的dependencies中，在生产和开发环境下都可用，**npm -D**：安装在开发环境，注册到package.json的devDependencies中，打包到生产环境时不会把这些module打包进去

devDependencies：生产环境运行时需要的module，dependencies：开发环境

## MIME类型、url主体、 http/https区别、http请求头格式、http status状态码

**http状态码：**

1xx：指示信息——请求已接收，继续处理

2xx：成功——请求成功

3xx：重定向——要完成请求必须进行更进一步的操作

4xx：客户端错误——请求有语法错误或请求无法实现

5xx：服务器端错误——服务器未能实现合法的请求

## http工作原理

1、客户端连接到Web服务器 

2、发送HTTP请求

3、服务器接受请求并返回HTTP响应

4、释放连接TCP连接

5、客户端浏览器解析HTML内容

## axios、fetch区别

1.都提供网络请求api、都支持异步

2.fetch写法.then().then()，axios写法是.then().catch()，fetch的第一个.then()里必须用return res.json()把响应体return出来

2.请求失败返回的promise对象，fetch不会把它标记为reject、而是在resolve里把ok属性标记为false，axios会标记为reject

3.axios可以用interceptors拦截请求和响应，fetch不可

## else

异步I/O流程

在每个tick的过程中，如何判断是否有事件需要处理呢

**onenote补充**

node常问的面试题？http server非常重要

restful api 是什么？非restful api 是什么？

