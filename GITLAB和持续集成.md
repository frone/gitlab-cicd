# Gitlab 和 CICD

## 硬件需求

至少双核+4G内存，推荐最低服务器配置双核+8G内存。

4G内存的服务器安装Gitlab基本上就卡死了

## Gitlab安装准备

1. docker 安装
```shell
   sudo yum install docker
   sudo yum install docker-compose
```
2. Gitlab 镜像

   docker pull gitlab/gitlab-ce

3. 运行

   ```shell
   $ docker run -d  -p 6600:443 -p 6601:80 -p 6602:22 --name gitlab --restart always -v /home/gitlab/config:/etc/gitlab -v /home/gitlab/logs:/var/log/gitlab -v /home/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce
   # -d：11
   # -p：将容器内部端口向外映射
   # --name：命名容器名称
   # -v：将容器内数据文件夹或者日志、配置等文件夹挂载到宿主机指定目录
   ```

4. 配置网址、端口

   ```shell
   # 映射文件
   vim /home/gitlab/config/gitlab.rb
       gitlab_rails['registry_host'] = "xx.xxx.xxx.xxx"
       gitlab_rails['registry_port'] = "222"
   
   # 进入镜像
   docker exec -it 22e1cc625ff8 bash
   vi /opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml
     # 1. GitLab app settings
     # ==========================
   
     ## GitLab settings
     gitlab:
       ## Web server settings (note: host is the FQDN, do not include http://)
       host: 10.119.116.160
       port: 8181
       https: false
       
   docker exec gitlab gitlab-ctl reconfigure
   ```
   
5. 配置https

   1. 修改配置文件

   ```shell
    [xieshuang@VM_177_101_centos gitlab]$ sudo vim /etc/gitlab/gitlab.rb
    
    #13行的 http >> https
    external_url 'https://ip:port'
    
    #修改nginx配置 1100行
    nginx['redirect_http_to_https'] =true
    nginx['ssl_certificate'] = "/etc/gitlab/ssl/server.crt"
    nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/server.key"
   ```

   2. 生成密钥与证书

   ```shell
   #秘钥脚本，将以下内容保存为shell脚本，然后运行
   #出现提示输入信息的地方输入信息，先输入域名然后4次证书密码，任意密码，四次保持一致。
   
   #!/bin/sh
   
   # create self-signed server certificate:
    
   read -p "Enter your domain [139.199.125.93]: " DOMAIN
    
   echo "Create server key..."
    
   openssl genrsa -des3 -out $DOMAIN.key 1024
   
   echo "Create server certificate signing request..."
    
   SUBJECT="/C=US/ST=Mars/L=iTranswarp/O=iTranswarp/OU=iTranswarp/CN=$DOMAIN"
    
   openssl req -new -subj $SUBJECT -key $DOMAIN.key -out $DOMAIN.csr
    
   echo "Remove password..."
    
   mv $DOMAIN.key $DOMAIN.origin.key
   openssl rsa -in $DOMAIN.origin.key -out $DOMAIN.key
    
   echo "Sign SSL certificate..."
    
   openssl x509 -req -days 3650 -in $DOMAIN.csr -signkey $DOMAIN.key -out $DOMAIN.crt
    
   echo "TODO:"
   echo "Copy $DOMAIN.crt to /etc/nginx/ssl/$DOMAIN.crt"
   echo "Copy $DOMAIN.key to /etc/nginx/ssl/$DOMAIN.key"
   echo "Add configuration in nginx:"
   echo "server {"
   echo "    ..."
   echo "    listen 443 ssl;"
   echo "    ssl_certificate     /etc/nginx/ssl/$DOMAIN.crt;"
   echo "    ssl_certificate_key /etc/nginx/ssl/$DOMAIN.key;"
   echo "}"
   ```

   3. 复制到证书

   ```shell
    #移动到相应的位置
   sudo mkdir -p /etc/gitlab/ssl
   sudo chmod 700 /etc/gitlab/ssl/ -R
   su cp 139.199.125.93.crt /etc/gitlab/ssl/server.crt
    
   sudo cp 139.199.125.93.key /etc/gitlab/ssl/server.key
   ```

## 导出项目内容

1. settings --> advanced --> Export project

   ![](http://img.alpstudy.com//imgs/20200323150258.png)

2. 添加remote 并push

## CI/CD相关准备

1. gitlab runner 安装

```
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash

yum install gitlab-runner -y

gitlab-runner register

gitlab-runner install

gitlab-runner start
```

2. docker 相关权限修改

```shell
# 添加docker用户组和docker用户，并将用户添加到组中
groupadd -r docker
useradd -g docker -r docker -p 'password here'

# 然后将gitlab-runner添加到docker组中*
usermod -aG docker gitlab-runner

# 最后将docker服务停止，切换到docker用户再重新启动服务，gitlab-runner用户就和docker用户组一样有rw权限
service docker stop
su docker
sudo service docker start
```

3. .gitlab-ci.yml

```shell
stages:
  - deploy

docker-deploy:
  stage: deploy
# 执行Job内容
  script:
# 通过Dockerfile生成cicd-demo镜像
    - docker build -t testgolang_brown .
# 删除已经在运行的容器
    - if [ $(docker ps -aq --filter name= testgolang_brown) ]; then docker rm -f testgolang_brown;fi
# 通过镜像启动容器，并把本机8001端口映射到容器8001端口
    - docker run -d -p 5005:8001 --name testgolang_brown testgolang_brown
#   tags:
# 执行Job的服务器
    # - mbp13
#   only:
# 只有在golang分支才会执行
    # - golang
```



3. Dockerfile

```shell
# 镜像文件
FROM golang:latest
# 维修者
MAINTAINER Brown "Brown.xie@cwdata.com"

# 镜像中项目路径
WORKDIR $GOPATH/src/chinaase.com/testgolang
# 拷贝当前目录代码到镜像
COPY . $GOPATH/src/chinaase.com/testgolang
# 制作镜像
RUN go build .

# 暴露端口
EXPOSE 8001

# 程序入口
ENTRYPOINT ["./cicd_test"] 
```

## 参考资料

1. [docker 安装gitlab以及使用](https://www.cnblogs.com/jackielyj/p/12009739.html)
2. [GITLAB runner 安装](https://my.oschina.net/uwith/blog/2243372)
3. [使用gitlab-ci工具快速代码编译、集成和发布](https://github.com/yangshun2005/gitlab-cicd)
4. [gitlab-runner](https://blog.csdn.net/Tri_C/article/details/85263126)
5. [Gitlab 安装配置, 更改配置,使用自己的nginx服务](https://www.jianshu.com/p/f64ccbb660ca)
6. [GitLab添加HTTPS](https://www.cnblogs.com/xieshuang/p/8488458.html)

