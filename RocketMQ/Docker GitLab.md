

[来源](https://mp.weixin.qq.com/s/6GyYlR9lpVcjgYmHMYLi0w)



### 下载Gitlab的Docker镜像

```shell
docker pull gitlab/gitlab-ce
```





### 开启防火墙

由于Gitlab运行在1080端口上，所以需要开放该端口，该端口号，千万不要直接关闭防火墙，否则Gitlab会无法启动

```shell
# 开启1080端口
firewall-cmd --zone=public --add-port=1080/tcp --permanent

# 重启防火墙才能生效
systemctl restart firewalld

# 查看已经开放的端口
firewall-cmd --list-ports
```







### 启动Gitlab

```shell
docker run --detach \
--publish 10443:443 --publish 1080:80 --publish 1022:22 \
--name gitlab \
--privileged=true \
--volume /mydata/gitlab/config:/etc/gitlab \
--volume /mydata/gitlab/logs:/var/log/gitlab \
--volume /mydata/gitlab/data:/var/opt/gitlab \
gitlab/gitlab-ce
```





### 访问Gitlab

`http://127.0.0.1:1080/`  由于Gitlab启动比较慢，需要耐心等待10分钟左右



可以通过命令 `docker logs gitlab -f`  动态查看容器启动日志查看 gitlab 是否已经启动完成







### Gitlab 的使用

Gitlab启动完成后第一次访问，需要重置root帐号的密码，重置完成后输入账号密码登陆，选择创建项目、创建组织、创建帐号



##### 创建组织

创建一个组织，然后在这个组织下分别创建用户和项目，这样同组织的用户就可以使用该组织下的项目

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwkLKgUhRUWnDkfy2L7J0QHG5df1oIRsV0XdWufHJtgKGrHP6MFiaWzEbzs8jick743VONYY17DYsbHg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





##### 创建用户

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwkLKgUhRUWnDkfy2L7J0QHGhlgHf0pkN3lTDhQQUmTM7KlMKhEcytC5LMGPOGggia47hiatWKuy8upw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)







##### 创建项目

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwkLKgUhRUWnDkfy2L7J0QHGMsvSmqYVbuk7M24f62D6V4w6kZGiaqJ93icfL1r5zWIV9kwpexBP2HdQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





##### 将用户分配到组织

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwkLKgUhRUWnDkfy2L7J0QHGb2aOBvLaqrZ6L3bicr11hCWZNGvoPR9ibY0jT4CBicUlA1t0sMv0JkZgQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





### clone项目

```shell
git clone http://192.168.3.101:1080/macrozheng/hello.git
```



```shell
# 进入项目工程目录
cd hello/

# 将当前修改的文件添加到暂存区
git add .

# 提交代码
git commit -m "first commit"

git pull

git push
```



创建本地分支并提交

```shell
# 切换并从当前分支创建一个dev分支
git checkout -b dev

# 将新创建的dev分支推送到远程仓库
git push origin dev
```



其他常用命令

```shell
# 切换到dev分支
git checkout dev

# 查看本地仓库文件状况
git status

# 查看本地所有分支
git branch

# 查看提交记录
git log
```

