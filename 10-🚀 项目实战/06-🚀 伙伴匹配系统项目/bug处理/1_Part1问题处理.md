---
文章标题: "[[1_Part1问题处理]]" 
文章作者: Dakkk
文章概要: |
  文章汇总了Vue 3开发中常见的5个问题及解决方案，涵盖Vant依赖、TypeScript识别Vue文件、Vite热加载、Vant版本兼容性及Vue数据引用错误，提供实际开发中的问题排查与解决思路。
文章标签:
- "Vue 3"
- "Vant"
- "TypeScript"
- "Vite"
- "故障排除"
- "前端开发"
- "热加载"
相关文章:
- "[[17_Element-plus组件库]]"
- "[[12_Vue3 + Vite]]"
- "[[16_Vue3 状态管理pinia]]"
- "[[1_Vue 3环境安装 和 blog前端项目搭建]]"
- "[[12_添加Table组件加载Loading、表单对话框提交按钮Loading动画]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/06-🚀 伙伴匹配系统项目/bug处理/1_Part1问题处理.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2024-08-11 18:15:12
---


## 问题1

问题：报错 `Failed to resolve import "vant/es" from "src/components/HelloWorld.vue". Does the file exist?`

原因：尽管是按需引入vant的，但是我们还是需要安装vant依赖

解决：使用命令行，安装vant依赖，命令`npm i vant`

## 问题2

问题：vue中ts无法识别引入的vue文件，提示找不到xxx.vue 模块解决办法![image.png|380|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/67f94dd57bccf8d64d839428b2e345e0.png)
原因：typescript 只能理解 .ts 文件，无法理解 .vue文件

解决：在项目根目录或 src 文件夹下创建一个后缀为`shims-vue.d.ts`的文件，并写入以下内容：

```text
declare module '*.vue' {
  import { ComponentOptions } from 'vue'
  const componentOptions: ComponentOptions
  export default componentOptions
}
```

## 问题3

问题：更新css样式，刷新浏览器不会热加载

原因：没有开启热加载

解决：在`vite-config-ts`中设置热更新
```java
server: { host: 'localhost', hmr: true }
```

改完代码保存后，重新断开跑一下才会生效。


## 问题4

问题：Uncaught TypeError: Toast is not a function

原因：使用组件库是vant3版本的，但是npm安装的是vant4版本

解决：更换vant的版本，最好是按照最新的vant版本来，**看清楚快速入门的说明**


### 问题5

问题：Vue3报错：Property “xxx“ was accessed during render but is not defined on instance.

原因：执行的数据没有使用单引号 或 双引号包括起来

解决：外层是双引号，内部引用字段使用单引号包括起来
