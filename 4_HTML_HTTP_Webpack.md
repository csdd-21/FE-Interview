# html

## h5新增

- 新增选择器 document.querySelector、document.querySelectorAll
- 拖放(Drag and drop) API
- 媒体播放的 video 和 audio
- 本地存储 localStorage 和 sessionStorage
- 语意化标签 article、footer、header、nav、section
- 地理位置 Geolocation
- 全双工通信协议 websocket
- 离线存储manifest
- 历史管理 history
- 跨域资源共享(CORS) Access-Control-Allow-Origin
- 绘画 canvas

- 移除的元素：
  - 纯表现的元素：basefont、big、center、font、 s、strike、tt、u
  - 对可用性产生负面影响的元素：frame、frameset、noframes

# http

## 状态码

- **1xx，信息响应**
  - 100 Continue 继续，一般在发送post请求时，已发送了http header之后服务端将返回此信息表示确认，之后发送具体参数信息

- **2xx，成功响应**
  - 200 OK 请求成功
  - 201 Created 请求成功并且服务器创建了新的资源
  - 202 Accepted 服务器已接受请求，但尚未处理

- **3xx，重定向**
  - 301 Moved Permanently 永久重定向，被请求的资源已永久移动到新位置，并且将来任何对此资源的引用都应该使用本响应返回的若干个uri之一。因此客户端应当自动把请求的地址修改为从服务器反馈回来的地址
  - 302 Found 临时重定向，请求的资源临时从不同的uri响应请求。由于这样的重定向是临时的，客户端应当继续向原有地址发送以后的请求。只有在Cache-Control或Expires中进行了指定的情况下，这个响应才是可缓存的
  - 304 Not Modified 自从上次请求后，请求的文件未修改过。用于协商缓存时服务器验证浏览器的缓存文件仍有效后返回

- **4xx，客户端错误**
  - 400 Bad Request 服务器无法理解请求的格式，客户端不应当尝试再次使用相同的内容发起请求。或请求参数有误
  - 401 Unauthorized 请求未授权
  - 403 Forbidden 禁止访问
  - 404 Not Found 找不到如何与uri相匹配的资源

- **5xx，服务端错误**
  - 500 Internal Server Error 服务器遇到了不知道如何处理的情况
  - 503 Service Unavailable 服务器端暂时无法处理请求（可能是过载或维护）

##  报文

###   请求头（request）

```http
GET /home.html HTTP/1.1
Host: developer.mozilla.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://developer.mozilla.org/testpage.html
Connection: keep-alive
Upgrade-Insecure-Requests: 1
If-Modified-Since: Mon, 18 Jul 2016 02:36:04 GMT
If-None-Match: "c561c68d0ba92bbeb8b0fff2a9199f722e3a621a"
Cache-Control: max-age=0
...
```

###  响应头（response）

```http
200 OK
Access-Control-Allow-Origin: *
Connection: Keep-Alive
Content-Encoding: gzip
Content-Type: text/html; charset=utf-8
Date: Mon, 18 Jul 2016 16:06:00 GMT
Etag: "c561c68d0ba92bbeb8b0f612a9199f722e3a621a"
Keep-Alive: timeout=5, max=997
Last-Modified: Mon, 18 Jul 2016 02:36:04 GMT
Server: Apache
Set-Cookie: mykey=myvalue; expires=Mon, 17-Jul-2017 16:06:00 GMT; Max-Age=31449600; Path=/; secure
Transfer-Encoding: chunked
Vary: Cookie, Accept-Encoding
X-Backend-Server: developer2.webapp.scl3.mozilla.com
X-Cache-Info: not cacheable; meta data too large
X-kuma-revision: 1085259
x-frame-options: DENY
...
```

# 浏览器

## 存储机制

|        | cookie                                                       | sessionStorage     | localStorage           |
| ------ | ------------------------------------------------------------ | ------------------ | ---------------------- |
| 用途   | 标示用户身份而储存在用户本地上的数据（通常经过加密）<br />同源http请求时会自动携带cookie<br />浏览器与服务器之间通信 | 存储在浏览器       | 存储在浏览器           |
| 大小   | 不超过4k                                                     | 5m左右             | 5m左右                 |
| 有效期 | 过期时间取决于expires值                                      | 随标签页关闭而删除 | 长久有效，除非手动删除 |

## 缓存机制

### 按缓存位置分类

- `memory cache`：内存缓存，浏览器关闭就会清除掉，或者超过上限，就会清除掉旧的资源,几乎所有的请求资源都会进入内存缓存。常用有`<link rel="preload/prefetch/prerender">`

  - preload：用户可能需要在当前页面中加载该资源，所以浏览器必须预先获取和缓存它
  
  - prefetch：用户未来的浏览有可能需要加载目标资源，所以浏览器在网络空闲时可预先获取和缓存对应资源，优化用户体验
  
- `disk cache`：会严格根据http头信息中的字段来判定哪些资源是否可以缓存，哪些资源是是否可用的，

  它允许相同的资源在跨会话、甚至跨站点的情况下使用

- `Service Worker`：缓存位置在开发者工具Application -> Cache Storage里面，这个缓存是永久性的，即关闭tab或者浏览器，下次打开依然还在（而 memory cache 不会）。除非手动调用api删除或者容量超过限制被浏览器全部清空

  **请求网络**

  如果一个请求在上述3个位置都没有找到缓存，那么浏览器会发送网络请求去服务器获取内容。之后再把这个资源添加到缓存中去

### **按失效策略分类**

#### 强缓存（Expires、Cache-control）

强缓存的字段包含两个：`Expires和Cache-control`，区别是前者是绝对时间、后者是相对时间

```js
Expires: Thu, 10 Nov 2017 08:45:11 GMT
Cache-control: max-age=2592000
```

`Cache-control`字段常用的值：

- max-age：距离请求发起的时间的秒数
- no-store：真正意义上的“不要缓存”，所有内容都不走缓存
- no-cache：虽然字面意思是“不要缓存”，但实际上还是要求客户端缓存内容，之后再次发起请求时与服务器协商后决定是否使用这个资源
- public：可以被任何中间人缓存，包括客户端、代理服务器、CDN等
- private：默认值，所有的内容只有客户端才可以缓存，代理服务器等其它不能缓存
- must-revalidate：如果超过了max-age的时间，浏览器必须向服务器发送请求，验证资源是否还有效

#### 协商缓存（Last-Modified、Etag）

- 当强制缓存失效时，就需要使用协商缓存，由服务器决定缓存内容是否可用。浏览器会携带缓存标识去请求服务器。如果缓存未失效，则返回304状态码表示继续使用，于是客户端继续使用缓存；如果失效，则返回200状态码、新的数据和缓存规则


- 浏览器发送请求时携带的请求头为If-Modified-Since或If-None-Match。（请求头If-Modified-Since值等于首次请求资源时响应头里的Last-Modified值，请求头If-None-Match值等于首次请求资源时响应头里的Etag 值）
- Etag的优先级、精度高于Last-Modified

### 使用场景

- 不常变化的资源使用强缓存，经常变化的资源使用协商缓存   

## 渲染机制（从输入url到显示页面过程）

1. 输入url后，浏览器首先会查看对应的资源是否有缓存，如果没有缓存，则发起新请求。如果有缓存，会检查缓存文件是否仍有效

2. 浏览器先查看Cache-control: max-age=N请求头，判断资源是否仍在有效期N以内。无Cache-control属性便会去查看是否包含Expires属性，通过比较Expires的值和请求头里面Date属性的值来判断缓存是否还有效。如果max-age和expires值都没有。那就判断是否为协商缓存机制，浏览器会携带缓存标识去请求服务器。如果缓存未失效，则返回304状态码表示继续使用，于是客户端继续使用缓存；如果失效，则返回200状态码、新的数据和缓存规则。浏览器发送请求时携带的请求头为If-Modified-Since或If-None-Match。（请求头If-Modified-Since值等于首次请求资源时响应头里的Last-Modified值，请求头If-None-Match值等于首次请求资源时响应头里的Etag 值）

   注：第1、第2其实说的就是前面“缓存机制”章节的内容

3. 浏览器解析url获取协议，主机，端口，path

4. 浏览器组装一个http（get）请求报文

5. 浏览器解析域名获取ip地址顺序为：浏览器缓存 -> hosts文件 -> 路由器缓存 -> isp dns缓存 -> dns递归查询

6. 打开一个socket与目标ip地址端口建立tcp链接，三次握手如下：

   （1）客户端发送一个tcp的SYN=1，Seq=X的包到服务器端口

   （2）服务器发回SYN=1， ACK=X+1， Seq=Y的响应包

   （3）客户端发送ACK=Y+1， Seq=Z

7. tcp链接建立后发送http请求

8. 服务器检查http请求头是否包含缓存验证信息，如果缓存文件有效则返回304状态码，否则返回200状态码和最新资源

9. 服务器将响应报文通过tcp连接发送回浏览器

10. 浏览器接收http响应，然后根据情况选择关闭tcp连接或者保留重用，关闭tcp连接的四次握手为：

   （1）主动方发送Fin=1， Ack=Z， Seq= X报文

   （2）被动方发送ACK=X+1， Seq=Z报文

   （3）被动方发送Fin=1， ACK=X， Seq=Y报文

   （4）主动方发送ACK=Y， Seq=X报文

11. 如果资源可缓存，进行缓存

12. 对响应进行解码（例如gzip压缩）

13. 根据资源类型决定如何处理（假设资源为html文档）

14. 解析html文件，构建dom树，当解析器发现非阻塞资源，例如一张图片，浏览器会请求这些资源并且继续解析。当遇到一个css文件时，浏览器会请求该css文件且请求完成后同时解析，这些都不会阻塞html文档的解析。但是对于没有`async`或者`defer`属性的`<script>`标签，浏览器会停止解析和渲染HTML，而去请求并执行`<script>`里的文件。尽管浏览器的预加载扫描器加速了这个过程，但过多的脚本仍然是一个重要的瓶颈。

15. 解析css文件并构建cssom树

16. 根据dom树和cssom树构建渲染树

    （1）从dom树的根节点遍历所有可见节点（不可见节点包括`<script>、<meta>`这样本身不可见的标签、或被css隐藏的节点如`display: none`）

    （2）对每一个可见节点，找到对应的cssom规则并应用

    （3）发布可视节点的计算样式和内容

17. 解析和执行js文件：

    （1）html解析器遇到没有async或defer属性的script时，会停止解析html，而去下载和执行js文件

    （2）当遇到设置了async属性的script时，会同时下载js，并继续解析html。js下载完成后会立马执行

    （3）当遇到设置了defer属性的script时，会同时下载js，并继续解析html。js下载完成后不是立马执行，而是延迟执行js直到可以访问到完整文档树，且js的执行顺序为在文档中的出现顺序

18. 显示页面（HTML解析过程中会逐步显示页面）


## 跨域

1. **jsonp**：利用`<script src='跨域ip地址'>`，`<script>`标签不受浏览器同源策略的限制
2. **devServer的proxy属性**：在vue.config.js文件里设置devServer:{ proxy:{ } }代理
3. **后端添加允许跨域CORS响应头**：res.setHeader('Access-Control-Allow-Origin','*')，还可以设置methods、max-age等
4. **nginx代理**：proxy_pass

## 多页签通信

cookie、localstorage、websocket

## XSS/CXRF攻击及防御

待

# else

- 打开一个浏览器，会启动几个进程

# 优化

- **css优化：**
  - [ ] 多个css合并，尽量减少http请求
  - [ ] css雪碧图来减少http网络请求次数（现在用iconfont解决这个问题）
  - [ ] 将css文件放在页面最上面（script放在body）
  - [ ] 选择器优化嵌套，尽量避免层级过深
  - [ ] 充分利用css继承属性，减少代码量
  - [ ] 抽象提取公共样式，减少代码量
  - [ ] 属性值为0时，不加单位
  - [ ] 属性值为小于1的小数时，省略小数点前面的0
- **js优化：**
  - [ ] script放在html文件的<body>
  - [ ] 大对象、闭包使用完后立即赋值为null
- **vue优化：**（风格指南）
  - [ ] 按需加载
  - [ ] 异步加载
  - [ ] 懒加载
  - [ ] keep-alive
  - [ ] 分包
- **图片/大文件优化：**
  - [ ] 图片懒加载
  - [ ] 图片预加载
  - [ ] 图片预览/放大加载
  - [ ] 图片压缩优化
  - [ ] 图片、大文件切片后上传优化
  - [ ] 图片断点续传
  - [ ] 小图片转base64

- **其它：**
  - [ ] 输入框实时搜索防抖优化
  - [ ] 生产环境cdn引入优化
  - [ ] nprogress，页面跳转时的进度条提示
  - [ ] 减少http请求(雪碧图,base64,保留请求结果操作本地变量)
- **webpack打包优化：**
  - [ ] 压缩css、js  
  - [ ] happyPack
  - [ ] dll

- **nginx缓存优化：**
  - [ ] 图片、css等缓存1个月
  - [ ] gzip

# webpack

- [ ] webpack-bundle-analyzer（体积图）
- [ ] splitChunks/CommonsChunkPlugin（打包公共模块，如vue、element）
- [ ] tree-shaking（只支持es6的import、生产模式下自动开启、vue3已支持）
- [ ] happypack/thread-loader（利用多进程打包提升构建速度）
- [ ] dllplugin（预先编译资源第三方模块/动态链接库）

- [ ] sourcemap（将打包压缩后的代码映射源文件代码中的过程，方便调试回溯）
- [ ] hmr（热更新，更改代码时局部刷新）

## 分析阶段

- **webpack-bundle-analyzer**

  分析打包后文件的依赖及体积大小

- **speed-measure-webpack-plugin**

  分析整个打包的总耗时，以及每一个loader和每一个plugins构建所耗费的时间，耗时过长会红色标出

##优化阶段

**为不同环境配置不同规则**

- 在chainWebpack为"production"和"development"环境配置不同的规则。不同的环境会有不一样的配置，比如在development需要设置sourceMap，在production环境需要由于cdn引入因此需要设置externals

### 体积优化

- **splitChunks**

  SplitChunks它可以将多个Chunk中公共的部分提取出来。公共模块的提取可以为项目带来几个收益：
  开发过程中减少了重复模块打包，可以提升开发速度；
  合理分片后的代码可以更有效地利用客户端缓存（比如可以用splitChunks打包vue、react等）

- **tree-shaking**

  只支持es6的import、生产模式下自动开启。在vue2没作用，但是vue3已经更好的支持了webpack的tree-shaking，所有的api都要导入后才可以使用，未导入或导入未使用的，在经由webpack打包时都会tree-shaking

- **压缩js：uglifyjs**-webpack-plugin（webpack5为terser-webpack-plugin ）

  默认parallel:true，使用多进程并发运行以提高构建速度。 并发运行的默认数量为os.cpus().length-1

- **压缩css、抽取css**

- **压缩图片：url-loader**。使用url-loader，fallback为file-loader（默认值limit:4096），webpack会将图片或视频等文件小于4kb的转为base64打包进资源所在的js文件里，小于4kb的图片转为base64的好处在于可以减少http请求，而大于4kb的会直接拷贝到dist，可以在chainWebpack里修改limit大小值

### 速度优化

- **HappyPack/thread-loader**利用多进程打包提升构建速度

  因为Node是单线程运行的，所以Webpack在打包的过程中也是单线程的，特别是在执行Loader的时候，这样会导致等待的情况。HappyPack可以将Loader的同步执行转换为多进程并行执行，webpack官网已推出thread-loader也是多进程并行打包

- **缓存 Cache 相关，开启了cacheDirectory**，会在项目的node_modules\\.cache文件夹下生成对应的缓存文件来提升速度，默认开启的cache缓存有babel-loader、vue-loader、terser-webpack-plugin

- **预先编译资源模块（DllPlugin）/动态链接库**：我们在打包的时候，一般来说第三方模块是不会变化的，所以我们想只要在第一次打包的时候去打包一下第三方模块，并将第三方模块打包到一个特定的文件中，当第二次 webpack 进行打包的时候，就不需要去 node_modules 中去引入第三方模块，而是直接使用我们第一次打包的第三方模块的文件就行。webpack.DllPlugin 就是来解决这个问题的插件，使用它可以在第一次编译打包后就生成一份不变的代码供其他模块引用，这样下一次构建的时候就可以节省开发时编译打包的时间

### 开发中调试

- **sourceMap将打包压缩后的代码映射源文件代码中的过程，方便调试回溯**

  可用在开发和生产环境。经过Webpack打包压缩后的代码基本上已经不具备可读性，此时若代码抛出了一个错误，要想回溯它的调用栈是非常困难的。而有了source map，再加上浏览器调试工具（dev tools），要做到这一点就非常容易了。

  配置sourceMap方法：devtool: "inline-source-map"（只对js生效），还需要对应的css或js文件里设置options: {sourceMap: true}

- **HMR热更新**

  只用在开发环境。webpack-dev-server与浏览器之间维护了一个websocket，当本地资源发生变化时WDS会向浏览器推送更新事件，并带上这次构建的hash，让客户端与上一次资源进行比对。通过hash的比对可以防止冗余更新的出现。vue-loader自动的实现了hmr。需要配置的有devServer: {hot: true}以及new webpack.HotModuleReplacementPlugin()。

**其它**

首屏加载的JS资源地址是通过页面中的script标签来指定的，而间接资源（通过首屏JS再进一步加载的JS）的位置则要通过output.publicPath来指定

import函数还有一个比较重要的特性。ES6 Module中要求import必须出现在代码的顶层作用域，而Webpack的import函数则可以在任何我们希望的时候调用

首先，DllPlugin 与 DllReferencePlugin 可以用来预构建 vendor 包，这样只要一次预构建后没有额外的依赖变更，那么启动开发环境的速度就会显著提升。

所以实际上 DllPlugin 可以认为是只用于开发环境的。至于 CommonsChunkPlugin 则是用来把多个包中的公共依赖抽取为同一个 Chunk，这可以显著减小生产环境的尺寸。

关于二者区别，可以认为 DllPlugin 是用于提速开发环境构建速度的，而 CommonsChunkPlugin 则是用于优化包尺寸的。

# git

- `git pull`：相当于是从远程获取最新版本并`merge`到本地

- `git fetch`：相当于是从远程获取最新版本到本地，不会自动`merge`

