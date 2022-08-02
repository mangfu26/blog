---
layout: post
title: Steps组件使用
index_img: /images/post/primevue-cover.png
date: 2022-08-01 10:12:38
categories:
- 开发
- 前端
- PrimeVue框架
tags:
- 前端
- PrimeVue
---

# PrimeVue框架--Steps组件使用


## 1. 组件简介

> **PrimeVue**
> 
> Steps 组件是向导工作流中步骤的指示器。
> 
> https://www.primefaces.org/primevue/steps

通常称为 `步骤条`, `步进条`


按理来说，组件的使用直接参考官方文档即可，不需要记录到博客。

但是 PrimeVue 的步骤条与 Element 框架点区别，PrimeVue 的步骤条是基于 Vue Router 的，之前没有接触过，使用过程中踩了一两个坑，所以还是打算用博客的形式记录一下使用方法，供日后参考，而不是苦哈哈的再去研究官方文档💀


## 2. 使用方法

> **小提示**:
> 
> 所有代码都基于Vue3项目，所以确保您拥有一个vue3项目
>
> 如果您没有，请使用 [Vue-CLI](https://cli.vuejs.org/zh/) 创建，参考：[https://cli.vuejs.org/zh/guide/creating-a-project.html](https://cli.vuejs.org/zh/guide/creating-a-project.html)


**创建一个Vue3项目**
```sh
[mf~/project/demo] $ vue create primevue-steps
```

**选择Vue3选项**
```sh
Vue CLI v4.5.15
┌──────────────────────────────────────────┐
│                                          │
│   New version available 4.5.15 → 5.0.8   │
│     Run npm i -g @vue/cli to update!     │
│                                          │
└──────────────────────────────────────────┘

? Please pick a preset: 
  Default ([Vue 2] babel, eslint) 
❯ Default (Vue 3) ([Vue 3] babel, eslint) 
  Manually select features 
```

**等待创建完成**

```sh
Vue CLI v4.5.15
✨  Creating project in /home/mf/project/demo/primevue-steps.
🗃  Initializing git repository...
⚙️  Installing CLI plugins. This might take a while...


added 1300 packages in 1m
🚀  Invoking generators...
📦  Installing additional dependencies...


added 71 packages in 13s
⚓  Running completion hooks...

📄  Generating README.md...

🎉  Successfully created project primevue-steps.
👉  Get started with the following commands:

 $ cd primevue-steps
 $ npm run serve

 WARN  Skipped git commit due to missing username and email in git config, or failed to sign commit.
You will need to perform the initial commit yourself.
```

创建完成后，切换到项目目录(`例子中是 /home/mf/project/demo/primevue-steps`)

```sh
cd primevue-steps
```


### 2.1 安装 Vue Router

由于 steps组件 是基于 Vue Router 的，所以使用前需要安装 Vue Router

```sh
npm install vue-router@4
```

```sh
[mf~/project/demo/primevue-steps] (master) $ npm install vue-router@4

added 2 packages, and audited 1374 packages in 18s

1 package is looking for funding
  run `npm fund` for details

25 vulnerabilities (11 moderate, 7 high, 7 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details.
```

### 2.2 安装 PrimeVue 

```sh
npm install primevue@^3.15.0 --save
npm install primeicons --save
```

```sh
[mf~/project/demo/primevue-steps] (master) $ npm install primevue@^3.15.0 --save

added 2 packages, and audited 1376 packages in 17s

1 package is looking for funding
  run `npm fund` for details

25 vulnerabilities (11 moderate, 7 high, 7 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details.
[mf~/project/demo/primevue-steps] (master) $ npm install primeicons --save

up to date, audited 1376 packages in 9s

1 package is looking for funding
  run `npm fund` for details

25 vulnerabilities (11 moderate, 7 high, 7 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details.
[mf~/project/demo/primevue-steps] (master) $ 
```

### 2.2 创建测试视图(view)

在 `primevue-steps/src/` 目录下创建目录 `views`，用于存储视图（`primevue-steps/src/views`），在此目录下创建视图文件 `TestView.vue`

```html
<template>
    <h3>PrimeVue Steps 组件测试视图</h3>
</template>

<script>
    export default {
        name: 'TestView',
        components: {},
        setup() {
            
        }
    }
</script>
```

### 2.3 为项目引入路由并为测试视图创建一个路由

在 `primevue-steps/src/` 目录下创建目录 `router`，用于存储路由文件（`primevue-steps/src/router`），在此目录下创建路由文件 `index.js`

```js
import { createRouter, createWebHistory } from "vue-router";
import HelloWorld from "@/components/HelloWorld"
import TestView from "@/views/TestView"

// 两个路由，根路由(/) 和 测试视图路由(/tv)
// 根路由和默认的示例HelloWorld组件绑定，用于显示我们的首页
// 测试视图路由和我们的测试视图绑定，用于显示我们的测试视图
const routes = [
    {
        name: "index",
        path: "/",
        component: HelloWorld
    }, {
        name: "testView",
        path: "/tv",
        component: TestView
    }
]

const router = createRouter({
    history: createWebHistory(),
    routes
})

export default router;
```

路由文件创建完成，在 `primevue-steps/src/main.js` 文件中导入并使用路由

```js
import { createApp } from 'vue'
import App from './App.vue'
// 导入路由
import router from '@/router'
// 导入PrimeVue样式
import 'primeicons/primeicons.css'
import 'primevue/resources/primevue.min.css'
import 'primevue/resources/themes/saga-blue/theme.css'


createApp(App).use(router).mount('#app')
```

最后在 `primevue-steps/src/App.vue` 中注释掉默认内容，插入 `<router-view></router-view>` 标签，路由所对应的组件/视图都将渲染到此处

```html
<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png">
  <HelloWorld msg="Welcome to Your Vue.js App"/> -->
  <router-view></router-view>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    // HelloWorld
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

```

启动测试服务器

```sh
[mf~/project/demo/primevue-steps] (master) $ npm run serve

 DONE  Compiled successfully in 300ms                                                                                                                                                                                11:39:34 AM


  App running at:
  - Local:   http://localhost:8081/ 
  - Network: http://192.168.8.7:8081/


```

![](/images/post/Use-Primevue-Steps-component-vue3/1.gif)


### 2.4 在测试视图中引入Steps组件并使用

> **知识点：**
>
> PrimeVue中Steps组件的步骤组件切换使用嵌套路由实现，关系图如下：

![](/images/post/Use-Primevue-Steps-component-vue3/2.jpg)

#### 2.4.1 在测试视图中使用steps组件

```html
<template>
    <h3>PrimeVue Steps 组件测试视图</h3>
    <!-- 
        sreps组件本身 
        :model="stepsItems" 绑定步骤信息，每个步骤项为一个对象，指定步骤名称和要跳转的子路由地址
        :readonly="true" 只读步骤，步骤条不可点击，如果为false，则可以点击相应的步骤项跳转到的子路由
    -->
    <Steps :model="stepsItems" :readonly="true" />
    <!-- 
        嵌套路由渲染区, 用于渲染步骤组件
        :formData="formObject" 向子路由传递数据。可选，一般用于给子路由的组件传递初始值，如果不需要可以忽略
        @prevPage="prevPage($event)" 监听子路由prevPage事件, 用于跳转到上一页(上一步)
        @nextPage="nextPage($event)" 监听子路由nextPage事件, 用于跳转到下一页(下一步)
        @complete="complete" 监听子路由complete事件，该事件触发时表示步骤完成
    -->
    <router-view 
        v-slot="{ Component }" 
        :formData="formObject" 
        @prevPage="prevPage($event)" 
        @nextPage="nextPage($event)"
        @complete="complete($event)"
    >
        <keep-alive>
            <component :is="Component" />
        </keep-alive>
    </router-view>
</template>

<script>
import { ref } from 'vue'
// 导入router, 用于控制子路由
import { useRouter } from 'vue-router'
// 导入 steps 组件
import Steps from 'primevue/steps'

export default {
    name: 'TestView',
    components: { Steps },
    setup() {
        // 路由实例, 控制子由跳转
        const router = useRouter()
        // 步骤条信息
        const stepsItems = ref([
            { label: '步骤1', to: '/tv/p1' },
            { label: '步骤2', to: '/tv/p2' },
            { label: '步骤3', to: '/tv/p3' },
        ])
        // 数据
        const formObject = ref({
            msg: 'Steps数据'
        })

        /**
         * @function 子路由事件处理函数, 跳转到下一页(下一个步骤)
         * @param {*} data 子路由组件传递的数据
         */
        const nextPage = (data) => {
            // 这里进行子路由组件数据的处理
            // 这里为了演示只是向控制台打印传递的数据
            console.info(`Steps: ${stepsItems.value[data.value.index].label}传递的数据`, data.value)
            // 跳转到下一个步骤, 下一个步骤的路由信息由stepsItems定义
            router.push(stepsItems.value[data.value.index + 1].to);
        }

        /**
         * @function 子路由事件处理函数, 跳转到上一页(上一个步骤)
         * @param {*} data 子路由组件传递的数据
         */
        const prevPage = (data) => {
            router.push(stepsItems.value[data.value.index - 1].to);
        };

        /**
         * @function 子路由事件处理函数, 步骤完成
         * @param {*} data 子路由组件传递的数据
         */
        const complete = (data) => {
            // 这里做步骤完成后的操作, 这里为了演示只是进行了简单的提示,和打印控制台
            console.info(`Steps: ${stepsItems.value[data.value.index].label}传递的数据`, data.value)
            alert('步骤完成')
        };

        return {
            stepsItems, formObject,
            nextPage, prevPage, complete
        }
    }
}
</script>
```

访问 `/tv` ,可以看到步骤条已经创建。

![](/images/post/Use-Primevue-Steps-component-vue3/3.png)


#### 2.4.2 创建子路由组件

在 `primevue-steps/src/components` 目录下创建路由文件三个组件提供个`steps`组件使用

- P1.vue
- P2.vue
- P3.vue

![](/images/post/Use-Primevue-Steps-component-vue3/4.png)

**P1.vue**

```html
<template>
    <div>
        <h4>步骤一：/tv/p1</h4>
        <!-- 下一页控制按钮 -->
    <button @click="nextPage">下一步</button>
    </div>
</template>

<script>
import {ref} from 'vue'

export default {
    name: 'P1',
    components: { },
    // 声明接收steps组件传递的数据
    props: {
        formData: Object
    },
    // 声明事件
    emits: ['prevPage', 'nextPage', 'complete'],

    setup(props, ctx) {
        console.log('步骤一, Steps传递的数据:', props.formData)

        // 当前组件的数据, nextPage事件发出时传递给Steps组件
        const data = ref({
            formData: {
                msg: '我是步骤一'
            },
            index: 0
        })

        /**
         * @function 向steps组件发出nextPage事件，让steps组件控制router跳转到下一页
         */
        const nextPage = () => {
            ctx.emit('nextPage', data)
        }

        return {
            nextPage
        }
    }

    
}
</script>

<style scoped>
    div {
        width: 100%;
        height: 100px;
        border-radius: 10px;
        margin-top: 10px;
        box-shadow: 0px 0px 10px 5px #666666;
    }
</style>
```

**P2.vue**

```html
<template>
    <div>
        <h4>步骤二：/tv/p2</h4>
        <!-- 上一页,下一页控制按钮 -->
        <button @click="prevPage">上一步</button>
        <button @click="nextPage">下一步</button>
    </div>
</template>

<script>
import {ref} from 'vue'

export default {
    name: 'P2',
    components: { },
    // 声明接收steps组件传递的数据
    props: {
        formData: Object
    },
    // 声明事件
    emits: ['prevPage', 'nextPage', 'complete'],

    setup(props, ctx) {
        console.log('步骤二, Steps传递的数据:', props.formData)

        // 当前组件的数据, nextPage事件发出时传递给Steps组件
        const data = ref({
            formData: {
                msg: '我是步骤二'
            },
            index: 1
        })

        /**
         * @function 向steps组件发出nextPage事件，让steps组件控制router跳转到下一页
         */
        const nextPage = () => {
            ctx.emit('nextPage', data)
        }

        /**
         * @function 向steps组件发出prevPage事件，让steps组件控制router跳转到上一页
         */
        const prevPage = () => {
            ctx.emit('prevPage', data)
        }

        return {
            nextPage, prevPage
        }
    }

    
}
</script>

<style scoped>
    div {
        width: 100%;
        height: 100px;
        border-radius: 10px;
        margin-top: 10px;
        box-shadow: 0px 0px 10px 5px #666666;
    }
</style>
```

**P3.vue**

```html
<template>
    <div>
        <h4>步骤三：/tv/p3</h4>
        <!-- 上一页,下一页控制按钮 -->
        <button @click="prevPage">上一步</button>
        <button @click="complete">完成/确定</button>
    </div>
</template>

<script>
import {ref} from 'vue'

export default {
    name: 'P3',
    components: { },
    // 声明接收steps组件传递的数据
    props: {
        formData: Object
    },
    // 声明事件
    emits: ['prevPage', 'nextPage', 'complete'],

    setup(props, ctx) {
        console.log('步骤三, Steps传递的数据:', props.formData)

        // 当前组件的数据, nextPage事件发出时传递给Steps组件
        const data = ref({
            formData: {
                msg: '我是步骤三'
            },
            index: 2
        })

        /**
         * @function 向steps组件发出prevPage事件，让steps组件控制router跳转到下一页
         */
        const prevPage = () => {
            ctx.emit('prevPage', data)
        }

        /**
         * @function 向steps组件发出complete事件，告知steps组件步骤完成
         */
        const complete = () => {
            ctx.emit('complete', data)
        }

        return {
            prevPage, complete
        }
    }

    
}
</script>

<style scoped>
    div {
        width: 100%;
        height: 100px;
        border-radius: 10px;
        margin-top: 10px;
        box-shadow: 0px 0px 10px 5px #666666;
    }
</style>
```

#### 2.4.3 为子路由组件指定路由地址

修改路由文件 `primevue-steps/src/router/index.js`，为TestView视图添加子路由

```js
import { createRouter, createWebHistory } from "vue-router";
import HelloWorld from "@/components/HelloWorld"
import TestView from "@/views/TestView"
// 导入子路由组件
import P1 from "@/components/P1"
import P2 from "@/components/P2"
import P3 from "@/components/P3"


const routes = [
    {
        name: "index",
        path: "/",
        component: HelloWorld
    }, {
        name: "testView",
        path: "/tv",
        component: TestView,
        // setps组件子路由
        children: [
            {path: 'p1', component: P1},
            {path: 'p2', component: P2},
            {path: 'p3', component: P3},
        ]
    }
]

const router = createRouter({
    history: createWebHistory(),
    routes
})

export default router;
```

完成后刷新页面并访问路径 `/tv/p1`，即可看到步骤页

![](/images/post/Use-Primevue-Steps-component-vue3/5.gif)


## 3. 演示Demo

文章中所用的测试Demo已经推送到github：[https://github.com/mangfu26/blog-share/tree/primevue-steps](https://github.com/mangfu26/blog-share/tree/primevue-steps)

你可以使用以下命令克隆使用

```sh
git clone -b primevue-steps https://github.com/mangfu26/blog-share.git primevue-steps
```