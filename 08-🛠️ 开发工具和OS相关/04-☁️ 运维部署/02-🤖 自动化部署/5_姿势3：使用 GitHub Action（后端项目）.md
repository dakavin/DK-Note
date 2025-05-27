---
文章标题: "[[5_姿势3：使用 GitHub Action（后端项目）]]" 
文章作者: Dakkk
文章概要: |
  文章详尽阐述了如何利用GitHub Actions实现Java后端项目的自动化部署。通过配置GitHub Secrets、编写YAML工作流文件，集成Maven构建、Docker容器化、Nginx配置以及SSH/SCP远程操作，实现了代码推送后自动构建、传输并部署到服务器的完整CI/CD流程。
tags:
- "GitHub Actions"
- "CI/CD"
- "自动化部署"
- "Docker"
- "Maven"
- "SSH"
- "Nginx"
- "Java后端"
相关文章:
- "[[6_姿势4：使用 Jenkins（后端）]]"
- "[[7_Part5（部署）]]"
- "[[9_总结]]"
- "[[3_姿势2：Docker部署项目]]"
- "[[8_补充：本地使用Docker安装Jenkins]]"
文章分类: "🛠️ 开发工具和OS相关"
文章路径: "08-🛠️ 开发工具和OS相关/04-☁️ 运维部署/02-🤖 自动化部署/5_姿势3：使用 GitHub Action（后端项目）.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 01:03:09
---

## 1 github设置和创建相关目录

在该项目的github仓库下设置对应的相关配置
- `SSH_HOST`: 服务器的IP地址或域名
- `SSH_USER`: SSH用户名
- `SSH_PRIVATE_KEY`: SSH私钥（可以使用`cat ~/.ssh/id_rsa`获取）

具体方法，参考上面哈，设置完成效果如下

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/5fbc265e63bc79258b659f27e9e83e51.png)




创建对应的文件夹，和编写build.yml文件（注意相关配置的路径要保持不变！）

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/17793c9a360a19cf2f171aaa5ca4bfc5.png)


## 2 编写yml文件

### 分析

工作流中，
- 步骤1：检出代码
- 步骤2：设置 Java 环境
- 步骤3：使用 Maven 构建项目，并跳过测试
- 步骤4：设置 SSH Key
- 步骤5：通过 SSH 确保目标路径存在
- 步骤6：通过 SSH 复制 Dockerfile 和 JAR 文件到服务器
- 步骤7：通过 SSH 构建 Docker 镜像
- 步骤8：通过 SSH 复制 SSL 证书到服务器
- 步骤9：通过 SSH 复制 Nginx 配置文件到服务器
- 步骤10：通过 SSH 重启 Docker 容器


修改build.yml文件
```yml
name: Deploy usercenter-backend  
  
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
      - name: Checkout code  
        uses: actions/checkout@v2  
  
      # 步骤2：设置 Java 环境  
      - name: Set up JDK 8  
        uses: actions/setup-java@v2  
        with:  
          distribution: 'adopt'  
          java-version: '8'  
  
      # 步骤3：使用 Maven 构建项目，并跳过测试  
      - name: Build with Maven  
        run: mvn clean package -DskipTests  
  
      # 步骤4：设置 SSH Key      - name: Setup SSH Key  
        run: |  
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > deploy_key  
          chmod 600 deploy_key  
  
      # 步骤5：通过 SSH 确保目标路径存在  
      - name: Ensure target directories exist  
        run: |  
          ssh -o StrictHostKeyChecking=no -i deploy_key ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} << 'EOF'  
          mkdir -p /docker-data/usercenter-backend  
          mkdir -p /docker-data/nginx/cert/usercenter-backend-cert  
          mkdir -p /docker-data/nginx/conf.d  
          EOF  
  
      # 步骤6：通过 SSH 复制 Dockerfile 和 JAR 文件到服务器  
      - name: Copy Dockerfile and JAR via SSH  
        uses: appleboy/scp-action@v0.1.1  
        with:  
          host: ${{ secrets.SSH_HOST }}  
          username: ${{ secrets.SSH_USER }}  
          key: ${{ secrets.SSH_PRIVATE_KEY }}  
          source: "Dockerfile"  
          strip_components: 0  
          target: "/docker-data/usercenter-backend/"  
      - name: Copy JAR via SSH  
        uses: appleboy/scp-action@v0.1.1  
        with:  
          host: ${{ secrets.SSH_HOST }}  
          username: ${{ secrets.SSH_USER }}  
          key: ${{ secrets.SSH_PRIVATE_KEY }}  
          source: "target/*.jar"  
          strip_components: 0  
          target: "/docker-data/usercenter-backend/"  
  
      # 步骤7：通过 SSH 构建 Docker 镜像  
      - name: Build Docker image on server  
        run: |  
          ssh -o StrictHostKeyChecking=no -i deploy_key ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} << 'EOF'  
          cd /docker-data/usercenter-backend  
          docker build -t usercenter-backend:v0.0.1 .  
          EOF  
  
      # 步骤8：通过 SSH 复制 SSL 证书到服务器  
      - name: Copy SSL certs via SSH  
        uses: appleboy/scp-action@v0.1.1  
        with:  
          host: ${{ secrets.SSH_HOST }}  
          username: ${{ secrets.SSH_USER }}  
          key: ${{ secrets.SSH_PRIVATE_KEY }}  
          source: "nginx/config.d/usercenter-backend-cert/*"  
          strip_components: 3  
          target: "/docker-data/nginx/cert/usercenter-backend-cert/"  
  
      # 步骤9：通过 SSH 复制 Nginx 配置文件到服务器  
      - name: Copy Nginx config via SSH  
        uses: appleboy/scp-action@v0.1.1  
        with:  
          host: ${{ secrets.SSH_HOST }}  
          username: ${{ secrets.SSH_USER }}  
          key: ${{ secrets.SSH_PRIVATE_KEY }}  
          source: "nginx/config.d/usercenter-backend.conf"  
          strip_components: 2  
          target: "/docker-data/nginx/conf.d/"  
  
      # 步骤10：通过 SSH 重启 Docker 容器  
      - name: Restart Docker container  
        run: |  
          ssh -o StrictHostKeyChecking=no -i deploy_key ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} << 'EOF'  
          docker stop usercenter-backend || true  
          docker rm usercenter-backend || true  
          docker run -d --name usercenter-backend \  
            -p 8080:8080 \  
            usercenter-backend:v0.0.1  
          docker restart nginx  
          EOF
```

## 测试

提交代码测试一下

看下图可以看到workflow正常运行
![后端action.gif|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/820915821349962bbc81ef457ea1f332.gif)

查看文件夹内容是否正确

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/102ed368285ba0b5d87b4c7e35df5721.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/741bc1211105fe71f59ebe26520b38e3.png)

ok，大功告成！！