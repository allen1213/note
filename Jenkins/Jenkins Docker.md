



```shell
docker pull jenkins/jenkins:lts

mkdir -p /docker/jenkins
# docker容器中jenkins用户和用户组id为1000，需要修改后目录才能映射成功，否则会报权限错误
chown -R 1000:1000 /docker/jenkins

docker run  -p 8081:8080 \
-v /docker/jenkins:/var/jenkins_home \
--name jenkins -d jenkins/jenkins:lts

# 访问 127.0.0.1:8081
# 查看登录密码
cat /docker/jenkins/secrets/initialAdminPassword
# 7a621c41881c4c9bb6a57cf7c84c2818

# 输入密码后，若出现 This Jenkins instance appears to be offline.
# 则找到该文件并修改
find / -name "hudson.model.UpdateCenter.xml"
vi /docker/jenkins/hudson.model.UpdateCenter.xml
# 将 <url>https://updates.jenkins.io/update-center.json</url>
# 修改为 <url>http://updates.jenkins.io/update-center.json</url>
# 或者改为国内镜像 <url>https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json</url>
docker restart jenkins

# 选择Install suggested plugins，若出现以下错误：
# An error occurred during installation: No such plugin: cloudbees-folder
# 则需要下载 https://updates.jenkins-ci.org/download/plugins/cloudbees-folder/ 对应版本的cloudbees-folder
# 将其放到 /docker/jenkins/war/WEB-INF/detached-plugins 




```

