
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
