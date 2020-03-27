# HARBOR

- [离线安装包下载](https://github.com/goharbor/harbor/releases)
- [GITHUB安装说明](https://github.com/goharbor/harbor/blob/release-1.10.0/docs/installation_guide.md)
- [官网安装说明](https://goharbor.io/docs/1.10/install-config/)
- https://www.cnblogs.com/hanxiaohui/p/9257855.html

1. 下载、解压online installer

   ```shell
   bash $ tar xvf harbor-online-installer-version.tgz
   ```

   

2. 修改配置文件`harbor.yml`

3. Default installation without Notary, Clair, or Chart Repository Service

   ```
    $ sudo ./install.sh
   ```

4. default password `admin` and `Harbor12345`

   ```
   cd /opt/harbor/
   docker-compose stop/start
   ```

   

5. 新建项目并推送docker image

   ```shell
   $ docker login reg.yourdomain.com
   $ docker push reg.yourdomain.com/myproject/myrepo:mytag
   ```


# Yapi 
- [docker镜像](https://hub.docker.com/r/silsuer/yapi)  v1.4.1

- [docker镜像新](https://hub.docker.com/r/mrjin/yapi)

- [安装文档](https://github.com/jinfeijie/yapi)

  修改version 和 ports

  默认密码：`ymfe.org`

  测试地址:http://39.105.193.209:5001/

  brown.xie@cwdata.com

- 安装cross request插件

  - 下载源码

  ```
  git clone https://github.com/YMFE/cross-request
  ```

  
  - 打开插件管理 `chrome://extensions/`
  - 打开开发者模式
  - 点击左边的`加载已解压扩展程序`，选择cross-request文件夹位置即可

![](https://user-gold-cdn.xitu.io/2020/2/6/17019641c66efc5a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)