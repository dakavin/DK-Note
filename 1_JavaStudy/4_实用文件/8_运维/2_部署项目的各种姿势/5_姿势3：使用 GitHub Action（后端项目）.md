
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