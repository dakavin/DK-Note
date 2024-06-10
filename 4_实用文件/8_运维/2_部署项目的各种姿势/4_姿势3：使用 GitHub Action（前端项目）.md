
首先，是在Docker的nginx容器的基础上进行的 GitHub Action 工作流

即，我们需要保证
- nginx容器的运行和挂载的数据卷存在的！！！
- 项目是在GitHub仓库中的！！！

## 1 配置GitHub Secrets（添加 SSH Secrets）

名词解析：
- `SSH_HOST`: 服务器的IP地址或域名
- `SSH_USER`: SSH用户名
- `SSH_PRIVATE_KEY`: SSH私钥（可以使用`cat ~/.ssh/id_rsa`获取）

**步骤一：创建服务器的私钥和公钥**

首先进入 Linux 系统的用户目录下的 .ssh 目录
```
cd ~/.ssh/
```

使用ssh-keygen生成一组公私秘钥对
```javascript
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"


如 ssh-keygen -t rsa -C "123@gmail.com"
```

命令生成之后使用 ls 命令查看一下
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/390e1cf3a6794253f960497a0737f238.png)


然后ls，查看服务器的私钥和公钥
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/8ff572661783460eeb76fcfc99171b0b.png)

没有pub后缀的是私钥，有pub后缀的是公钥，我们可以使用cat命令，查看秘钥
```shell
# 查看私钥
cat ~/.ssh/id_rsa

# 查看公钥
cat ~/.ssh/id_rsa.pub
```

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/38becff29e059f758b5d16cb22527790.png)

**将公钥添加到服务器**

- 使用以下命令将公钥复制到服务器的 `authorized_keys` 文件中：
```shell
ssh-copy-id -i ~/.ssh/id_rsa.pub root@服务器ip地址
```
- 如果 `ssh-copy-id` 工具不可用，可以手动将公钥内容追加到服务器上的 `~/.ssh/
authorized_keys` 文件：
```shell
ssh root@服务器ip地址
mkdir -p ~/.ssh
echo "公钥内容" >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
- 验证SSH连接，在服务器使用新生成的私钥文件验证SSH连接：
```shell
ssh -i ~/.ssh/id_rsa root@服务器ip地址
```

---

**步骤二：添加公钥**

按照操作步骤，将id_rsa.pub文件内容复制到GitHub仓库中

![|380|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/adfc5bfb848c0a152d5c01e490f9eaaa.png)

点击Add按钮，增加SSH公钥信息，可以看到添加成功

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/c64f6c1bcebb9bbf46001a5aecbefa0a.png)

---

**步骤三：添加私钥**

在`home`项目的GitHub仓库中，导航到`Settings` -> `Secrets` -> `Actions`，添加以下私钥

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/d7bd1ec3c4b52849d8bcde57b54de27e.png)

项目的私钥名称为`SSH_PRIVATE_KEY`，将cat获取的私钥内容全部复制进去

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/0ef1ec8ff9f497eb07972ef0021aacc7.png)

---

**步骤四：添加 `SSH_USER` 和 `SSH_HOST`**

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/7bf11f03220ad178889ef5531f29c73a.png)


## 2 创建GitHub Actions工作流文件

正式开始之前，你需要掌握 GitHub Action 的基础语法：
- `workflow` （工作流程）：持续集成一次运行的过程，就是一个 workflow。
- `name`: 工作流的名称。
- `on`: 指定次工作流的触发器。push 表示只要有人将更改推送到仓库就会触发工作流运行。（点击这里了解如何指定特定分支，路径或标签）
- `jobs`: 将工作流运行的所有作业组合到一起。
- `build-and-deploy`: 定义的作业的名称。
- `runs-on`: 将作业配置为在最新版本的 Ubuntu Linux 上运行。这意味着作业将在 GitHub 托管的新虚拟机上执行。有关使用其他运行器的语法示例，请参阅 GitHub 操作的工作流语法。
- `steps`: 将作业中运行的所有步骤组合在一起。嵌套在此部分下的每个项都是一个单独的操作或 shell 脚本。
- `uses`: 指定需要运行的 action。
- `env`: 指定运行 action 时需要用到的环境变量的值。

创建home项目的的Actions相关文件，在home项目的根目录下新建 `.github/workflow/build.yml` 文件，，名称默认或者自定义修改，目录结构如下图：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/12afe6e3f8ca39e96f8d545f710dba26.png)

配置文件如下：
```yml
name: Deploy Home  
  
# 触发条件：当代码推送到 master 分支时触发工作流  
on:  
    push:  
        branches:  
            - master  
  
jobs:  
    deploy:  
        # 使用最新的 Ubuntu 作为运行环境  
        runs-on: ubuntu-latest  
  
        steps:  
            # 步骤1：检出代码  
            # 这是一个 GitHub 官方的动作，用于检出仓库代码到 runner 环境  
            - name: Checkout code  
              uses: actions/checkout@v2  
  
            # 步骤2：缓存 Node.js 模块  
            - name: Cache Node.js modules  
              uses: actions/cache@v2  
              with:  
                  path: ~/.npm  
                  key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}  
                  restore-keys: |  
                      ${{ runner.os }}-node-    
  
            # 步骤3：设置 Node.js 环境  
            # 使用 actions/setup-node@v2 设置 Node.js 版本为 20.12.0            
            - name: Setup Node.js  
              uses: actions/setup-node@v2  
              with:  
                  node-version: '20.12.0'  
  
            # 步骤3：安装依赖项  
            # 运行 npm install 安装项目的所有依赖项  
            - name: Install dependencies  
              run: npm install  
  
            # 步骤4：构建项目  
            # 运行 npm run build 构建项目，将生成的文件放入 dist 文件夹  
            - name: Build project  
              run: npm run build  
  
            # 步骤5：设置 SSH Key            
            - name: Setup SSH Key  
              run: |  
                  echo "${{ secrets.SSH_PRIVATE_KEY }}" > deploy_key  
                  chmod 600 deploy_key  
  
            # 步骤6：通过 SSH 确保目标路径存在  
            # 通过 SSH 连接到服务器并创建必要的文件夹路径  
            # 如果目录已经存在也不会报错  
            - name: Ensure target directories exist  
              run: |  
                  ssh -o StrictHostKeyChecking=no -i deploy_key ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} << 'EOF'  
                  mkdir -p /docker-data/nginx/html/home  
                  mkdir -p /docker-data/nginx/cert/home-cert  
                  mkdir -p /docker-data/nginx/conf.d  
                  EOF  
  
            # 步骤7：通过 SSH 复制 dist 文件夹内容到服务器  
            - name: Copy dist files via SSH  
              uses: appleboy/scp-action@v0.1.1  
              with:  
                  host: ${{ secrets.SSH_HOST }}  
                  username: ${{ secrets.SSH_USER }}  
                  key: ${{ secrets.SSH_PRIVATE_KEY }}  
                  source: "dist/*"  
                  strip_components: 1 # 确保不会在目标路径中创建额外的目录层级。  
                  target: "/docker-data/nginx/html/home/"  
  
            # 步骤8：通过 SSH 复制 SSL 证书到服务器  
            - name: Copy SSL certs via SSH  
              uses: appleboy/scp-action@v0.1.1  
              with:  
                  host: ${{ secrets.SSH_HOST }}  
                  username: ${{ secrets.SSH_USER }}  
                  key: ${{ secrets.SSH_PRIVATE_KEY }}  
                  source: "nginx/config.d/home-cert/*"  
                  strip_components: 3  
                  target: "/docker-data/nginx/cert/home-cert/"  
  
            # 步骤9：通过 SSH 复制 Nginx 配置文件到服务器  
            - name: Copy Nginx config via SSH  
              uses: appleboy/scp-action@v0.1.1  
              with:  
                  host: ${{ secrets.SSH_HOST }}  
                  username: ${{ secrets.SSH_USER }}  
                  key: ${{ secrets.SSH_PRIVATE_KEY }}  
                  source: "nginx/config.d/home.conf"  
                  strip_components: 2  
                  target: "/docker-data/nginx/conf.d/"  
  
            # 步骤10：重启 Nginx 容器  
            - name: Restart Nginx container  
              run: |  
                  ssh -o StrictHostKeyChecking=no -i deploy_key ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} << 'EOF'  
                  docker restart nginx  
                  EOF
```

## 3 测试Actions工作流（Home项目）

我们这里进行测试，看看提交代码后，能否将文件正确COPY到对应的目录

`/docker-data/nginx/....`  统一改为 `/docker/nginx/....`

提交代码进行测试，检查actions工作流

![github action.gif|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/87d240a16e30165e26ce79d8901e5569.gif)

可以发现，完成了整个github aciton工作流

检查一下是否成功输出到`/docker/nginx/....`目录下，可以看到文件都被正确的拷贝了
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/6c93d3351d3c5520443428e8a2ecd26a.png)

验证文件是否会被覆盖，我们修改一下home.conf，修改如下

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/e25b93f26bc20ede3bdd54ee11f6a954.png)

当前的home.conf 文件如下

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/9d0a5684341e5b48fd22c4e59e4064ca.png)

继续提交代码，等待github action执行完成后，查看文件
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/4c950007f01986c843ae46b16eae9a23.png)

发现文件被成功覆盖了


修改配置文件恢复拷贝数据到docker-data的数据卷中 `/docker-data/nginx/....`  

再次提交检查是否覆盖原文件，源文件修改时间如下：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/b9903a5db25cb0cbabd7746e395718c3.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/e20378d059383ce17fcc874128dc4e97.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/8fc84dab6dc4e1b0705020acf8b8782f.png)

查看工作流完成后，数据卷的文件情况

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/d2402377ffe753836ef73258cd7e9b03.png)

可以发现，正常覆盖原来的文件


## 4 给usercenter-frontend项目也添加actions工作流

在该项目的github仓库下设置对应的相关配置
- `SSH_HOST`: 服务器的IP地址或域名
- `SSH_USER`: SSH用户名
- `SSH_PRIVATE_KEY`: SSH私钥（可以使用`cat ~/.ssh/id_rsa`获取）

具体方法，参考上面哈，设置完成效果如下

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/097828fc4da431d52c2a3c9dc1ac33b7.png)



创建对应的文件夹，和编写build.yml文件（注意相关配置的路径要保持不变！）

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/e592a20fdd05f8bc7026724126993842.png)

修改build.yml文件
```yml
name: Deploy U=usercenter-frontend  
  
# 触发条件：当代码推送到 master 分支时触发工作流  
on:  
    push:  
        branches:  
            - master  
  
jobs:  
    deploy:  
        # 使用最新的 Ubuntu 作为运行环境  
        runs-on: ubuntu-latest  
  
        steps:  
            # 步骤1：检出代码  
            # 这是一个 GitHub 官方的动作，用于检出仓库代码到 runner 环境  
            - name: Checkout code  
              uses: actions/checkout@v2  
  
            # 步骤2：设置 Node.js 环境  
            # 使用 actions/setup-node@v2 设置 Node.js 版本为 20.12.0            - name: Setup Node.js  
              uses: actions/setup-node@v2  
              with:  
                  node-version: '20.12.0'  
  
            # 步骤3：安装依赖项  
            # 运行 npm install 安装项目的所有依赖项  
            - name: Install dependencies  
              run: npm install  
  
            # 步骤4：构建项目  
            # 运行 npm run build 构建项目，将生成的文件放入 dist 文件夹  
            - name: Build project  
              run: npm run build  
  
            # 步骤5：设置 SSH Key            - name: Setup SSH Key  
              run: |  
                  echo "${{ secrets.SSH_PRIVATE_KEY }}" > deploy_key  
                  chmod 600 deploy_key  
  
            # 步骤6：通过 SSH 确保目标路径存在  
            # 通过 SSH 连接到服务器并创建必要的文件夹路径  
            # 如果目录已经存在也不会报错  
            - name: Ensure target directories exist  
              run: |  
                  ssh -o StrictHostKeyChecking=no -i deploy_key ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} << 'EOF'  
                  mkdir -p /docker-data/nginx/html/usercenter-frontend  
                  mkdir -p /docker-data/nginx/cert/usercenter-frontend-cert  
                  mkdir -p /docker-data/nginx/conf.d  
                  EOF  
  
            # 步骤7：通过 SSH 复制 dist 文件夹内容到服务器  
            - name: Copy dist files via SSH  
              uses: appleboy/scp-action@v0.1.1  
              with:  
                  host: ${{ secrets.SSH_HOST }}  
                  username: ${{ secrets.SSH_USER }}  
                  key: ${{ secrets.SSH_PRIVATE_KEY }}  
                  source: "dist/*"  
                  strip_components: 1 # 确保不会在目标路径中创建额外的目录层级。  
                  target: "/docker-data/nginx/html/usercenter-frontend/"  
  
            # 步骤8：通过 SSH 复制 SSL 证书到服务器  
            - name: Copy SSL certs via SSH  
              uses: appleboy/scp-action@v0.1.1  
              with:  
                  host: ${{ secrets.SSH_HOST }}  
                  username: ${{ secrets.SSH_USER }}  
                  key: ${{ secrets.SSH_PRIVATE_KEY }}  
                  source: "nginx/config.d/usercenter-frontend-cert/*"  
                  strip_components: 3  
                  target: "/docker-data/nginx/cert/usercenter-frontend-cert/"  
  
            # 步骤9：通过 SSH 复制 Nginx 配置文件到服务器  
            - name: Copy Nginx config via SSH  
              uses: appleboy/scp-action@v0.1.1  
              with:  
                  host: ${{ secrets.SSH_HOST }}  
                  username: ${{ secrets.SSH_USER }}  
                  key: ${{ secrets.SSH_PRIVATE_KEY }}  
                  source: "nginx/config.d/usercenter-frontend.conf"  
                  strip_components: 2  
                  target: "/docker-data/nginx/conf.d/"  
  
            # 步骤10：重启 Nginx 容器  
            - name: Restart Nginx container  
              run: |  
                  ssh -o StrictHostKeyChecking=no -i deploy_key ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} << 'EOF'  
                  docker restart nginx  
                  EOF
```

> 注意 不是每个前端项目都是 npm i 和 npm build的，还要需要根据项目实际情况进行修改代码
> 
> 这个项目就

提交代码运行，看看是否正确运行了

先检查，当前这些配置的最近修改时间

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/a6ebfbfe686038880371d8bef1f187ab.png)

actions执行完成后，查看最新的文件时间
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/52c40073a619e4b00057f52585360281.png)

## 5 优化

优秀的小伙伴可以发现，在执行build.yml文件的时候，npm i、build project、coyp dist 比较耗时

如下图可以看见
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/5a6a83573068b14b8f3472794035fbf6.png)

那么我们直接对这三个内容进行优化吧！

### 5.1 优化 `npm install`

- **使用缓存：** 你已经在使用 `actions/cache` 来缓存 Node.js 模块，这很好。确保缓存策略有效，可以查看是否命中缓存。
```yml
# 步骤2：缓存 Node.js 模块  
- name: Cache Node.js modules  
  uses: actions/cache@v2  
  with:  
    path: ~/.npm  
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}  
    restore-keys: |  
      ${{ runner.os }}-node-
```

- 配置npm的镜像源 **（添加了镜像源，感觉比较慢）**
```yml
# 步骤4：添加淘宝镜像源并安装依赖项  
# 运行 npm install 安装项目的所有依赖项  
- name: Add Taobao registry and install dependencies  
  run: |  
    npm config set registry https://registry.npmmirror.com  
    npm i
```

- 使用了 `yarn` 还是比较慢！（可能因为这个前端是基于ant design pro开发的，框架比较重）

### 5.2 优化构建项目

- **并行化任务：** 如果你的项目可以分块构建，考虑将构建过程并行化。
- **增量构建：** 如果你使用 Webpack，可以利用其缓存和增量构建功能来加速构建过程。
- **避免不必要的构建：** 通过检查代码变化，只在确实需要的情况下进行构建。
```yml
# 步骤5：构建项目  
# 运行 npm run build 构建项目，将生成的文件放入 dist 文件夹  
- name: Build project  
  run: |  
    export NODE_OPTIONS=--openssl-legacy-provider  
    npm run build -- --mode production --profile --json > build-stats.json

```

### 5.3 优化复制文件到服务器（感觉优化效果不大）

- **压缩文件：** 在传输前压缩文件，然后在目标服务器上解压缩，这样可以减少传输的数据量，尤其是对于大文件和大量文件非常有效。
```yml
- name: Compress dist files
  run: tar -czvf dist.tar.gz -C dist .

- name: Copy compressed dist files via SSH
  uses: appleboy/scp-action@v0.1.1
  with:
    host: ${{ secrets.SSH_HOST }}
    username: ${{ secrets.SSH_USER }}
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    source: "dist.tar.gz"
    target: "/docker-data/nginx/html/usercenter-frontend/"

- name: Decompress dist files on server
  run: |
    ssh -tt -o StrictHostKeyChecking=no -i deploy_key ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} << 'EOF'
    tar -xzvf /docker-data/nginx/html/usercenter-frontend/dist.tar.gz -C /docker-data/nginx/html/usercenter-frontend/
    EOF

```

实测，耗时比不压缩还要久，主要在解压的操作耗时很久！！！
**不建议使用**
### 5.4 完整的build.yml如下

优化点：
- 缓存Node.js模块
- 优化构建项目

对于拷贝dist文件，开始的时候会比较慢，后续就很快了，优化速度见下图：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/69d11aed130c538d1b7a5600495e90d9.png)

完整的优化代码如下：
```yml
name: Deploy usercenter-frontend  
  
# 触发条件：当代码推送到 master 分支时触发工作流  
on:  
  push:  
    branches:  
      - master  
  
jobs:  
  deploy:  
    # 使用最新的 Ubuntu 作为运行环境  
    runs-on: ubuntu-latest  
  
    steps:  
      # 步骤1：检出代码  
      # 这是一个 GitHub 官方的动作，用于检出仓库代码到 runner 环境  
      - name: Checkout code  
        uses: actions/checkout@v2  
  
      # 步骤2：缓存 Node.js 模块  
      - name: Cache Node.js modules  
        uses: actions/cache@v2  
        with:  
          path: ~/.npm  
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}  
          restore-keys: |  
            ${{ runner.os }}-node-  
  
      # 步骤3：设置 Node.js 环境  
      # 使用 actions/setup-node@v2 设置 Node.js 版本为 20.12.0      
      - name: Setup Node.js  
        uses: actions/setup-node@v2  
        with:  
          node-version: '20.12.0'  
  
      # 步骤4：运行 npm install 安装项目的所有依赖项  
      - name: Install dependencies  
        run: |  
          npm i  
  
      # 步骤5：构建项目  
      # 运行 npm run build 构建项目，将生成的文件放入 dist 文件夹  
      - name: Build project  
        run: |  
          export NODE_OPTIONS=--openssl-legacy-provider  
          npm run build -- --mode production --profile --json > build-stats.json  
  
      # 步骤6：设置 SSH Key      - name: Setup SSH Key  
        run: |  
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > deploy_key  
          chmod 600 deploy_key  
  
      # 步骤7：通过 SSH 确保目标路径存在  
      # 通过 SSH 连接到服务器并创建必要的文件夹路径  
      # 如果目录已经存在也不会报错  
      - name: Ensure target directories exist  
        run: |  
          ssh -o StrictHostKeyChecking=no -i deploy_key ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} << 'EOF'  
          mkdir -p /docker-data/nginx/html/usercenter-frontend  
          mkdir -p /docker-data/nginx/cert/usercenter-frontend-cert  
          mkdir -p /docker-data/nginx/conf.d  
          EOF  
  
      # 步骤8：通过 SSH 复制 dist 文件夹内容到服务器  
      - name: Copy dist files via SSH  
        uses: appleboy/scp-action@v0.1.1  
        with:  
          host: ${{ secrets.SSH_HOST }}  
          username: ${{ secrets.SSH_USER }}  
          key: ${{ secrets.SSH_PRIVATE_KEY }}  
          source: "dist/*"  
          strip_components: 1 # 确保不会在目标路径中创建额外的目录层级。  
          target: "/docker-data/nginx/html/usercenter-frontend/"  
  
      # 步骤9：通过 SSH 复制 SSL 证书到服务器  
      - name: Copy SSL certs via SSH  
        uses: appleboy/scp-action@v0.1.1  
        with:  
          host: ${{ secrets.SSH_HOST }}  
          username: ${{ secrets.SSH_USER }}  
          key: ${{ secrets.SSH_PRIVATE_KEY }}  
          source: "nginx/config.d/usercenter-frontend-cert/*"  
          strip_components: 3  
          target: "/docker-data/nginx/cert/usercenter-frontend-cert/"  
  
      # 步骤10：通过 SSH 复制 Nginx 配置文件到服务器  
      - name: Copy Nginx config via SSH  
        uses: appleboy/scp-action@v0.1.1  
        with:  
          host: ${{ secrets.SSH_HOST }}  
          username: ${{ secrets.SSH_USER }}  
          key: ${{ secrets.SSH_PRIVATE_KEY }}  
          source: "nginx/config.d/usercenter-frontend.conf"  
          strip_components: 2  
          target: "/docker-data/nginx/conf.d/"  
  
      # 步骤11：重启 Nginx 容器  
      - name: Restart Nginx container  
        run: |  
          ssh -o StrictHostKeyChecking=no -i deploy_key ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} << 'EOF'  
          docker restart nginx  
          EOF
```



