# DevOps



环境：腾讯云EC

106.54.207.160



# 一、DevOps介绍



敏捷式开发

[DevOps]()，字面意思是Development &Operations的缩写，也就是开发&运维。

CI/CD:  是DevOps 实践中的核心流程，代表持续集成(Continuous Integration) 和持续交付/部署(Continuous Delivery/Deployment)





# 二、入门：配置环境



## 2.1 Git安装 (Code阶段工具)

省略。

Git学习：见Git.md





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



## 2.3 Docker/Docker-Compose安装 (Operate阶段工具)

省略。





## 2.4 Jenkins安装与配置



```bash
# 
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



















