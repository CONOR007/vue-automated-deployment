## 前言
由于在前两篇博客中做了些前期的准备工作,所以对于那些准备工作这里就不多赘述了.
## 一. 在云服务器上安装并配置 Nginx
纯客户端渲染的应用在配置部署方面要依赖专门的web服务器,这里用到的是Nginx,如下是安装及配置nginx的各个步骤.
```bash
# yum是nginx系统中的软件管理工具 利用它下载安装nginx
yum install nginx

# 查找nginx文件
which nginx
# 查寻nginx版本号
nginx -v

# 启动 Nginx
nginx
# 重启
nginx -s reload
# 结束 
nginx -s stop

# 检查nginx下nginx.conf配置文件是否ok
cd /etc/nginx

# 查看配置文件
cat nginx.conf 
nginx.conf内容中`井号`开头的是注释
worker_connections 1024; 表示客户端连接服务端个数限制在1024个

# 备份配置文件
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.default

# 修改配置文件
vim /etc/nginx/nginx.conf
# 按esc然后把光标移动到修改的位置即可修改
这里修改server对应的:
# 配置了两个访问的域名地址
  server_name  47.242.36.24 172.24.40.234;  
# Vue项目存放的地方
  root         /home/www; 								 
# 然后按:wq 回车即可保存修改

# 测试配置文件是否有语法错误
nginx -t

# 记得在home下新建www文件
cd /home
mkdir www

# 查看错误日志
cat /var/log/nginx/error.log

# 安装git
yum install git

# 查看运行 nginx 进程的账号
ps aux | grep nginx
```

## 二. 部署[vue.js项目](https://github.com/CONOR007/vue-automated-deployment.git)

- Vue项目中package.json的name不能是中文,否则启动不了网站

- 在Vue项目根目录`新建.github/workflows/deploy.yml`并写入下面的内容

```bash
name: Publish And Deploy Demo
on:
  # 模拟发布分支 push的时候就去自动部署
  push:
    branches:
      - master

jobs:
  build-and-deploy:
    # 乌班图的系统
    runs-on: ubuntu-latest
    steps:
    # 部署到服务器
    - name: Deploy
      uses: appleboy/ssh-action@master
      #  在git账号中配置secrets并填写一些服务器信息,然后通过它获取
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.KEY }}
        port: ${{ secrets.PORT }}
        debug: true
        # 运行在服务端的命令
        script: |
        	# tmp临时目录
          cd /tmp
          # 克隆git代码 记得安装git
          git clone https://github.com/CONOR007/vue-automated-deployment.git
          cd /tmp/vue-automated-deployment
          # 运行项目中deploy.sh脚本文件
          chmod +x ./deploy.sh
          ./deploy.sh
```

- 在根目录下新建`deploy.sh`脚本文件,并输入下面的内容

```bash
#!/bin/bash

# 安装依赖
cnpm install

# 打包
npm run build

# 删除 ngnix 指向的文件夹下的文件
rm -rf /home/www/*

# 将打包好的文件复制过去
mv /tmp/vue-automated-deployment/dist/* /home/www

# 删除 clone 的代码
rm -rf /tmp/vue-automated-deployment
```

- 如果 nginx 启动失败，查看错误日志，权限问题，使用下面方式解决

```bash
# 查看错误日志
cat /var/log/nginx/error.log
cd /home/www
# 更改 www 目录的所有者
chown nginx:nginx -R .
```

