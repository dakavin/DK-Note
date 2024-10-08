

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
