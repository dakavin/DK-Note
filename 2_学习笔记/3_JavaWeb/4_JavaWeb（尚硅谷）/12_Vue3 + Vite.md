# 1、Vue3 简介和快速使用

## 1.1 Vue3介绍

![](assets/Pasted%20image%2020240319223802.png)

> Vue (发音为 /vjuː/，类似 **view**) 是一款用于构建用户界面的 JavaScript 框架。它基于标准 HTML、CSS 和 JavaScript 构建，并提供了一套声明式的、组件化的编程模型，帮助你高效地开发用户界面。无论是简单还是复杂的界面，Vue 都可以胜任。官网为:<https://cn.vuejs.org/>

 **Vue的两个核心功能：**

-   **声明式渲染**：Vue 基于标准 HTML 拓展了一套模板语法，使得我们可以声明式地描述最终输出的 HTML 和 JavaScript 状态之间的关系。
	- `如下图：将js中的message和html中的sapn关联起来`
-   **响应性**：Vue 会自动跟踪 JavaScript 状态并在其发生变化时响应式地更新 DOM 
	- `如下图：js中的message发生变化，span中的内容也变化`

![](assets/Pasted%20image%2020240319224506.png)
## 1.2 快速使用（非工程化）

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
    </head>
    <body>
        <!-- 这里也可以用浏览器打开连接,然后将获得的文本单独保存进入一个vue.js的文件,导入vue.js文件即可 -->
        <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
        <div id="app">
            <!-- 给style属性绑定colorStyle数据 -->
            <!-- {{插值表达式 直接将数据放在该位置}} -->
           <h1 v-bind:style="colorStyle">{{headline}}</h1>
           <!-- v-text设置双标签中的文本 -->
           <p v-text="article"></p>
           <!-- 给type属性绑定inputType数据 -->
           <input v-bind:type ="inputType" value="helloVue3"> <br>
           <!-- 给按钮单击事件绑定函数 -->
           <button  @click="sayHello()">hello</button>
        </div>

        <script>
            //组合api
            const app = Vue.createApp({
                // 在setup内部自由声明数据和方法即可!最终返回!
                setup(){
                    //定义数据
                    //在VUE中实现DOM的思路是: 通过修改修数据而影响页面元素
                    // vue3中,数据默认不是响应式的,需要加ref或者reactive处理,后面会详细讲解
                    let inputType ='text'
                    let headline ='hello vue3'
                    let article ='vue is awesome'  
                    let colorStyle ={'color':'red'}        
                    // 定义函数
                    let sayHello =()=>{
                        alert("hello Vue")
                    }
                    //在setup函数中,return返回的数据和函数可以在html使用
                    return {
                       inputType,
                       headline,
                       article,
                       colorStyle,
                       sayHello
                    }
                }
            });
            //挂载到视图
            app.mount("#app");
        </script>
    </body>
</html>
```
# 2、Vue3 通过Vite实现工程化

## 2.1 Vite介绍
![|200](assets/Pasted%20image%2020240319231356.png)![|400](assets/Pasted%20image%2020240319231405.png)
> 在浏览器支持 ES 模块之前，JavaScript 并没有提供原生机制让开发者以模块化的方式进行开发。这也正是我们对 “打包” 这个概念熟悉的原因：使用工具抓取、处理并将我们的源码模块串联成可以在浏览器中运行的文件。时过境迁，我们见证了诸如 [webpack](https://webpack.js.org/ "webpack")、[Rollup](https://rollupjs.org/ "Rollup") 和 [Parcel](https://parceljs.org/ "Parcel") 等工具的变迁，它们极大地改善了前端开发者的开发体验

+ 当我们开始构建越来越大型的应用时，需要处理的 JavaScript 代码量也呈指数级增长。
+ 包含数千个模块的大型项目相当普遍。基于 JavaScript 开发的工具就会开始遇到性能瓶颈：通常需要很长时间（甚至是几分钟！）才能启动开发服务器，即使使用模块热替换（HMR），文件修改后的效果也需要几秒钟才能在浏览器中反映出来。如此循环往复，迟钝的反馈会极大地影响开发者的开发效率和幸福感。

> Vite 旨在利用生态系统中的新进展解决上述问题：浏览器开始原生支持 ES 模块，且越来越多 JavaScript 工具使用编译型语言编写。https://cn.vitejs.dev/guide/why.html前端工程化的作用包括但不限于以下几个方面：

1.  `快速创建项目`：使用脚手架可以`快速搭建项目基本框架`，避免从零开始搭建项目的重复劳动和繁琐操作，从而节省时间和精力。

2.  `统一的工程化规范`：前端脚手架可以`预设项目目录结构、代码规范、git提交规范等统一的工程化规范`，让不同开发者在同一个项目上编写出风格一致的代码，提高协作效率和质量。

3.  `代码模板和组件库`：前端脚手架可以`包含一些常用的代码模板和组件库`，使开发者在实现常见功能时不再重复造轮子，避免因为轮子质量不高带来的麻烦，能够更加专注于项目的业务逻辑。

4.  `自动化构建和部署`：前端脚手架可以`自动进行代码打包、压缩、合并、编译等常见的构建工作`，可以通过集成自动化部署脚本，自动将代码部署到测试、生产环境等。
## 2.2 Vite创建Vue3工程化项目

### 1. 项目的创建、启动、停止

> 1 使用命令行创建工程

+ 在磁盘的合适位置上,创建一个空目录用于存储多个前端项目
+ 用vscode打开该目录
+ 在vocode中打开命令行运行如下命令（`不需要提前创建工程目录`）

```shell
npm create vite@latest
```

+ `第一次使用vite时会提示下载vite,输入y回车即可,下次使用vite就不会出现了`![](assets/Pasted%20image%2020240319232117.png)
+ `注意： 选择vue+JavaScript选项即可`

> 2 安装项目所需依赖

+ cd进入刚刚创建的项目目录
+ npm install命令安装基础依赖

``` shell
cd ./vue3-demo1
npm install
```

> 3 启动项目

+ 查看项目下的package.json![|400](assets/Pasted%20image%2020240319232816.png)
- 启动项目
```shell
npm run dev
```

![](assets/Pasted%20image%2020240319232949.png)

> 5 停止项目

- 命令行上 ctrl+c
### 2. 项目的目录结构

> 1. 下面是 Vite 项目结构和入口的详细说明

![|200](assets/Pasted%20image%2020240320135725.png)

- ,vscode 目录：和编辑器有关，无序理会

- node_modules：存放项目所需要的依赖文件

-   `public 目录：用于存放一些公共资源`，如 HTML 文件、图像、字体等，这些资源会被直接复制到构建出的目标目录中。

-   `src 目录：存放项目的源代码`，包括 JavaScript、CSS、Vue 组件、图像和字体等资源。在开发过程中，这些文件会被 Vite 实时编译和处理，并在浏览器中进行实时预览和调试。以下是src内部划分建议：
    1.  `assets` 目录：用于`存放一些项目中用到的静态资源`，如图片、字体、样式文件等。
    2.  `components` 目录：用于`存放组件相关的文件`。组件是代码复用的一种方式，用于抽象出一个可复用的 UI 部件，方便在不同的场景中进行重复使用。
    3.  `layouts` 目录：用于`存放布局组件的文件`。布局组件通常负责整个应用程序的整体布局，如头部、底部、导航菜单等。
    4.  `pages` 目录：用于`存放页面级别的组件文件`，通常是路由对应的组件文件。在这个目录下，可以创建对应的文件夹，用于存储不同的页面组件。
    5.  `plugins` 目录：用于`存放 Vite 插件相关的文件`，可以按需加载不同的插件来实现不同的功能，如自动化测试、代码压缩等。
    6.  `router` 目录：用于`存放 Vue.js 的路由配置文件`，负责管理视图和 URL 之间的映射关系，方便实现页面之间的跳转和数据传递。
    7.  `store` 目录：用于`存放 Vuex 状态管理相关的文件`，负责管理应用程序中的数据和状态，方便统一管理和共享数据，提高开发效率。
    8.  `utils` 目录：用于`存放一些通用的工具函数`，如日期处理函数、字符串操作函数等。

-   `vite.config.js 文件`：Vite 的配置文件，可以通过该文件配置项目的参数、插件、打包优化等。该文件可以使用 CommonJS 或 ES6 模块的语法进行配置。

-   `package.json 文件`：标准的 Node.js 项目配置文件，包含了项目的基本信息和依赖关系。其中可以通过 scripts 字段定义几个命令，如 dev、build、serve 等，用于启动开发、构建和启动本地服务器等操作。

-   `Vite 项目的入口为 src/main.js 文件，这是 Vue.js 应用程序的启动文件，也是整个前端应用程序的入口文件`。在该文件中，通常会引入 Vue.js 及其相关插件和组件，同时会创建 Vue 实例，挂载到 HTML 页面上指定的 DOM 元素中。

>  2.vite的运行界面

+ 在安装了 Vite 的项目中，可以在 npm scripts 中使用 `vite` 可执行文件，或者直接使用 `npx vite` 运行它。下面是通过脚手架创建的 Vite 项目中默认的 npm scripts：(package.json)

```json
{
  "scripts": {
    "dev": "vite", // 启动开发服务器，别名：`vite dev`，`vite serve`
    "build": "vite build", // 为生产环境构建产物
    "preview": "vite preview" // 本地预览生产构建产物
  }
}
```

+ 运行设置端口号：(vite.config.js)

```javascript
//修改vite项目配置文件 vite.config.js
export default defineConfig({
  plugins: [vue()],
  server:{
    port:3000
  }
})
```
### 3. 项目组件（SFC入门！！！）

> `什么是VUE的组件?`

- `一个页面作为整体，是由多个部分组成的，每个部分在这里就可以理解为一个组件`
- `每个.vue文件就可以理解为一个组件`，多个.vue文件可以构成一个整体页面
- 组件化给我们带来的另一个好处就是`组件的复用和维护`非常的方便![](assets/Pasted%20image%2020240320140822.png)
> `什么是.vue文件?`

+ 传统的页面有`.html文件.css文件和.js文件`三个文件组成(多文件组件) 
+ `vue将这文件合并成一个.vue文件(Single-File Component，简称 SFC,单文件组件)`

+ .vue文件对js/css/html统一封装,这是VUE中的概念 该文件由三个部分组成    `<script>    <template>    <style>`
	+ template标签     代表组件的html部分代码	代替传统的.html文件
	+ script标签           代表组件的js代码 代替传统的.js文件
	+ style标签            代表组件的css样式代码 代替传统的.css文件	

- template标签下，vscode的语法只能有一个内标签，否则报红(不影响使用)![](assets/Pasted%20image%2020240320144357.png)

> `工程化vue项目如何组织这些组件?`

+ index.html是项目的入口,其中 `<div id ='app'></div>`是用于挂载所有组建的元素
+ index.html中的`script标签引入了一个main.js文件，具体的挂载过程在main.js中执行`
+ main.js是vue工程中非常重要的文件,他`决定这项目使用哪些依赖,导入的第一个组件`
+ App.vue是vue中的核心组件,所有的其他组件都要通过该组件进行导入,该组件通过路由可以控制页面的切换![|400](assets/Pasted%20image%2020240320142456.png)  ![](assets/Pasted%20image%2020240320140952.png)
	1. index.html页面导入main.js文件
	2. main.js文件，将App.vue这个组件，挂载到index.html页面中id为app的元素上
### 4. 响应式入门和setup函数

> 1 `使用vite创建一个 vue+JavaScript项目`

```shell
npm create vite
npm install 
npm run dev
```

+ App.vue

```html
<script>  //去掉了标签内的setup字样
    //存储vue页面逻辑js代码
</script>

<template>
    <!-- 页面的样式的是html代码-->
</template>

<style scoped>
    /** 存储的是css代码! <style scoped> 是 Vue.js 单文件组件中用于设置组件样式的一种方式。
    它的含义是将样式局限在当前组件中，不对全局样式造成影响。 */
</style>
```

> 2 vue3响应式数据入门

- `非响应式数据`: 修改后VUE不会更新DOM

- `响应式数据`:   修改后VUE会更新DOM
	- `VUE2中数据默认是响应式的`
	- `VUE3中数据要经过ref或者reactive处理后才是响应式的`
		- ref是VUE3框架提供的一个函数,需要导入
		- let counter = 1
		- ref处理的响应式数据`在js编码修改的时候需要通过.value操作`
		- ref响应式数据`在绑定到html上时不需要.value`
		- ref和reactive函数需要导入`import {ref，reactive} from 'vue'`

```html
<script type="module">
    //存储vue页面逻辑js代码
    import {ref} from 'vue'
    export default{
        setup(){
            let counter = ref(1)
            function increase(){
                // 通过.value修改响应式数据
                counter.value++
            }
            function decrease(){
                counter.value--
            }
            return {
                counter,
                increase,
                decrease
            }
        }
    }
</script>
<template>
    <div>
      <button @click="decrease()">-</button>
      {{ counter }}
      <button @click="increase()">+</button>
    </div>
    
</template>

<style scoped>
    button{
        border: 1px solid red;
    }
</style>
```

> 3 vue3 setup函数和语法糖

+ 位置：src/App.vue
+ `<script type="module" setup>` 通过`setup关键字`可以省略 export default {setup(){   return{}}}这些冗余的语法结构

```vue
<script type="module" setup>
    import {ref} from 'vue'
    // 定义响应式数据
    let counter = ref(1)
    // 定义函数
    function increase(){
        counter.value++
    }
    function decrease(){
        counter.value--
    }
    
</script>
<template>
    <div>
      <button @click="decrease()">-</button>
      {{ counter }}
      <button @click="increase()">+</button>
    </div>
    
</template>

<style scoped>
    button{
        border: 1px solid red;
    }
</style>

```
### 5. 关于样式的导入问题

1. 全局样式引入，修改main.js文件中的引入的css样式文件即可，或者继续导入其他的css样式文件![|300](assets/Pasted%20image%2020240320145139.png)

   ```javascript
import './style/reset.css' //书写引入的资源的相对路径即可！
   ```

2. vue文件（单独组件）在script标签引入

   ```javascript
import './style/reset.css'
   ```

3. Vue文件（单独组件）在style中引入

   ```javascript
@import './style/reset.css'
   ```

- 总结：`一般很少在vue文件中的style标签下写样式，就算写也是针对当前文件的`

### 5. JS和TS的选择问题

> TS是JS的一个超集,使用TS之后,JS的语法更加的像JAVA,实际开发中用的确实更多,那么这里为什么我们没有立即使用TS进行开发,原因如下

1 为了降低难度,提高前端工程化的效率

2 对于学JAVA的我们来说,学习TS非常容易,但是还是需要一些时间

3 TS不是非学不可,不用TS仍然可以正常开发工程化的前端项目

4  尚硅谷已经发布了TS的专项课程,请大家在B站上自行搜索 "尚硅谷  TS" 

5  建议大家先学完完整的前端工程化内容,然后再根据需求单独学习TS即可


## 3、补充

### 3.1 vscode快速创建vue3模版

![](assets/Pasted%20image%2020240320163057.png)

- 修改vue.json文件![](assets/Pasted%20image%2020240320163132.png)

- vue2
```json
"Print to console": {
		"prefix": "vue",
		"body": [
			"<template>",
			"  <div class='$1'>",
			"",
			"  </div>",
			"</template>",
			"",
			"<script>",
			"export default {",
			"",
			"}",
			"</script>",
			"",
			"<style scoped>",
			"",
			"</style>",
			"$2"
		],
		"description": "Vue Template"
	}
```

- vue3
```json
"Print vue with setup": {
		"prefix": "vuest",
		"body": [
			"<template>",
			"  <div class='$1'>",
			"",
			"  </div>",
			"</template>",
			"",
			"<script setup>",
			"const props = defineProps({",
			"",
			"});",
			"</script>",
			"",
			"<style scoped lang='less'>",
			"",
			"</style>",
		],
		"description": "Log output to console"
	},

```





