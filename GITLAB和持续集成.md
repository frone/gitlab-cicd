# Gitlab 准备

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
   # -d：后台运行
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
   
   # 进入镜像修改仓库显示HOST
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

5. gitlab runner 安装

   ```
   curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash
   
   yum install gitlab-runner -y
   
   gitlab-runner register
   
   gitlab-runner install
   
   gitlab-runner start
   ```

   

6. dockerfile 对程序编译后打镜像

7. .gitlab-ci.yml CI/CD部署测试等

# CI/CD 相关环境准备

# 参考资料

1. [docker 安装gitlab以及使用](https://www.cnblogs.com/jackielyj/p/12009739.html)
2. [GITLAB runner 安装](https://my.oschina.net/uwith/blog/2243372)
3. [使用gitlab-ci工具快速代码编译、集成和发布](https://github.com/yangshun2005/gitlab-cicd)
4. [gitlab-runner](https://blog.csdn.net/Tri_C/article/details/85263126)