
## 1 前端实现

### 1.1 注册功能的开发

**开发思路：**
- 首先，根据前端页面的路径（如：user/login），找到对应的路由，==路由在config目录下的routes.ts文件中==
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1e391f917da28e259dfe30c8c55674ff.png)

- 根据上图，发现了登录页面的路由，那我们接着开发主页页面

---

**开发注册页面**
- 直接复制login页面的page，复制到register目录下
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dee5662c15630e0caf61ebe126ef4f9f.png)
- 修改index.tsx文件中，与Login相关的字段
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f9f4c621f6d112019da09d0d5a2a292d.png)
  - 先修改这两个地方即可
    ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7adc31e606dcc9b01e18287dddddee50.png)
- ==页面等会还需要修改！==

---

**url地址设置：设置什么才可以访问到我们刚刚设计的注册页面？**
- 设置config目录下的routes.ts文件，修改如图
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6957ea51119ae696c27e2c6fc941a9b3.png)
- 检查前端项目中存在的==重定向逻辑==，避免我们无法进入register页面
	- 一般重定向逻辑都在 `app.tsx`文件中，我们查看一下
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/193d7c6376a305a535a2fc2fb3ca637c.png)
	  - 我们修改这两段代码，修改如下图
	    ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/33efee74f30847fb4b987f8007e50ced.png)
	- 浏览器的url直接输入注册页面，验证重定向是否修改成功，成功如下图
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f77270061c0d7e6c359b4bf795ef4887.png)
> 补充：`app.tsx`文件是当前前端项目的全局文件，定义类刚进入一个页面时，需要执行的逻辑

---

**继续修改注册页面（html修改）**
- 删除获取登录用户信息的代码，代码如下图
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6a1292d7ac7c2af6528c982ce4320114.png)

- 修改为==账户密码注册==
	![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/de3bd2a98994892b06e67f3a3923117f.png)

- 增加一个==密码确认框==
	![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/949f5d02d46d09813aafb704dd0d8226.png)

- 去掉自动登录和相关底部的设置
	![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/377eb313e7b7198c2bf401325681c9d0.png)
- 全局替换字样 ==登录== 改为 ==注册==，想修改底部bar，登录为注册
	![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f3c7261a84f6ca83ba32eb2c981bb887.png)![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/310c5de0f17f0899f2ff62b9c22817c2.png)
	- 发现并没有修改成功，应该和全局组件有关，查看一下，发现和 **LoginForm** 这个标签有关
	- 先ctrl加鼠标左键点击这个标签，然后全局定位这个组件，发现在下图文件夹中
	- 补充：[3.2 ant design pro后台管理系统概念](0_项目知识补充.md#3.2%20ant%20design%20pro后台管理系统概念)
		![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/542cab4c84ffaba5bd6b7088335be5e1.png)
	- 和ant design框架的pro from有关，根据上述概念去[ProComponents官网](https://procomponents.ant.design/)看看，找到==组件==，然后查看==ProFrom==，==结果看不到哪里可以设置==
		![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1f309ed87feb575f2f61532d8a281da2.png)
	- 那直接看这个文件夹中的index.js文件，发现这里出现了==登录字样==![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bc6d72bc4afc2698d0d150133f86e963.png)
	- 我们直接模仿。这个searchConfig，来==在登录页面进行添加==，如下图
		![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/01dc887acee0dce02e1d254d4929bb17.png)
	- OK。修改成功

- 补充：去掉用户的登录状态
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/25455a4e2355fe10b68edfce38a3d8f9.png)
---

**编写注册表单提交的逻辑（js修改）**

- ==修改提交的参数==
	- 根据我们上一个Part，给登录增加的逻辑，那么我们定位到表单提交的方法 onFinish方法
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/675b6505059975c00c2c1f4a2335d0d8.png)
	- ctrl+鼠标左键，查看方法的参数，发现是我们之前登录使用的参数
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/798c1582148368bb7d958ed5ba49d8a5.png)
	- 我们自行编辑一个注册使用的参数
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4c7fda5bc97d1fb554c3a20dae174e9e.png)
	- 全局替换一下参数
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/83cafb1980120363ec13c943a3e6b9be.png)

- ==前端校验两次密码是否相等==，直接获取values中的两个密码，进行对比即可
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8cd996b5821755129fe37a505cbb349c.png)

- ==修改提交的url地址==
	- 修改方法传入的参数
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a16a5cee469a3adf9057598c3de8f32e.png)
	- 点进去login方法，复制这个方法，写一个register方法
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8422aa4485b446c32bfea68eeeb97d08.png)
	- 查看后端知道，注册返回用户的id，修改返回的值，即修改上图中的 `API.LoginResult`为`API.RegisterResult`，然后修改即可
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6ee1cd01894348244f7ed8b1a3cb3f7f.png)

	- 修改注册页面中，注册方法的逻辑，如下图
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/500a2a218eaa8fe4be4881eeb8ba82a9.png)


- 登录页面添加一个链接，链接到注册页面
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0f491e942304a14577157a6d7bfc0a7f.png)


- ==开心的去注册吧！==

**待优化的点：**
- 注册失败的提示可以丰富一点
	- 两次密码输入不一致（前端已经完成了，不太丰富，可能有别的优化方式）
	- 用户已被注册（输入用户名的时候，向数据库异步请求）

### 1.2 登录页面的开发（js）

**注意：需要`先`写获取用户的登录态的接口，前端向这个接口获取当前用户信息** [2.1 获取用户的登录态](#2.1%20获取用户的登录态)

**从后端获取当前用户信息**
- 查看前端项目全局入口文件`app.tsx`，可以发现有个方法是获取当前用户信息的，如下图
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7f10646a3e80382a0bd56050fbe13fd1.png)
- 我们修改进入这个`queryCurrentUser（）`方法中去，就来到了`api.ts`文件，可以看到
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d740edb68688ed687a612429bf625771.png)
- 对这个方法修改即可
	- 修改url和后端统一
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/13d1d8081f2311881da986d75150990e.png)
	- 修改方法的返回值，点击`API.CurrentUser`进去看一下，我们将前端框架的用户字段修改
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f9fe6538c4691dc8af457fe16065c50e.png)
- 回到`api.tsx`文件，对获取用户态的方法，再次修改
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b21edc5704049adfb5de29fbeb13f6d8.png)

 **优化全局文件app.tsx的重定向逻辑：**
- 将最初定义的白名单，放到文件前面，当成常量
	![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bf7799d1a6db1f623a205ea1388dd121.png)
	- 修改重定向规则
		![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e092ecebc21c2ed38d5e6af28117f5d4.png)
- 修改白名单名称为`NO_NEED_LOGIN_WHITE_LIST`
	![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9dbf55860c8c4a4c7ec9954f98606591.png)
- 在检查一下，代码错误修改（下图是个错误的代码--第一个框）
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4f9fadb2b09e98801ace7f56175a9df9.png)
- 📕解决：返回的 `currentUser`不能包装在data这个数据里面，不然`全局的initialState`读取不了这个用户，导致重定向到登录页面，正确的逻辑如下图：
	![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/59eda99e7f4f012f11e7911d210ceefc.png)

--- 

**前后端联调：测试是否成功登录**
- 可以登录成功，==但是右上角的头像和用户名一直在加载，显示用户名的水印也没有==
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9e9f100895d241f2410fc7b77fc962ef.png)
	- 因为登录的这个用户，在数据库中没有用户名和头像，去数据库中添加即可

- 添加完成后进行测试，==发现水印出来了，但是头像没出来==；因为字段对不上，我们数据库的是avatarUrl
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dda42af2097b875d17059079f28f1bb3.png)
- 全局搜索一下`avatar` 或者 去组件（`Components目录`）里面找一找也行，找到AvatarDropdown.tsx文件修改一下头像的变量名
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3e3baafd29e9fc03e9d42d01d0b118d3.png)
- 文件修改如下：直接在当前文件全局搜索`currentUser`，对其取值进行修改即可（下图还漏了一个地方，小伙伴可以自己搜索，很简单）
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9b528384b67ebf696defbdb5c84f33a1.png)
- 重新刷新一下，头像也出来了
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3e98e462a5f74d0360f651e74f843afc.png)

### 1.3 用户管理后台的开发

**用户管理页面权限设置**
- 框架有一个现成的，去看看代码（`src/pages/TableList/index.tsx`），这个代码很复杂，我们可以自己弄一个精简一点的
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/95a63a962bc24c8a1de3ba821aa2e078.png)
  - 在page文件夹下新建Admin文件夹，把user文件夹下的Register文件夹复制，粘贴到admin文件夹下，并改名为UserMange
    ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0663f8e59835c1c86019c8aa0edd9f3f.png)
- 去 route.ts 添加一个路由
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e44b419c39b0968a03bd406d2c522191.png)

- 访问一下，无权访问
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3a0e97f0095c45cd9ad46b79cfe72309.png)
- 对比上面的/user,发现/admin==多了`access:'canAdmin'`也就是管理员才能访问==
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/66bc8e453cb9faf72ad2122c0a1a6869.png)
	- access来自 ==access.ts 文件（框架中 access.ts 控制用户的访问权限）==，打开文件进行修改
		- 以后我们还可以==开发用户是否VIP的access==
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/95572e7d7d1ad752db0366ce35776e1a.png)
	- 使用一个管理员账户登录，发现可以进行用户管理
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1b1687cea54ad69f082ea4e1673cd53e.png)
**开发用户管理页面**
- 把二级管理员的路由删除掉
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c177eb0dfefcbd2bd39295c869321ea8.png)
- 对照小伙伴笔记，它们的页面显示的是 Admin.tsx文件的内容，但是我的直接显示注册页面
	- 发现自己的路由文件中没有下面这个参数，所以直接显示注册页面的内容，加上就和上述问题一致了
	  ![image.png|200|200|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c2e80a8333c1403739048394639ab8ea.png)
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6694f0467c0b4b0ae60d329562c803a8.png)
- 修改Admin.tsx文件的内容，9-40行的内容删除，剩下的内容如图：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dfbc06b41419a2762901b6511d02b549.png)
- 此时页面显示了我们的注册页面
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d2a7b3abfe047d43bbe1a6e0cc31377e.png)
**更换页面为表格页面**
- 去[官方文档](https://procomponents.ant.design/components/table)直接用现有的表格，这个就很不错，复制代码直接使用（==代码直接全部覆盖==）
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dbb36b002876730fe8ed7a409060f267.png)
- 删除多余的部分
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/89a5f879266c1f0e73bf6112131b512f.png)
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6de87a78690ca2aedd5494d3265b1b3e.png)
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f65d2e972ca9f17005950036c9fa730a.png)
- 修改列名，改为 `API.CurrentUser`
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/af6196b30b48f046e80cb47dd91fa95b.png)
- 修改表格展示的列，修改如下：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0a8301d502b43abd4c395bbd0ffa3e69.png)
	- 补充：日期需要加上 `valueType: 'dateTime'`
	- 问题：后端的日期和前端的日期对不上，[参考文章](https://cloud.tencent.com/developer/article/2226172)
	  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/016f6517e3063b9eb7e9df1158f054e9.png)

- 在api.ts增加一个搜索用户的接口，复制这段代码，粘贴到这段代码上面（==本质：将返回的用户列表封装到了API.NoticeIconList中的data数据==）
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a7d90b94b2c3d796d3638d0db25957ca.png)
- 回到用户管理的主页 index.tsx，方法修改为如下：
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dfa12c99ba5c1eb89b8f1c49244abc1e.png)
- 访问管理页面
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0c74e1dbcf90a41b6f9ecea53631b9a8.png)
- 头像显示有问题，改一些它的渲染逻辑
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0d34f9aca6517840333396e725f6d10b.png)
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/babc121baa6cc57f27791116488027e1.png)

- 这里角色显示0和1，不太友好，优化一下，参考官网的源码
  ![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cdca6348779220226ee41bea1c62232a.png)
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/53a0d9dcd242f424c98b8e62dd7b49c3.png)

## 2 后端实现

### 2.1 获取用户的登录态的接口

**UserController代码增加返回当前用户信息的接口**
```java
@GetMapping("/current")  
public User getCurrentUser(HttpServletRequest req){  
    Object userObj = req.getSession().getAttribute(USER_LOGIN_STATE);  
    User currentUser = (User) userObj;  
    if(currentUser == null){  
        return null;  
    }  
    //注意不能直接返回当前用户信息，最好实时更新一下（查库）  
    long userId = currentUser.getId();  
    //todo 没有判断用户是否被封号  
    User user = userService.getById(userId);  
    //注意getSafetyUser方法内部需要判空，避免数据库查出来的用户为null
    return userService.getSafetyUser(user);  
}
```

## 3 项目扩展思路（待优化的地方）

- 前端密码重复的提示（可能有别的优化方式）
- 注册没有比较友好的提示
- 用户注销
- 优化管理员查询用户的接口，支持更多样的查询
