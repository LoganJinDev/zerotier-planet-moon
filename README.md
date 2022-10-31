# docker一键部署私有 zerotier 构建私有plant moon 突破50台设备限制

### 前言

*   由于最近测试项目需要用到家里的`MacBook`开虚拟机，前一段用`frp` 内网穿透，但是测试项目需要转发很多端口，配置起来也很麻烦，于是乎想到了`zerotier`。
*   之前用过`zerotier`官网的服务，使用起来各种限制，于是就决定自己搭建planet和moon。
*   大概查阅了一些资料发现很多教程都不是构建真正的planet，而且配置繁琐，于是决定自己构建docker镜像。
*   镜像地址：[loganjindev/zerotier-ztncui](https://hub.docker.com/r/loganjindev/zerotier-ztncui "loganjindev/zerotier-ztncui")
*   Github地址：[https://github.com/nurnli/zerotier-planet-moon](https://github.com/nurnli/zerotier-planet-moon "https://github.com/nurnli/zerotier-planet-moon")

### 镜像简介

*   [x] 默认用户：`admin`
*   [x] 默认密码：`loganjin.cn`
*   [x] zerotier-one版本：`1.10.1`
*   [x] 基础镜像：`debian:bullseye-slim`
*   [x] 附加功能：`构建planet、构建mooon、查询配置moon方法`
*   [x] 访问地址：`http://IPaddress:3000`或`https://IPaddress:3443`
*   [x] docker仓库镜像：`loganjindev/zerotier-ztncui:latest`
*   [x] 阿里云仓库镜像：`registry.cn-shanghai.aliyuncs.com/logandev/zerotier-ztncui:latest`
*   [x] 环境变量：`MYADDR MYDOMAIN HTTP_PORT HTTPS_PORT ZTNCUI_PASSWD NODE_ENV HTTP_ALL_INTERFACES`

### 部署教程

##### 一、如未安装docker请自行安装也可参考我博客此篇文章:[快速安装Docker及配置](https://www.loganjin.cn/article/docker-install/)

##### 二、Docker命令行部署：

```shell
# 下载项目
git clone https://github.com/nurnli/zerotier-planet-moon.git
# 切换到项目目录
cd zerotier-planet-moon
# 启动命令
docker run -d \
    --net=host \
    --name zerotier-planet \
    --restart unless-stopped \
    -v ./zerotier-one:/var/lib/zerotier-one \
    -v ./ztncui/etc:/opt/key-networks/ztncui/etc \
    loganjindev/zerotier-ztncui:v1.0.0
# 参数注释
--net=host #使用主机网络
--name zerotier-planet # 容器名
--restart unless-stopped # 容器重启策略
-v ./zerotier-one:/var/lib/zerotier-one \ # 将容器/var/lib/zerotier-one挂载到主机./zerotier-one
-v ./ztncui/etc:/opt/key-networks/ztncui/etc \ # 将容器/opt/key-networks/ztncui/etc挂载到主机./ztncui/etc
```

##### 三、docker-compose部署

```shell
# 下载项目
git clone https://github.com/nurnli/zerotier-planet-moon.git
# 切换到项目目录
cd zerotier-planet-moon
# 启动命令
docker-compose up -d

#如果未指定密码,可执行 docker exec -it zerotier-planet cat /var/log/docker-ztncui.log|grep Password 获取密码
```

##### 四、构建planet和moon

```shell
# 构建 moon
docker exec -it zerotier-planet build moon
# 构建 planet
docker exec -it zerotier-planet build planet
# 查询客户端配置moon方法
docker exec -it zerotier-planet build moonid
```

##### 五、zerotier客户端配置planet

*   防火墙开放端口：`3000 3443 9993/tcp 9993/udp 3180`

*   端口简介：
    *   3000：`ztncui http访问`
    *   3443：`ztncui https访问`
    *   9993：`9993/tcp 9993/udp 均为zerotier使用 必须开放`
    *   3180：`planet moon下载端口 如果不需要可不开放`

*   windows配置planet
    *   下载planet文件：`可通过浏览器访问http://IPaddress:3180`或`通过服务器./ztncui/etc/myfs目录获取`
    *   替换planet：`将下载的planet替换C:\ProgramData\ZeroTier\One\目录下的planet `
    *   重启服务：`系统搜索 "服务" 然后找到"ZeroTier One"点 重启动 即可  `

*   linux配置planet
    *   下载planet文件：`可通过浏览器访问http://IPaddress:3180`或`通过服务器./ztncui/etc/myfs目录获取`
    *   替换planet：`将下载的planet替换/var/lib/zerotier-one目录下的planet `
    *   重启服务：`service zerotier-one restart`

*   完成以上步骤planet配置就完成了

##### 六、zerotier客户端配置moon

*   查询方法：`docker exec -it zerotier-planet build moonid`会得到以下内容
    *   `Your ZeroTier moon id is 89xxxxxxab, you could orbit moon using "zerotier-cli orbit 89xxxxxxab 89xxxxxxab"`
*   windows配置moon
    *   以管理员方式打开cmd输入`zerotier-cli orbit 89xxxxxxab 89xxxxxxab`即可
*   linux配置moon
    *   命令行输入`zerotier-cli orbit 89xxxxxxab 89xxxxxxab`即可

##### 七、创建网络及加入网络

*   浏览器打开网站：`http://IPaddress:3000`或`https://IPaddress:3443`
*   登录：`默认用户：admin 默认密码：loganjin.cn`
*   点击`Add network`输入网络名称 点击`Create Network`
*   点击`Network`查看**Network ID**
*   客户端加入网络：`zerotier-cli  join Network ID`
*   客户端常用命令

```shell
# 加入网络
zerotier-cli join Network ID
# 查看当前加入的网络列表
zerotier-cli listnetworks
# 删除网络
zerotier-cli leave Network ID
```

### 结语

*   此镜像基于[kmahyyg/ztncui-aio](https://github.com/kmahyyg/ztncui-aio)项目改造，在此特别感谢，此外已将zerotier更新至v1.10.1最新版。
*   **配置官方planet+moon** 不是很稳定有时ping会丢包，因此推荐 **配置私人planet+moon** 更稳定。
*   经测试**配置私人planet+moon**对比**只配置私人planet**ping time基本都在10-20s，可根据实际情况部署planet moon。
*   如果部署或者其他问题欢迎去[我的博客(www.loganjin.cn)](https://www.loganjin.cn/)留言或者微信公众号([Python技术交流圈](https://img-blog.csdnimg.cn/img_convert/09f3ccbb0f9231855f20b0f5fca7da16.png#pic_center))留言交流哦。
