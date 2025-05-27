---
文章标题: "[[1_Part1-BUG]]" 
文章作者: Dakkk
文章概要: |
  文章针对Ant Design Pro CLI缺少Umi选项、Node.js v17+因OpenSSL兼容性报错、以及Spring Boot单元测试中JUnit4无法获取Spring Bean等常见开发问题，提供了具体原因分析和实用的解决方案。
文章标签:
- "Ant Design Pro CLI"
- "Node.js"
- "OpenSSL"
- "Spring Boot"
- "JUnit"
- "版本兼容性"
- "故障排除"
- "单元测试"
相关文章:
- "[[0_基础加强]]"
- "[[3_SpringMVC]]"
- "[[7_Springboot]]"
- "[[_使用工具的软件驱动]]"
- "[[0_参考文章索引]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/05-📋 用户中心项目/0_bug处理/1_Part1-BUG.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-17 13:55:23
---

## 问题1

问题：安装最新的ant-design/pro-cli的时候，在pro create myapp创建项目的时候 `没有umi选项！`

原因：ant-design版本过高，直接跳过了

解决：删除之间的ant-design，重新安装3.1.0版本
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/251303808daa4e367b890597326a32fb.png)

## 问题2

问题：项目运行时出现`node:internal/crypto/hash:80   this[kHandle] = new _Hash(algorithm, xofLen, algorithmId, getHashCache());`

原因：出现这个问题是node.js 的版本问题，因为 node.js V17开始版本中发布的是OpenSSL3.0, 而OpenSSL3.0对允许算法和密钥大小增加了严格的限制，可能会对生态系统造成一些影响。故此以前的项目在使用 nodejs V17以上版本后会报错。而github项目很多都是之前版本的npm，所以运行时候会出现这个问题。

解决：
- 打开package.json文件夹，在start或者dev处前面加上 
	```json
	set NODE_OPTIONS=--openssl-legacy-provider && 原来的代码
	```
- 如图所示：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5d63120ccfe2f2f54d7c465d0908be70.png)

## 问题3

问题：在test包下创建我们自己的单元测试类，使用@SpringBootTest注解，无法获取到Spring环境中的bean，需要在类上使用@RunWith注解添加Spring配置类的类对象才可以使用

原因：我们使用的是Junit4这个jar包的@Test注解，所以没有@RunWith注解是无法加载Spring环境的，从而无法导入bean

解决：将@Test注解，修改为org.junit.jupiter.api的注解，就可以解决这个问题了
