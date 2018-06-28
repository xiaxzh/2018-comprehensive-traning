## 中大活动小程序设计文档（总体）

### 架构设计

此软件是一款微信小程序软件，运行在微信客户端此平台上

整体上，软件分离为前后端两个部分，前后两端通过[API文档](https://activityplussysu.docs.apiary.io/#)进行沟通，开发过程可以相互分离，开发效率得以提高

前端使用Vue前端框架，采取MVVM(Model-View-ViewModel)的架构，在前端页面里，Model用纯JavaScript对象表示，View负责显示，两者做到了最大限度的分离。把Model和View关联起来的就是ViewModel，ViewModel负责把Model的数据同步到View中，还负责把View的修改同步回Model。所以，采用数据劫持结合发布者-订阅者模式从而实现Model、View双向绑定的Vue.js是不错的选择

后端使用Go语言作为编程语言，采取Service Oriented Architecture的设计理念，使用Restful风格的API设计。一个API为前端提供一个服务，利于随时拓展添加新的服务以适应新的需求。后端整体分为Web层、业务层以及持久化层

其中Web层主要负责接收请求，对数据做初步校验，然后封装传给业务层

业务层负责处理后台具体的业务逻辑，向上接受Web层传递下来的对象，向下获取持久化层提供的数据接口

持久化层主要负责数据的持久化，大部分是对数据库的读写操作，此处我们使用了将程序对象自动持久化到关系数据库中的ORM

至此，我们认为，对中大活动小程序的架构已经进行了整体而不粗略的解释

### 模块划分

#### 前端模块划分

Pages是微信小程序前端工作的页面划分依据，每一个页面相当于html网页的一个页面。

前端文件目录（主要目录）：

```
front-end
  - pages
  	- discussion
  	- myaccount
  	- posterwall
  - images
  - app.js               设置全局的变量，实现API函数
  - app.json             对所有页面导航的设置
  - app.wxss             对所有页面的样式的全局设置
```

1. discussion页面（讨论区）

```
- discussion
	- comments                         实现讨论区具体评论的页面
		- comments.js
		- comments.json
		- comments.wxml
		- comments.wxss
	- main				   实现展示所有讨论区帖子的页面
		- main.js
		- main.json
		- main.wxml
		- main.wxss
	- newpost                          实现发表新的帖子的页面
		- newpost.js
		- newpost.json
		- newpost.wxml
		- newpost.wxss
```

2. myaccount页面（我的）

```
- myaccount
	- main			          实现“我的”的主页面（可以连接到我的帖子，我报名的，我的消息页面）
    		- main.js	
    		- main.json
    		- main.wxml
    		- main.wxss
	- myenrollments			  实现我报名的的页面
    		- myenrollments.js
    		- myenrollments.json
    		- myenrollments.wxml
    		- myenrollments.wxss
	- mynotifications                 实现我的消息的页面
		- mynotifications.js
		- mynotifications.json
		- mynotifications.wxml
		- mynotifications.wxss
	- myposts        		  实现我的帖子的页面
		- myposts.js 
		- myposts.json
		- myposts.wxml
		- myposts.wxss
```

3. posterwall页面（海报墙）

```
- myaccount
	- main			          实现海报墙的主页面，展示所有可报名的活动
    		- main.js	
    		- main.json
    		- main.wxml
    		- main.wxss
	- details			  实现具体活动详情的页面
		- details.js
		- details.json
		- details.wxml
		- details.wxss
	- enroll                          实现报名活动的页面
		- enroll.js
		- enroll.json
		- enroll.wxml
		- enroll.wxss
	- enroll_error       	          实现报名活动后，报名失败的页面
		- enroll_error.js
		- enroll_error.json
		- enroll_error.wxml
		- enroll_error.wxss
	- enroll_success    	          实现报名活动后，报名成功的页面
		- enroll_success.js
		- enroll_success.json
		- enroll_success.wxml
		- enroll_success.wxss
```

#### 后端模块划分

Package是Golang中最基本的分发单位和工程管理中依赖关系的体现，一个目录属于一个Package

后端文件目录(主要目录):

```
service-end
  - controller
  - logs
  - middleware
  - models
  - router
```

1. controller package 处理业务逻辑的业务层模块


```
  - controller
      - handle.go       模块集中处理业务逻辑
      - service.go      模块内部的辅助函数
      - type.go         定义模块内部的结构体数据
```


2. logs 后端程序中自定义日志信息的模块

```
  - logs
    - logs.go           定义后端程序中日志信息
```

3. middleware Web中间件模块

```
  - middleware
    - auth.go           后端程序用户验证的Web中间件
```

4. models 数据持久化模块

```
  - models
    - entities
      - entities.go     定义模块内部的结构体数据
      - init.go         数据库、ORM初始化
    - service
      - service.go      模块集中处理数据持久化业务
```

5. router Web层模块

```
  - router
    - router.go         路由，分发请求至业务层
```

### 软件设计技术

#### 前端软件设计技术

1. MSSM：对逻辑和UI进行了完全隔离，这个跟当前流行的react，agular，vue有本质的区别，小程序逻辑和UI完全运行在2个独立的Webview里面，而后面这几个框架还是运行在一个webview里面的，如果你想，还是可以直接操作dom对象，进行ui渲染的。

2. 组件机制：引入组件化机制，但是不完全基于组件开发，跟vue一样大部分UI还是模板化渲染，引入组件机制能更好的规范开发模式，也更方便升级和维护。

3. 多种节制：不能同时打开超过5个窗口，打包文件不能大于1M，dom对象不能大于16000个等，这些都是为了保证更好的体验。

#### 后端软件设计技术

1. [Docker容器化](./service-end/Dockerfile)，为了能在开发者和服务期间都能准确无二意地运行起服务器，并且在服务器中实现容器的隔离（服务器中还运行了其他的服务容器）。我们将后端软件容器化

2. [TravisCI自动集成](./service-end/.travis.yml)，只要有新的代码，自动抓取并提供运行环境，执行测试，完成构建并部署到服务器
