# DevOps



环境：腾讯云EC

106.54.207.160



# 一、DevOps介绍



敏捷式开发

[DevOps]()，字面意思是Development &Operations的缩写，也就是开发&运维。

CI/CD:  是DevOps 实践中的核心流程，代表持续集成(Continuous Integration) 和持续交付/部署(Continuous Delivery/Deployment)





# 二、入门：配置环境



## 2.1 Git安装 (Code阶段工具)

省略。这里用idea自带的git工具。





## 2.2 安装JDK和maven

```bash
wget https://wenyi-bucket1.oss-cn-hangzhou.aliyuncs.com/apache-maven-3.9.12-bin.tar.gz

wget https://wenyi-bucket1.oss-cn-hangzhou.aliyuncs.com/jdk-24.0.1_linux-x64_bin.tar.gz

tar -zxvf jdk-24.0.1_linux-x64_bin.tar.gz -C /usr/local

tar -zxvf apache-maven-3.9.12-bin.tar.gz -C /usr/local

cd /usr/local

mv jdk-24.0.1/ jdk/

mv apache-maven-3.9.12/ maven

ls
```







## 2.2 Gitlab安装



单独准备服务器，采用Docker安装

- 查看GitLab镜像

  ```sh
  docker search gitlab
  ```

- 拉取GitLab镜像

  ```sh
  docker pull gitlab/gitlab-ce
  ```

- 准备docker-compose.yml文件

  要修改：external_url 'http://106.54.207.160:8929'，这里106.54.207.160是我VM的public IP

  

  ```yml
  version: '3.1'
  services:
    gitlab:
      image: 'gitlab/gitlab-ce:latest'
      container_name: gitlab
      restart: always
      environment:
        GITLAB_OMNIBUS_CONFIG: |
          external_url 'http://106.54.207.160:8929'
          gitlab_rails['gitlab_shell_ssh_port'] = 2224
      ports:
        - '8929:8929'
        - '2224:2224'
      volumes:
        - './config:/etc/gitlab'
        - './logs:/var/log/gitlab'
        - './data:/var/opt/gitlab'
  ```

- 启动容器（需要稍等一小会……）

```bash
docker-compose up -d
```



访问GitLab首页（默认username是root）



查看root用户初始密码

```bash
docker exec -it gitlab cat /etc/gitlab/initial_root_password
```

![image-20260103130050457](./CICD.assets/image-20260103130050457.png)









## 2.3 Docker/Docker-Compose安装 (Operate阶段工具)

省略。腾讯云EC自带docker环境。





## 2.4 Jenkins安装



```bash
# 拉取jenkins镜像
docker pull jenkins/jenkins:2.516.1-lts



```



docker compose yaml文件：

```yaml
version: "3.1"
services:
  jenkins:
    image: jenkins/jenkins:2.516.1-lts
    container_name: jenkins
    ports:
      - 8080:8080
      - 50000:50000
    volumes:
      - ./data/:/var/jenkins_home/
```



启动容器：

```bash
docker-compose up -d
```





token:

```bash
glpat-I6rfOB9aOfI0RB4hLNirgW86MQp1OjEH.01.0w14361qc
```



发现权限不足：

```bash
root@jenkins:/usr/local/docker# docker logs -f jenkins
touch: cannot touch '/var/jenkins_home/copy_reference_file.log': Permission denied
Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?
touch: cannot touch '/var/jenkins_home/copy_reference_file.log': Permission denied
Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?
```



给data权限：

```bash
root@jenkins:/usr/local/docker# chmod -R 777 data
root@jenkins:/usr/local/docker# ls
data  docker-compose.yaml
```



restart: 

```bash
root@jenkins:/usr/local/docker# docker-compose restart
[+] Running 1/1
 ⠿ Container jenkins  Started 
```



重启之后，再看log，发现错误日志已经没有了：

```bash
root@jenkins:/usr/local/docker# docker logs -f jenkins

# 给出了admin密码

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

8d708db7ef8a448badbfc438ca82e0fe
```





（3）登录

稍等大概1分钟：

用上面的password登录：

![image-20220810021552048](./CICD.assets/image-20220810021552048.png)





安装plugins：选 Select plugins to install

![image-20220810021638145](./CICD.assets/image-20220810021638145.png)





安装插件：插件可能安装失败，失败的话可以之后再手动安装。我这里全部成功了。

![image-20220810021832210](./CICD.assets/image-20220810021832210.png)





全部选择root：

![image-20220810022747606](./CICD.assets/image-20220810022747606.png)



选默认url



成功： 

![image-20220810022852923](./CICD.assets/image-20220810022852923.png)



## 2.5 安装plugin

确认几个plugin，然后点击Install without restart

1、Git Parameter

2、Publish Over SSH





## 2.6 配置Jenkins

1）

```bash
# 把jdk和maven复制到data目录下
root@jenkins:/usr/local/docker/data# cp -r /usr/local/jdk ./
root@jenkins:/usr/local/docker/data# cp -r /usr/local/maven ./
root@jenkins:/usr/local/docker/data#
```



（2）指定 Jenkins 的 jdk目录

/var/jenkins_home/jdk



![image-20260103130655854](./CICD.assets/image-20260103130655854.png)

![image-20260103130717174](./CICD.assets/image-20260103130717174.png)

（3）指定 Jenkins 的 maven 目录

和前面jdk一样：/var/jenkins_home/maven

![image-20260103130752574](./CICD.assets/image-20260103130752574.png)

然后点击save







（4）准备publish

![image-20260103130858710](./CICD.assets/image-20260103130858710.png)



滑到最下面，可以找到插件：**Publish over SSH**

![image-20260103131031978](./CICD.assets/image-20260103131031978.png)

![image-20260103131052902](./CICD.assets/image-20260103131052902.png)



测试连接成功：

![image-20260103131121404](./CICD.assets/image-20260103131121404.png)





Apply then Save：

![image-20220814235445188](./CICD.assets/image-20220814235445188.png)







# 三、Jenkins pull project



## 3.1 create new item

![image-20260103131231178](./CICD.assets/image-20260103131231178.png)







## 3.2 选择Freestyle





## 3.3 找到Source Code Management

填入Respository URL。这里用的gitlab，所以要用gitlab的登陆用户名和密码。root和前面“docker exec -it gitlab cat /etc/gitlab/initial_root_password”看到的密码。



![image-20260103135237852](./CICD.assets/image-20260103135237852.png)







## 3.4 开始build



![image-20260103135317655](./CICD.assets/image-20260103135317655.png)





查看日志：

![image-20260103135440655](./CICD.assets/image-20260103135440655.png)





发现build成功：

![image-20260103135844416](./CICD.assets/image-20260103135844416.png)



## 3.5 查看project位置



```bash
root@jenkins:/home/azureuser# docker exec -it jenkins bash
jenkins@86216a6c0943:/$ cd ~
jenkins@86216a6c0943:~$ pwd
/var/jenkins_home
# ~ 就是/var/jenkins_home目录

jenkins@86216a6c0943:~$ ls
# 这个目录下有之前通过volume同步过来的jdk和maven。
# 还有一个目录叫：workspace。所有的工程都会放到workspace里
```



![image-20220815002818955](./CICD.assets/image-20220815002818955.png)





```bash
jenkins@86216a6c0943:~$ cd workspace/
jenkins@86216a6c0943:~/workspace$ ls
jenkinstest  jenkinstest@tmp
jenkins@86216a6c0943:~/workspace$ cd jenkinstest
jenkins@86216a6c0943:~/workspace/jenkinstest$ ls
pom.xml  src
```





# 三、Jenkins构建jar



## 1、进入Configure















