
本小节中，将带着大家通过 `Animate.css` 给登录页添加入场动画，让页面过渡更加友好，提升逼格。

## 1 最终完成的效果图

![169266850100221.gif|400](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4ba9a3e625eca5163a9084e854db77cf.gif)

## 2 Animate.css 是什么？

`Animate.css` 是一个跨浏览器的 CSS 动画库，提供了许多预设的、流畅的动画效果。用户只需添加几个 CSS 类名，就可以轻松实现复杂的动画效果，无需编写任何 JavaScript 代码。

## 3 为什么使用 Animate.css？

- **简单易用**：通过添加或删除类名，你可以触发动画，这使得实现动画效果变得非常直观。
- **高性能和跨浏览器**：`Animate.css` 专为性能进行了优化，并在多数现代浏览器中表现良好。
- **丰富的动画选择**：无需从零开始设计动画，库中提供了大量预制动画，如“fadeIn”、“bounce”、“zoomIn”等。
- **与 Vue 配合完美**：Vue 提供了 `transition` 组件，可以与 `Animate.css` 轻松结合，为你的 Vue 应用添加精美动画。

## 4 开始动手

### 4.1 安装

打开命令行，执行如下安装命令：
```
npm install animate.css --save
```

### 4.2 引入

在 `main.js` 文件中引入它：
```
import 'animate.css';
```

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9d5a2f4ed89fa7c5e5ebe0ce0586a078.png)

引入后，就可以添加相关动画效果了。

### 4.3 选择自己喜欢的动画效果

可访问 `Animate.css` 官网：[https://animate.style/](https://animate.style/) ，右侧栏是支持的动画，点击可查看效果，喜欢哪个就复制哪个，并应用到自己的项目中：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0667699f2d40a3f0836322b1a99a9683.png)

### 4.4 添加弹跳动画

编辑 `/admin/login.vue` ，给右边栏的父级 `div` 添加 `bounceInRight` 动画，样式类名如下：

```
animate__animated animate__bounceInRight animate__fast
```

- `animate__animated` : Animate.css 中的一个核心类名，它被用于触发动画效果。这个类名为元素定义了动画的基础属性，使其能够与其他具体的动画效果类名（如 `animate__fadeIn`、`animate__bounce` 等）结合使用，从而产生动画效果;
- `animate__bounceInRight` : 指定为从右弹入；
- `animate__fast` : 指定动画速度为快；

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8151bd828236f3f5364a81190740eb26.png)

再给左边栏的父级 `div` 添加 `bounceInLeft` 动画，样式类名如下：

```
animate__animated animate__bounceInLeft animate__fast
```
- `animate__bounceInLeft` : 指定从左弹入动画；

![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/feaeb45d1763a570f52f9991ee15c2da.png)


添加完成相关动画，再次访问页面，即可看到文章头所示的效果啦~

## 5 结语

`Animate.css` 提供了一种简单但强大的方式来给 Web 应用增加吸引人的动画效果。与 Vue 3 的整合使得在应用中添加动画变得更为直接和简洁。为你的用户带去更加动态和生动的体验，无疑可以提高用户的满意度和参与度。
