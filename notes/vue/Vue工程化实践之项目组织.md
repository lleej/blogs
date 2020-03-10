title: Vue工程化实践之项目组织
date: 2019-04-25
tags: Vue
categories: 前端 Vue Element
layout: post

------

摘要：文本是使用Vue和Element进行前端工程化系列的第一篇文章，重点介绍：前端工程的项目组织、常用库、开发工具和提升效率的插件。对于部门的前端开发可以统一采用此种工程化的思路。

<!-- more -->

# 支撑环境

beJBPfLxr3wudVth

## Vue 环境

1. 使用最新的Vue脚手架工具`vue-cli 3.0`初始化项目
2. Vue使用`2.6`及以上版本

## 编译和打包

1. 使用`babel`进行`es6`语法的编译
2. 使用`eslint`进行语法检查
3. 使用`webpack`进行项目打包
4. 使用`travis`进行项目的集成

## 测试

1. 使用`jest`进行项目的测试
2. 使用`mock`模拟数据

## 工具和插件

1. 使用`VS Code`进行开发
2. `Debugger for Chrome`插件，在Chrome浏览器中进行调试
3. `ESLint`插件，进行代码检查
4. `Prettier`插件，进行代码格式化
5. `Vetur`插件，Vue语法高亮和提示

## 类库

推荐使用以下类库

### 前端组件

饿了吗的前端组件库`element-ui`

### Ajax

访问后台服务使用`axios`

### 



# 项目组织

为了统一项目中代码文件的组织以及对应的各种配置，提出如下项目组织方式：

```bash
├── dist                       // 构建目录 存放打包后的文件  
├── mock                       // 项目mock 模拟数据  
├── public                     // 不打包资源 通常存放 index.html 和 favicon.ico
├── src                        // 源代码
│   ├── api                    // 所有请求
│   ├── assets                 // 主题/字体/图片等静态资源
│   ├── components             // 全局 公用组件
│   ├── directives             // 全局 指令
│   ├── filters                // 全局 过滤器
│   ├── icons                  // 项目 svg icons
│   ├── lang                   // 国际化 language
│   ├── layout                 // 页面 布局框架 支持多种布局风格
│   ├── router                 // 路由
│   ├── store                  // 全局 store管理
│   ├── styles                 // 全局 样式
│   ├── utils                  // 全局 公用方法
│   ├── vendor                 // 公用 第三方库
│   ├── views                  // 页面
│   ├── App.vue                // 入口页面
│   ├── main.js                // 入口 加载组件 初始化等
│   └── permission.js          // 权限管理
├── tests                      // 测试用例
│   └── unit                   // 单元测试
│   └── e2e                    // 集成测试
├── .babel.config.js           // babel-loader 配置
├── eslintrc.js                // eslint 配置项
├── .gitignore                 // git 忽略项
├── package.json               // package.json
└── vue.config.js              // 项目 配置
```

## 页面目录(views)

使用`views`目录名作为所有页面存放的根路径(参照Vue脚手架命名)。每个页面放置在独立的子目录下

```bash
├── views
│   ├── login                   // 登录页面 目录
│   │   ├── index.vue           // 页面默认文件名
│   │   ├── components          // 页面级定制组件 目录
```

- 页面的默认文件名为`index.vue`
- `components`目录只存放页面级定制组件

按照业务模块来划分页面，页面命名要易懂并采用一致的命名规范(英文全称 / 中文全称)，**模块名称一律小写**

## 接口目录(api)

接口目录统一存放前端页面访问后台数据的请求。建议接口文件采用与页面相同的命名方式，便于后期的维护。

```bash
├── api
│   ├── login.js                // 登录业务 接口文件
```



##公用组件(components)

公用组件目录统一存放全局公用组件。

每个组件使用一个独立的子目录存放

##状态目录(store)

存放项目全局使用的状态信息，可以在页面/组件内灵活的通过`计算属性`实现状态的获取，并通过`dispatch`高效的更新状态信息。**注意：状态信息在页面刷新后会丢失，因此对于需要持久保存的状态数据应该存储到`cookie`中**

全局公用的状态信息存放在状态根目录的`index.js`中，业务模块使用的全局状态信息存放在`modules`目录中的独立文件中，便于后期的维护。

```bash
├── store
│   ├── index.js                // 公用状态 文件
│   ├── modules                 // 业务模块使用的状态 目录
│   │   ├── user.js             // 用户模块的状态 文件
```

业务模块的状态文件，建议使用命名空间

```javascript
const state = {...}
const mutations = {...}
const actions = {...}
export default {
  namespaced: true,             // 使用命名空间
  state,
  mutations,
  actions
}

// 访问方式
this.$store.user.xxx
```























