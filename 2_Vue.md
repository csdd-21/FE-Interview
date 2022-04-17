# **掌握原理**

## **响应式原理、异步更新**

本节需要掌握vue2、vue3各自的响应式原理、vue2响应式原理的弊端/为何改进、如何收集依赖、何时触发依赖实现更新、异步更新机制是什么/优点

- **vue2实现响应式：**
  
  - **原理：**
    - 实例初始化时遍历data里的所有property，并使用Object.defineProperty把这些property全部绑定getter/setter函数，访问对象的属性时getter函数触发并通过dep.depend()收集依赖，修改对象属性的值时setter函数触发并通过dep.notify()通知watcher
    - 每个组件实例都对应一个watcher实例，它会在组件渲染的过程中把“接触”过的数据property记录为依赖。之后当依赖项的setter触发时，会通知 watcher，从而使它关联的组件重新渲染
    - vue2对对象的检测是，为对象的每一个属性绑定Object.defineProperty的get、set方法（包括嵌套属性，需要递归）从而实现响应式，而对数组的检测，是对数组变更方法进行重载（7个方法，push()、pop()、shift()、unshift()、splice()、sort()、reverse()）
    - Vue无法检测property的动态添加或移除，因此初始化过后仍需添加响应式property应使用Vue.set()或Vue.delete()
  - **特点：**
    - 无法检测数组元素的新增或删除，需要对数组方法进行重写，无法检测数组长度修改
    - 无法检测到对象属性的添加或删除
    - 必须遍历对象的每个属性，为每个属性都进行get、set拦截
    - 必须遍历深层嵌套的对象属性
  
- **vue3实现响应式：**

  - **原理：**
    
    - vue3中用ref、reactive来创建响应式数据，ref是RefImpl类的实例，创建一个ref实例会返回带有value属性的对象，之后对该ref对象的读取和修改都是通过value属性，在RefImpl类内部为value绑定了getter、setter函数从而达到对ref数据访问的控制。ref用来包裹基本数据类型，如果用ref包裹引用数据类型，vue还是会将该引用数据类型先用reactive处理后再由ref处理
    
    - 而reactive一般用来包裹引用数据类型，包裹基本数据类型时会报错。vue3对引用数据类型的响应式处理抛弃了Object.defineProperty，改成了用Proxy实现。用reactive创建数据后返回的是proxy代理对象，之后便可在该代理对象上进行操作，Proxy在代理对象前架设了一层拦截器来控制对象访问，相比js的Object.defineProperty只提供了对象属性的get、set拦截操作，es6的Proxy提供了13种拦截操作
    
      （proxy本身只能用来代理对象，而不能代理基本数据类型，因此vue3还需ref对原始值响应式处理）
    
    - computed的处理方式和ref基本一致，computed里的参数为函数时，默认为getter函数且返回一个不可变的响应式ref对象，而参数为一个具有get、set函数的对象时，可用来创建可写的ref对象
    
  - **特点：**
    
    - 不需要对数组的方法进行重载（解决vue2问题）
    - 可以检测到对象属性的添加或删除，Proxy支持多达13种拦截操作（解决vue2问题）
    - 针对整个对象，而不是对象的某个属性
    - 仍需要将嵌套对象属性进行遍历为响应式 
  
  注：响应式处理发生在生命周期beforeCreate、created之间
  
- **异步更新：**
  
  - 异步更新，只要侦听到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环（‘tick’）中发生的所有数据变更。在缓冲时去除重复数据对于避免不必要的计算和DOM操作是非常重要的。如果同一个 watcher 被多次触发，只会被推入到队列中一次。然后，在下一个的事件循环“tick”中，Vue刷新队列并执行实际 (已去重的) 工作

<img src="https://cn.vuejs.org/images/data.png" alt="响应式更新原理图" style="zoom: 50%;" />

## **生命周期**

本节需要掌握生命周期的整个流程、包括各个声明周期的前后各做什么事情、AST树、虚拟dom、vue2/vue3分别如何处理响应式数据、异步更新过程和优点，以及几个重要的阶段：首次可调用data methods..的位置？哪里生成的AST树？在哪里生成的render函数？在哪里执行的render函数？哪里生成的vnode（虚拟dom）？哪里进行的diff算法？在beforeMount阶段获取refs结果是什么？首次可通过$refs获取模板引用在哪？在beforeUpdate阶段获取$refs结果是什么？nextTick是什么？组件在首次加载时会触发哪些钩子、在运行阶段时会触发哪些钩子？

|                      vue声明周期流程图                       |
| :----------------------------------------------------------: |
| 创建vue根实例（vue2：new Vue()，vue3为Vue.createApp()）<br />初始化events和生命周期（vue2：events包括$on、$off、$emit、$listener） |
|                         beforeCreate                         |
| 初始化注入和响应式（data、ref、reactive响应式处理）<br/>（vue2：provide、inject、props、data、computed、watch、methods，vue3：ref、reactive..）<br /> |
|         created（第一次可以使用data、methods..数据）         |
| 判断有没有el选项作为挂载点， 没有就使用$mount指定的元素作为挂载点， <br /> if（render）{  执行下一个阶段  }  <br />   else if（template）{  将template编译成render函数，执行下一个阶段  } <br />else {  把挂载点的内容作为template编译成render函数，执行下一个阶段  }<br />（模板语法/template -> 抽象语法树/AST -> render函数） |
|     beforemount（此时的$refs还是个空的、vnode挂载前的）      |
| 调用render函数，生成虚拟dom，之前的挂载点el元素将被新生成的$el替代掉，$refs就是在这一步被解析出来的 |
|       mounted（第一次可以操作dom、获取$refs模板引用）        |
|              响应式数据发生变化触发beforeUpdate              |
| beforeUpdate（还可拿到更新前的dom/数据，也可在这里继续对数据修改) |
| 重新触发render函数生成新的vnode、与老的vnode进行diff算法比较且更新到真实dom上。vue采用异步更新，开启一个队列、在这个队列里计算所有数据变更后、再更新dom，从而避免了不必要的dom操作 |
|                     更新完成触发updated                      |
|             组件卸载或离开页面触发beforeUnmount              |
| beforeUnmount（实例功能完全正常，应当在卸载前移除手动添加的监听器，如clearTimeout、clearInterval等清除操作） |
|           卸载完成后data、methods..都不可再访问了            |
|                          unMounted                           |

一些细节：

- vue3的生命周期函数的几点小变化：
  - 所有的生命周期现在在setup里面都是用onX导入后使用（因为支持webpack5的tree-shaking）
  - beforeDestroy改名为beforeUnmount，destroyed改名为unmounted
  - 没有onBeforeCreate、onCreated这两个钩子了，因为setup就是初始化过程，即以前需要写在onBeforeCreate、onCreated钩子里的操作，现在写在setup就好了
- 获取$refs模板引用最早是在mounted，因为$refs是挂载后的产物，在mounted才挂载完成，在那之前获取$refs是空的、没有意义的，而对于运行阶段的beforeUpdate获取$refs表示的是更新前的旧的数据
- nextTick在dom挂载完成或dom更新完成后立即调用（同样类似还有watch、watchEffect的{flush:post}）
- `<template>`到页面渲染的过程：模板语法（即`<template>`标签里的东西）-> 抽象语法树（AST树、Abstract Syntax Tree 抽象语法树）-> 渲染函数（h函数） -> 虚拟节点（vnode，在这里进行`diff`算法） -> 渲染成界面（html真实DOM）
- 需要掌握如何实现响应式更新、异步更新（看上节的内容）
- 生命周期单词的过去式、非过去式是有含义的。表格里有底纹的为生命周期阶段、无底纹的为vue默认行为

<img src="https://cn.vuejs.org/images/lifecycle.png" alt="Vue2生命周期图示" style="zoom: 33%;" />

<img src="https://v3.cn.vuejs.org/images/lifecycle.svg" alt="vue3生命周期图示" style="zoom: 67%;" />

## **虚拟dom、key、diff算法**

- **虚拟`dom`：**

  - **特点：**
    - 将真实的dom节点用js对象的形式表示，当节点改变时，不是直接修改真实dom，而是创建一个新的`vnode`与旧的`vnode`进行`diff`算法比较后，将比较的结果映射到真实的dom上去，从而提高更新的效率。（因为操作真实dom消耗性能，它上面会有很多原生的property和events）
    - 对于页面首次加载，vnode生成过程是：模板语法/template -> 抽象语法树/AST -> render函数 -> 虚拟dom。render函数执行结果的产物就是vnode，这一步发生在beforeMount和mounted之间。而在运行阶段，若响应式数据发生变化，会重新触发render函数生成新的vnode、与老的vnode进行diff算法比较且更新到真实dom上，这一步发生在beforeUpdate和updated之间
  - **好处**：
    - 计算变更、去重操作，减少不必要的dom操作
  - **坏处**：
    - 页面首次加载时由于需要生成vnode因此会增加首次渲染时间
    - 无法保留临时状态，如input框输入值

- **key：**

  - **作用：**`diff`算法比较新旧`vnode`时的身份标识
  - **使用：**`<template>`标签支持key，且当同v-for一起使用时，v-for、key都要写在`<template>`标签上，vue会自动为v-if提供key

- **`diff`算法**： 数据改变 -> 新旧`vnode`（计算变更） -> 操作真实DOM  -> 视图更新

  ​                       patch：复用或更新  |  复用：类型相同、值相同  |  更新：类型相同、值不同

  - Vue在更新节点首先会比较新旧节点数组的长度，遍历长度短的那个数组，接下来就是分成两种情况：

    - **没有key：**使用的是就地更新的策略，新、旧vnode进行patch，如果两者类型不同，即不可复用，于是删除旧vnode并将新vnode后映射到真实dom上，如果两者类型相同但值不同，就复用旧vnode，并且更新旧vnode为新的值后映射到真实dom上，依次类推
    
    - **有key的：**先从短的数组头部开始遍历，判断新旧vnode的类型和key值是否一样，是的话就patch上，不是的话就跳出循环从尾部遍历，同样进行判断新旧vnode的类型和key值是否一样，直到遇到没patch上的就跳出循环。到这一步后再分3种情况：
    
      (1) 如果旧vnode全patch上并且有新vnode未patch的话，则根据新vnode创建新节点
    
      (2) 如果新vnode全patch上并且有旧vnode未patch的话，则卸载旧节点
    
      (3) 如果以上两种情况都未符合，即新、旧vnode数组里都有未patch上，这时候就把新vnode存进map里，以新vnode的key作为map里元素的键，然后与旧vnode进行映射比较，key值映射上的就patch成功，到最后剩余的旧节点卸载掉，新节点创建出来。到此diff算法更新完成
    

```js
//无key，直接patch，patch里面会比较类型和值
const patchUnkeyedChildren=()=>{ .. patch(n1,n2,...) .. }
          
//有key，会先比较类型和key是否相等，是的话才进行patch
const patchKeyedChildren=()=>{if(..isSameVNodeType(n1, n2)){ patch(n1,n2,...)}..}

function isSameVNodeType(n1: VNode, n2: VNode): boolean {
  //...
  return n1.type === n2.type && n1.key === n2.key
}
```

## **`vue3`新增**

本节需要掌握vue3中新增的特性，以及这些特性的特点：

- **用`Proxy`替代`Object.defineProperty`实现引用数据类型的响应式处理**

  Object.defineProperty针对对象的某个属性，而Proxy针对整个对象，可以检测到对象属性的添加或删除。且Proxy不需要对数组的方法进行重载、省去很多hack

- **`Composition API/setup`**

  代码逻辑更集中、不会像vue2那样分散，适合抽离、分装、复用

- **支持 `tree-shaking`**

  vue2中定义在vue实例上的api无论使不使用，最后都会打包进产物中，因为它们都是全局上的一个属性、无法拆开，造成了体积浪费，vue3支持了webpack的tree-shaking，所有的api都需导入后使用，未导入、导入未使用的都会在webpack打包时被去除掉，从而减小了打包后的体积

- **对typescript更好的支持**

- **新增Fragment节点**

  - 支持多个根节点，多个根节点时，vue3会创建一个`<Fragment>`对其进行包裹

- **Teleport**

## **移除`vue2`**

- vue3移除了`.sync`，用`v-model:`替代
- vue3移除了`.native`修饰符，未在emits/defineEmits中定义的事件监听器都会被认为是原生事件监听器加在子组件根节点上，除非设置子组件{inheritAttrs:false}
- vue3移除了`$listeners`，事件监听器移到了`$attrs`对象里

- vue3移除了`$children `
- vue3移除了`$on、$off、$once`，因此`eventBus`也不可使用了
- vue3移除了`$scopedSlots`，作用域插槽数据移到了`$slots`里

# **vue2与vue3的重要Api**

本节需要完全掌握vue2、vue3中重要的api的使用、原理、配合适量的源码，vue3的`ref、reactive、computed、watch、watchEffect`如何使用和特点、vue2/vue3中`computed`区别、vue3中`watch、watchEffect`区别

## **vue2**

- **`data`：**
  
  - vue在初始化时会为data中的所有数据项绑定getter、setter函数，即进行响应式处理
  - vue使用`$`前缀通过组件实例暴露自己的内置API，以及为内部property保留`_`前缀。因此在data中定义的变量名不可以`$`、`_`开头（否则报错）
  
- **`computed`：**
  
  - 依赖其他属性计算值，且对结果进行缓存，当所依赖的数据项发生改变时才会重新计算结果，否则直接返回上一次缓存的计算结果
  
  - 默认为getter函数，也可写成getter、setter形式，（不能写成箭头函数）
  
    ```js
    computed: {
        a: function () { return xx },   //getter
        b: {    //getter、setter
          get: ,
          set: function (v) {}
        }
    }
    ```
  
- **`watch`：**
  
  - 监听数据项的改变，有以下几种写法：
  
    ```js
    watch: {
        a: ()=>{},   //如果不需要使用this，可用此写法，但如果需要用this就一定不能使用箭头函数
        b: function (newVal, oldVal) {},   //watch的函数写法（还有对象写法）
        c: 'someMethod',   //在methods选项里定义方法
        d: {
          handler: function (newVal, oldVal),
          deep: true    //在vue2里想为watch设置选项参数时需把watch写成对象形式
          immediate: true
        },
        e: [             //监听e的改变，并执行多个回调函数
          'handle1',               
          function handle2 (newVal, oldVal) {},
          {
            handler: function handle3 (newVal, oldVal) {},
          }
        ],
        'e.f': function (val, oldVal) {}   //只监听对象里的某个属性
    }
    //也可以通过api调用
    var stop = this.$watch('a', cb)
    stop()    //取消监听
    ```
  
- **`methods`：**
  
  - computed、watch、methods，都不可用箭头函数定义（最好都不要，否则无法把this绑定到vue实例）
  - vue2源码里会遍历methods里的所有方法，调用`bind`把它们的`this`绑定到vue实例上去，因此不要使用箭头函数，因为这会导致methods方法的this无法指向vue实例（箭头函数没有this，无法改变this指向）

**computed、methods的区别：**多次调用computed，若所依赖的数据项未改变，会直接返回上一次缓存的计算结果，而methods多次调用多次执行

**computed、watch的区别：**computed只能同步，而watch是异步

## **vue3**

vue3支持webpack5的tree-shaking，因此现在所有api都需要导入后再使用：

- `ref`：

  - 常用：`ref()、isRef()、unRef()、toRef()、toRefs()、shallowRef()`

    注：`toRef()、toRefs()`用在解构时仍希望数据为响应式时使用

- `reactive`：

  - 常用：`reactive()、isReactive()、shallowReactive()、isProxy()、toRaw()`

- `readonly`：只读，不可修改

- `computed`：

  ```js
  //写法1：接受一个getter函数，并根据getter的返回值返回一个不可变的响应式ref对象
  const a = computed(()=>{})
  //写法2：接受一个具有get、set函数的对象，用来创建可写的ref对象
  const b = computed({
   get: () =>{},
   set: (newValue) => {},
  })
  ```
  
- `watch`：

  ```js
  const obj = reactive( {a:{b:1}} )
  const e = ref('e')
  const f = ref('f')
  watch( obj, (newValue,oldValue) => {})
  watch( ()=>obj.a.b, (newValue,oldValue) => {}) //监听对象里的某个属性，必须用getter返回，即必须写成()=>obj.a.b的形式（直接写成obj.a.b会报错）
  watch( ()=>_.cloneDeep(obj), (newValue,oldValue) => {}) //深拷贝过后,监听的newValue,oldValue才会是前后值不一样,否则newValue,oldValue打印值一样
  watch( [e,f], (newValue,oldValue) => {})//vue3新增的写法，可同时监听多个ref数据，写成数组形式
  const stop = watch(obj,()=>{}) 
  stop()  //停止监听
  ```

- `watchEffect`：立即执行传入的一个函数，同时响应式追踪其依赖，并在其依赖变更时重新运行该函数（写在watchEffect里的数据会被收集为其依赖，当这些依赖改变时才会触发watchEffect）

  ```js
  const stop = watchEffect(() =>{},{flush:'post'})   //对写在回调函数里的所有数据监听
  stop()  //停止监听
  ```

**{ flush: pre/post/sync }：**

- flush是watch、watchEffect都有的参数，flush表示侦听器的执行时机，各个flush取值及表示：
  - flush:pre，默认为pre，打印的是挂载前或者更新前的数据

  - flush:post，打印的是挂载后或更新后的数据

  - flush:sync，watch是异步的，flush:pre/post都是打印一次最新值，如果设置flush:sync，在同一个函数里改变多次值，每次都会同步监听并打印出来（不建议使用）

-  watchEffect()的flush默认为pre，即在DOM挂载或更新之前运行副作用，所以当侦听器运行时，模板引用还未被更新。因此，使用模板引用的侦听器应该用 `flush: 'post'` 选项来定义，这将在 DOM 挂载或更新之后运行副作用，确保模板引用与 DOM 保持同步，并引用正确的元素
- 上面讨论的都是watch需要拿模板引用，才需要设置`flush: 'post'`以确保正确， 而如果watch只是监听ref、reactive数据的话，无论设置pre还是post，监听到的数据都是赋值后的数据（因为异步更新机制，打印的都是最终值，除非使用深拷贝返回才能打印出oldValue）

**`vue3的watch、watchEffect`区别：**

- 相同点：

  - 都可监听多个数据（watch监听多个数据时写成数组形式）
  - 都有flush参数
  - 都可以调用返回值函数来停止监听
  - 都可使用`onInvalidate`会作为回调的第三个参数传入来清除副作用
  
- 不同点：
  - `watch`惰性地执行副作用，即组件初始化阶段`watch`不执行（除非设置`immediate:true`）`watchEffect`在初始化阶段就执行监听
  - `watch`可以监听前后值，而`watchEffect`不可

**vue2、vue3里的watch：**

- vue3里的watch表现形式和vue2里的watch大致一致，不管vue2还是vue3监听的数据若为引用数据类型时，newValue和oldValue是一样的，都是返回最新值，如果希望拿到oldValue，应该对监听的数据进行深拷贝

- vue3里如果监听的是对象里的某一个属性(而不是整个对象)，不能直接写成obj.xx而应该写成()=>obj.xx，否则控制台会报错且显示obj.xx需要用getter函数返回

- vue3的选项式api里数组现在和对象一样需要设置deep:true才可以监听数组的改变，不过vue3的组合式api里数组用reactive包裹后，监听该数组，默认是深度监听（源码里reactive默认为深度监听）

- 监听ref、reactive包裹的引用数据类型，都是默认深层监听的（即不需要设置deep：true），但是需要注意ref如果包裹了引用数据类型，此时监听应是ref.value而不是ref（因为ref包裹引用数据类型时，是把该数据用reactive包裹后再赋值给value）

  ```js
  // immediate为true、false
  // {immediate: false}相当于不打印“undefined到赋初始值”（值的初始化、不是vue的初始化过程）
  const a = ref(0) 
  watch( a,
         (newV, oldV) => {console.log('a:', newV, oldV)},
         //{immediate: false} -> 打印a: 1 0，默认为false
         //{immediate: true}  -> 先打印a: 0 undefined 再打印 a: 1 0，共打印两次
   )
  a.value ++   //注意此行代码与watch的顺序会打印的结果是不同的
  ```

# **基础知识点**

## **`v-bind`**

- vue中的trusy、falsy：
  - vue2识别成falsy的只有`null undefined false`这3种
  - vue3的falsy除了不包括空字符串，即''或""，其余的js中的falsy都包括
- 自定义attribute与v-bind='obj'里的obj的属性值冲突时，会发生覆盖行为

## **`v-model`**

- **vue2的`v-model`：**

  - 在原生`<input>`标签上使用`v-model='xx'`，vue在内部会为不同的type使用不同的property并抛出不同的事件，如：

    - `type='text/textarea'`，相当于`v-bind:value='xx'和v-on:input='$event=value'`

    - `type='radio/checkbox'`，相当于`v-bind:checked='xx'和v-on:change='$event=checked'`

      注：不设置type值时默认为text，v-model相当于v-bind、v-on语法糖

  - 在自定义子组件上使用`v-model='xx'`，相当于传入给子组件内部的`<input>`标签对应的prop和监听该`<input>`标签对应的事件（子组件内部可通过emit()发送事件改变prop值）

- **vue3的`v-model`：**

  - property统一为modelValue，事件监听器统一为update:modelValue（注意在标签上要写小驼峰）
  - 移除了v-bind:xx.sync = ''的.sync修饰符，用v-model:xx='yy'替代（vue2的.sync用来prop双向绑定）

|      | 写法                                              | props      | 事件监听器        |
| ---- | ------------------------------------------------- | ---------- | ----------------- |
| vue2 | v-model = ''                                      | value      | input             |
|      | v-bind:xx.sync = ''                               | xx         | update:xx         |
| vue3 | v-model = ''                                      | modelValue | update:modelValue |
|      | v-model:xx = ''（移除.sync修饰符，用v-model替代） | xx         | update:xx         |

## **动态样式绑定 `class、style`**

- 动态绑定样式有class、style两种方法，且都支持值为字符串、对象、数组的写法
- 在自定义子组件上设置class、style，vue2会将其挂载在子组件根节点上，且不可手动指定挂载点。而在vue3里class、style会被存进$attrs对象里，且可手动指定挂载点，在子组件设置`{inheritAttrs:false}`来取消默认挂载根节点，以及在希望挂载的位置标签上设置$attrs.xx

## **条件渲染 `v-if、v-show`**

- `v-show`：
  - 只是在有无display:none这个style属性之间切换。无论初始条件是什么都会被渲染出来，后面只需要切换 css，DOM还是一直保留着的，所以v-show在初始渲染时有更高的开销，但是切换开销很小，更适合于频繁切换的场景
- `v-if`：
  - 惰性渲染机制，值为true时组件才被渲染，切换条件时会触发销毁/挂载组件，因此切换时开销更高，更适合不经常切换的场景
  - vue3会自动为v-if分配key，因此可不自己指定

注：两者都会引起重排重绘

## **列表渲染 `v-for`**

本节需要掌握v-for列表渲染为什么需要提供key/提供key有什么好处、vue2、vue3中v-if、v-for优先级比较

- `v-for`列表渲染为什么需要提供key：
  
  - **不使用key时：**vue使用“就地更新”的策略。如果数据项的顺序被改变，Vue将不会移动DOM元素来匹配数据项的顺序，而是就地更新每个元素。这个默认的模式是高效的，但是只适用于不依赖子组件状态或临时DOM状态 (例如：表单输入值) 的列表渲染输出，常见的渲染出错情况有：
  
    - v-for渲染多个input输入框，且每个输入框都有对应的value值，当改变input框的渲染顺序/位置时，value会维持修改前的顺序，而不是与对应的input框顺序一致（因为value是临时dom状态）
    - 嵌套路由的子路由之间跳转
  
    解决上面的例子由于节点复用导致的渲染出错的方法就是，为这些元素各自绑定唯一的key值
  
  - **使用key时：**提供key使vue可以跟踪每个节点的身份，从而重新排序和复用现有元素，且注意使用基本数据类型作为key值，而不要使用对象或数组之类的（否则报错）
  
- `v-if、v-for`优先级：

  - 数组里的元素判断条件为同一个时，v-if应放在v-for的外层

  - 数组里的元素部分渲染，v-for遍历后再v-if判断，但在vue2、vue3由于优先级不同，写法不一样：

    - vue2：v-for > v-if，可写在同一层级，所有的元素都被v-for遍历出来后再进行v-if判断

    - vue3：v-if > v-for，不可写在同一层级，因为v-if没办法访问到v-for遍历出来的元素，因此需要用<`template`>包裹v-for放到v-if的外层

      注：数组里的元素部分渲染更推荐的做法是先在computed中对数据进行筛选后，在用v-for遍历出来

## **修饰符**

本节需要掌握事件修饰符、按键修饰符、系统修饰符、v-bind、v-on、v-model修饰符

- **事件修饰符**

  - **.stop**     
    - 阻止冒泡

  - **.prevent**     
    - 阻止默认事件，如a点击后的跳转、form提交表单后的刷新

  - **.capture**    
    - 设置事件为捕获机制，即先触发父组件再触发子组件的事件

  - **.self**    
    - 目标为自身时才触发事件，即不被冒泡机制、也不被捕获机制触发

  - **.once**

  ```html
  //分析以下设置capture、self、stop时的情况
  <button @click="click_father">
       father
       <button @click="click_son"> son </button>
  </button>
  
  click_father() {
        console.log('msg from father !')
  },
  click_son() {
      console.log('msg from son !')
  },
  ```

- **按键修饰符**：略

- **系统修饰符**：略

- **v-bind修饰符**

  - **.sync**   
    - 在vue2里可用.sync修饰符使子组件可以修改父传过来的prop值，但在vue3里已废除.sync用v-model替代

- **v-on修饰符**

  - **.native**
    - 在vue2，只有添加了.native修饰符的事件监听器会添加在子组件根节点上，未添加.native修饰符的事件监听器会移到$listeners对象里
    - 在vue3，移除了.native修饰符和$listeners对象，在子组件的根节点上添加的事件监听器若未在子组件的emits/defineEmits中定义的话，默认都会添加在子组件根节点上，且以onX形式存进$attrs对象里（除非设置{inheritAttrs:false}）。因此如果希望监听子组件内部的原生事件，需要在子组件的emits/defineEmits中定义，否则会触发两次（原生事件一次和emit手动触发一次）

- **v-model修饰符**

  - **.lazy**  
    -  监听change事件而不是input事件（input是输入状态下都会触发事件，而change是在blur或enter时候才会触发事件）

  - **.number**   
    - 输入纯数字时转成Number，输入非纯数字时转为字符串

  - **.trim**     
    - 去除首位空格，中间不会去除（因为使用的是原生js.trim方法）

  ```html
  <input v-model.number="num_1" />  //输入纯数值时会转成Number格式（仍可以输入非数值）
  <input v-model="num_2" type="number" /> //输入框为数值选择键，只能输入数值（但仍存为字符串）
  watch: {
      num_1: (newValue) => {
        console.log('num_1:', newValue)
      },
      num_2: (newValue) => {
        console.log('num_2:', newValue)
      },
  },
  ```

## **插槽**

本节需要掌握插槽定义、插槽缩写、默认插槽/具名插槽/作用域插槽、插槽的场景

- **定义：**子组件用`<slot>`占位且可为该插槽提供备用内容，父组件负责分发`<slot>`里的内容。利用插槽可实现组件复用

- **缩写：**#缩写。默认插槽v-slot='default'可以缩写为v-slot，但是#default不可以缩写为#

- **分类：**
  - **默认插槽：**可用`v-slot='default'、v-slot、#default`表示，或者直接不写就会被当做默认插槽
  - **具名插槽：**父里的`v-slot='xx'或#xx `与子里的`<slot name='xx'>`对应，xx为自定义槽名
  - **作用域插槽：**父里的插槽内容数据来自于子组件里的数据，子里的用`v-slot:xx='yy'`设置（yy为传到父组件的数据）

- **场景：**`<router-link>、<router-view>`都使用了作用域插槽

## **非`props`的`attrs`**

本节需要掌握，什么是非props的attrs？vue2、vue3里的class、style挂载在子组件的位置有何区别？vue3里移除了$listeners后事件监听器挂载在哪里？vue2、vue3里的attrs分别有什么？如何指定$attrs的挂载位置？

- **非`props`的`attrs`：**非props、非emits定义中的attributes

- **`class、style`：**vue2里只能挂载在子组件根节点上不能改变位置，而vue3里，class、style已经移到$attrs对象里，因此可设置{inheritAttrs:false}并调用$attrs.xx手动为其指定挂载位置

- **`$listeners`：**vue3里移除了$listeners，vue2的$listeners中的事件监听器为父作用域中的不含.native修饰符的v-on事件监听器，它可以通过v-on="$listeners"传入内部组件。vue3中的$attrs中的事件监听器为未在子组件的defineEmits里定义的自定义事件监听器，它们在$attrs以onX形式存在（且挂在子组件根节点上）

- **`$attrs`：**在vue2里包括静态绑定值或v-bind动态绑定值，在vue3里包括静态绑定值或v-bind动态绑定值、class、style、未在defineEmits中定义的事件监听器

- **如何指定`$attrs`挂载点：**自定义子组件上的数据默认都挂载在子组件的根节点上，可设置{inheritAttrs: false}设置不挂载子组件根节点上，然后通过$attrs.xx指定具体的挂载位置，但是如果有多个根节点，设置了{inheritAttrs: false}却不指定具体挂载点，就会报错

  | vue2                                                  | vue3                                                         |
  | ----------------------------------------------------- | ------------------------------------------------------------ |
  | 有.native -> 加到子组件根节点                         | 不在defineEmits定义 -> 加在子组件根节点，且以onX形式存进$attrs |
  | 无.native -> 不会加在子根节点、且存在$listeners对象里 | 在defineEmits里定义                                          |

## **组件间通信**

本节掌握组件间通信所有方法，包括props/emit、provide/inject、$root、$parent、ref、vuex、eventBus

- **`props/emit`：**
  - **`props`：**可为父传给子的props数据进行`type、require、validator`验证，或设置默认值`default`
  - **`emits`：**可在emits对触发的事件进行验证，在vue3强烈建议使用emits记录每个子组件所触发的所有事件，这尤为重要，因为vue3移除了.native事件修饰符，所有未在emits定义的事件都会被存入组件的$attrs并绑定在组件的根节点上（如果子组件触发了与原生事件同名的事件，就会导致触发两次）
- **`provide/inject`：**略
- **`$root、$parent`**（$children在vue3已被移除）
- **`ref`：**模板引用
- **`vuex`：**看vuex章节
- **`$on、$off`**：eventBus在vue3已移除

## **内置组件、动态组件、异步组件**

本节需要掌握，动态组件、异步组件的定义？用法？场景？动态组件的keep-alive及activated、deactivated钩子？

- **内置组件**：

  `<slot>、<keep-alive>、<transition>、<transition-group>、<teleport>` 

  `<router-link>、<router-view>`

- **动态组件**：（不分包，打包进父组件js文件里）

  ```html
  <component :is="currentTabComponent"></component>
  ```

- **异步组件**：（分包，可使用魔法注释`/*...*/`来定义分包的名称）

  - vue2：

    ```js
    //写法1
    const asyncModal = () => import('./Modal.vue')
    //写法2
    const asyncModal = {
      component: () => import('./Modal.vue'),
      delay: 200,
      timeout: 3000,
      error: ErrorComponent,
      loading: LoadingComponent
    }
    ```

  - vue3：

    ```js
    import { defineAsyncComponent } from 'vue'
    import ErrorComponent from './components/ErrorComponent.vue'
    import LoadingComponent from './components/LoadingComponent.vue'
    
    //写法1：不带选项的异步组件
    const asyncModal = defineAsyncComponent(() => import('./Modal.vue'))
    
    //写法2：带选项的异步组件
    const asyncModalWithOptions = defineAsyncComponent({
      loader: () => import('./Modal.vue'),  //component改为loader
      delay: 200,
      timeout: 3000,
      errorComponent: ErrorComponent,
      loadingComponent: LoadingComponent
    })
    ```

- **keep-alive：**

  - include、exclude、max
  - activated、deactivated钩子：略

## **自定义指令**

本节需要掌握，如何使用自定义指令？自定义指令的名称、参数、修饰符、值？自定义指令生命钩子函数？自定义指令使用场景？

- 定义：提供操作`dom`的一种途径，且可封装、复用
- 含义：`v-aa:bb.cc='dd'`，`aa`为自定义指令名称、`bb`为参数、`cc`为修饰符、`dd`为值
- 钩子函数：略
- 场景：略

## **过渡 & 动画**

- 过渡： v-enter-from v-enter-to v-leave-from v-leave-to

## **else**

- vue2在初始化阶段（beforeCreate-created/setup之间）的data computed..数据才是响应式的, 之后无法添加根级别响应式数据,除非使用Vue.set()（vue2才有Vue.set()，vue3在setup里无法访问vue实例）

- v-for可遍历数组、对象、数值：

  ```html
  // v-for遍历数组
  <div v-for="(value,index) in arr"></div> 
  // v-for遍历对象
  <div v-for="(value, key, index) in obj"></div> 
  // v-for遍历数值
  <div v-for="value in 5"></div>
  ```

- 区分概念：实例、节点、this、el、$el

- 区分概念：自定义事件、原生事件

- 区分$event、$event.target、$event.currentTarget

  ```js
  document.getElementById('father').addEventListener('click', function (e) {
    console.log('father!', e.target, e.currentTarget)
  })
  document.getElementById('son').addEventListener('click', function (e) {
    console.log('son!', e.target, e.currentTarget) 
  })
  //点击父
  打印father,father
  //点击子
  打印son,son
  打印son,father
  //e.currentTarget打印的是绑定了事件监听器的元素
  //e.target打印的是当前触发的元素
  ```

- data、computed可以访问prop的值，反过来则不行，因为在源码里是先initProp再initData、initComputed的

- 区分哪些是响应式数据，哪些不是响应式数据

- 自定义组件里的emit事件触发如果携带参数，就会覆盖掉$event

- vue2的SFC文件里的name选项有何作用：动态组件keep-alive、vue-devtool、递归组件

- 因为setup是围绕beforeCreate和created生命周期钩子运行的，所以不需要显式地定义它们，即在这2个钩子里编写的任何代码都应该直接在setup函数中编写

- 发生网络请求且不需要操作`dom`的话，应该把网络请求（异步）放在created(不能是beforeCreate,因为最早能获取到data methods..的是在created)

  如果需要操作dom，就放在mounted（如echarts）

- ref本质也是reactive，ref(obj)等价于reactive({value: obj})

- 因为 setup 是围绕 beforeCreate 和 created 生命周期钩子运行的，所以不需要显式地定义它们。换句话说，在这些钩子中编写的任何代码都应该直接在 setup 函数中编写

- 怎么里理解“Create app.$el andappend it to el”这句话呢，就是vue会先在内存中创建编译好的模板，然后在挂载到页面html上去

- 不能在defineProperty get/set里直接return obj.a，因为会无线循环调用下去直到栈溢出

- 前后端路由区别？

- 以路由守卫举例，不止是回答基本的知识，还要回答是否在项目中用过，如在做登录验证时在beforeEach里判断是否authorization，有的话就跳转成功，无的话就跳转到登录页面，以及做过路由钩子与动画、过渡的结合

- vue中希望删除数组里的元素会触发响应式更新方法有：Vue.delete、splice，不可用delete xx

- 输入框的实时搜索应该怎么做优化：1、.lazy，2、防抖delay='200'

- 多个页面之间如何通信？

  websocket、localStorage、cookie

- cookie怎么增删改查

  增：document.cookie='name=test_user'

  删改查可自己封装函数来实现，本质上原理都是，读取document.cookie里的字符串，先split(';')再split('=')，以;相隔表示不同的cookie键值对，=左边是key右边是value

  查询是document.cookie，新增一条就是document.cookie='xx'（不会覆盖，而是一直追加）

  cookie会在每次http请求时自动携带上，后端通过req.headers.cookie获取到

  cookie用来表示用户身份，常用在购物、身份验证等场景

- keep-alive原理

- vue有哪些内置组件？

- 父子组件生命周期执行顺序

  父beforeCreate -> 父created -> 父beforeMount -> 子beforeCreate -> 子created -> 子beforeMount -> 子mounted -> 父mounted，这就是父子钩子执行顺序，但是有特殊情况的是，若子组件为懒加载或异步组件的话，那顺序和这个就不一样了

- 在created请求数据和在mounted有什么区别吗？

  一般数据请求最早可在初始化后即在created，但是对于一些需要操作到dom的，如请求echarts的数据，应在mounted（另一种思路是仍在created里，只是赋值数据时在nextTick函数里）

  mounted请求数据后可能会白屏后数据才一下子展示出来

- 如何执行多个并发请求？

  axios.all或Promise.all

# `vuex`

本节vuex需要掌握，state、getters、mutations、actions、module的含义、定义、调用

## state

- 全局状态

  ```js
  const count = computed(()=>store.state.count)   //使用computed为其保留响应式
  ```

## getters

从state中派生出的状态，类似于vue中的计算属性computed

- **定义：**接收`state、getters、rootState、rootGetters`参数（注意参数有顺序）、且可通过属性或方法的方式访问

  ```js
  getters: {
    doneTodosCount: (state, getters, rootState, rootGetters) => { //通过属性访问，参数有顺序
      return getters.doneTodos.length
    },
    getTodoById: (state, getters, rootState, rootGetters) => (id) => { //通过方法访问
      return state.todos.find(todo => todo.id === id)
    }
  }
  ```

- **调用：**

  ```js
  store.getters.doneTodosCount   //通过属性访问
  store.getters.getTodoById(2)   //通过方法访问
  ```

## mutations

改变state全局状态值的唯一方法，推荐使用常量替代mutation事件类型

mutation必须是同步函数：devtools工具会记录mutation的日记，捕捉到每一个mutation方法执行时的前一状态和后一状态的快照，如果mutation执行异步操作，就无法追踪到数据变化

- **定义：**接收state、payload参数，必须是同步函数

  ```js
  mutations: {
    increment (state, payload) {
      state.count++
    }
  }
  ```

- **调用：**

  ```js
  store.commit( 'xx', {params_1:xx,params_2:xx} )        //调用方法1
  store.commit({ type:'xx', params_1:xx, params_2:xx })  //调用方法2
  ```

## actions

action提交的是mutation，而不是直接变更状态，action可以包含异步操作

- **定义：**可接收context、payload参数，其中context可解构为`state、rootState、getters`、`rootGetters、commit、dispatch`

  ```js
  actions: {
    actionA ({ state,rootState,getters,rootGetters,commit,dispatch }, payload) {
      return new Promise((resolve, reject) => {
        setTimeout(() => {
          commit('someMutation')
          resolve()
        }, 1000)
      })
    }
  }
  ```

- **调用：**

  ```js
  store.dispatch('xx', {params_1:xx,params_2:xx} )
  store.dispatch({ type:'xx', params_1:xx, params_2:xx })
  ```

## module

略



# `vue-router`

本章vue-router需要掌握，动态路由及匹配语法、传参、404？嵌套路由、路由复用？命名路由、命名视图？路由的2种定义方式：声明式、编程式路由？路由元信息、props、重命名、别名？路由的2种模式：hash、history？导航守卫（全局、路由独享、组件）？导航前、后获取数据？路由过渡动效？路由滚动行为？路由懒加载、分包？

## `<router-link>`

- **`props`：**

  ```html
  <router-link  
     to=''   //可以是字符串、对象
     replace
     custom  //custom+navigate是vue3用来自定义标签（vue2用tag='xx'，不设置默认就是渲染成a标签）
     active-class=''
     exact-active-class=''    //仅匹配a时生效
  >
  ```

- **`v-slot`：**

  ```html
  //navigate为触发导航的函数
  <router-link v-slot='{ href,route,navigate,isActive,isExactActive }’>
  ```

## `<router-view>`

- **`props`：**

  ```html
  <router-view name=''>  //官网说route也是一个props，却没有给例子怎么使用，待
  ```

- **`v-slot`：**

  ```html
  <router-view v-slot='{route, Component}'>    //经常是用在路由过渡效果
  ```

## 动态路由、匹配语法、传参、404

- 动态路由、匹配语法、传参：

  - **`:` 冒号：**在router.js路由表里用 : 冒号 设置动态路由参数，且可设置动态路由参数的匹配规则为正则表达式、可选参数（?）、可重复参数（+、*）

  - **`path、name`：**当用path匹配路由时，需要在url上提供动态路由参数（否则报错）

    ​                          当用name匹配路由时，需要在params参数里提供动态路由参数（否则报错）

    ​                          name优先级高于path，即匹配时2个都提供的话，会忽略path而用name去匹配路由

  - **`query、params`：**在vue文件里可通过route.params获取到动态路由参数

  ​                                       传递参数时，query会在url上显示，params则不会

  ​                                       path匹配路由时，params参数会被忽略（控制台会有这句的提示）

- 404：vue2里的404匹配需要放在路由表的最后一项，vue3不需要

## 嵌套路由、复用

- 嵌套路由为在父路由的children属性（数组）里设置子路由，注意子路由的path不能以'/'开头
- 使用带有参数的路由时需要注意的是，当用户从/users/johnny导航到/users/jolyne时，相同的组件实例将被重复使用。因为两个路由都渲染同个组件，比起销毁再创建，复用则显得更加高效。不过，这也意味着组件的生命周期钩子不会被调用。若不希望复用组件，可设置不同的routerKey

## 命名路由、命名视图

- 命名路由：在定义路由时，使用name

  ​        特点：匹配路由时name的优先级高于path

- 命名视图：`<router-view name=''>`使用name参数，与路由表里的name对应

  ​        特点：用在嵌套路由里，期望子组件渲染到对应位置（即对应的`<router-view name=''>`里）

## 声明式、编程式路由

略

## meta、props、redirect 、alias 

- meta 路由元信息：在路由里设置meta参数后，便可在页面通过route.meta、或导航守卫里通过to.meta获取
- props：在路由里设置{props:true}后，在vue文件里就可以像父子通信里的props那样使用
- redirect 重定向：redirect的值可以是字符串、命名路由、方法
- alias 别名：本质上就是url的一部分

## hash、history

- hash：
  - 基于浏览器的window.location对象，url带有#，#后的内容并不作为url的一部分向服务器发送请求，因此不需要额外配置
- history：
  - 基于浏览器的window.history对象，路由跳转时会向后端发送当前的url，因此需要在后端配置，否则刷新页面时会有404报错
    - 解决刷新时404报错：nginx在对应server的根路由下配置`try_files $uri /index.html`

## 导航守卫、导航解析过程

- 导航守卫可分为3种：全局、路由独享、组件里

  - 全局：beforeEach、beforeResolve、afterEach（针对所有路由）

  - 路由独享：beforeEnter、beforeLeave（针对某一条路由）

  - 组件里：beforeRouteEnter、beforeRouteUpdate、beforeRouteLeave（进入到组件里时触发的守卫）

    注：组件里的导航守卫是写在vue文件里的，在vue3的setup的话，守卫名称改成了onX

- 一个完整的导航解析过程：

  1. 导航被触发
  2. 在失活的组件里调用 `beforeRouteLeave` 守卫
  3. 调用全局的 `beforeEach` 守卫
  4. 在重用的组件里调用 `beforeRouteUpdate` 守卫(2.2+)（因为在嵌套路由里可能会出现组件复用情况）
  5. 在路由配置里调用 `beforeEnter`
  6. 解析异步路由组件
  7. 在被激活的组件里调用 `beforeRouteEnter`
  8. 调用全局的 `beforeResolve` 守卫(2.5+)
  9. 导航被确认
  10. 调用全局的 `afterEach` 钩子
  11. 触发 DOM 更新
  12. 调用 `beforeRouteEnter` 守卫中传给next的回调函数，创建好的组件实例会作为回调函数的参数传入

## 获取数据

有时候，进入某个路由后，需要从服务器获取数据，我们可以通过两种方式来实现：

- **导航完成之后获取**：先完成导航，然后在接下来的组件生命周期钩子中获取数据。在数据获取期间显示“加载中”之类的指示
- **导航完成之前获取**：导航完成前，在路由进入的守卫中获取数据，在数据获取成功后执行导航

## 过渡

- 基于单个路由的过渡：略
- 基于多个路由的过渡：略

## 滚动

- ```js
  const router = createRouter({
    history: createWebHashHistory(),
    routes: [...],
    scrollBehavior (to, from, savedPosition) {
       return { top:0 }      //需要return一个期望滚动到哪里的位置信息
    }
  })
  ```

## 路由懒加载、分包

- 路由懒加载会分包，添加了魔法注释，webpack打包时就会生成以该注释命名的chunk，不注释的话会生成chunk.哈希值，不便查看

