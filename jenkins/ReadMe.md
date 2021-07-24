# Jenkins

## Installation

[Installing Jenkins](https://www.jenkins.io/doc/book/installing/)

### Java Installation

```sh
# https://www.jenkins.io/doc/book/installing/linux/#debianubuntu

sudo apt update

# search all packages related with openjdk
sudo apt search openjdk

# Because there is only openjdk-8-jdk available, so that install it
sudo apt install openjdk-8-jdk

```

### Install Docker using the repository - Ubuntu

```sh
# https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository

sudo apt-get update

# install packages to allow apt to use a repository over HTTPS

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# set up the stable repository
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


# Install the Docker engine
# Update the apt package index, and install the latest version of Docker Engine and containerd
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# Test the docker installation
docker

```

### Install Jenkins via Docker

```sh
# Create a bridge network in Docker using the following docker network create command:
# Bridge network: https://docs.docker.com/network/bridge/
# Docker network create: https://docs.docker.com/engine/reference/commandline/network_create/
docker network create jenkins

# query all docker networks
root@iZbp18ily7toc6ikp5rtktZ:~# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
c7129c314b52   bridge    bridge    local
28d2f808c15f   host      host      local
8d6a3649cdfb   jenkins   bridge    local
f435464dd567   none      null      local
root@iZbp18ily7toc6ikp5rtktZ:~#


# 安装docker:dind image - 为了在后面可以在jenkins nodes中执行docker command

docker run \
  --name jenkins-docker \
  --rm \
  --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind \
  --storage-driver overlay2

# --rm: 当container停止的时候自动移除image  --optional
# --detach: 以后台模式运行docker container  --optional
# --privileged: 当前kernel版本还需要privileged access，当kernel版本更新的时候，该选项不再需要  --optional
# --network jenkins：使用上一步创建的bridge network jenkins
# --network-alias docker: 让docker在jenkins network中以hostname docker可以被识别到
# --env DOCKER_TLS_CERTDIR=/certs： 设置环境变量，指向Docker TLS证书的路径
# --volume jenkins-docker-certs:/certs/client： 将路径/certs/client匹配到volume jenkins-docker-certs
# --volume jenkins-data:/var/jenkins_home: 将路径/var/jenkins_home匹配到volume jenkins-data
# --publish 2376:2376: 将docker daemon 端口2376暴露在host machine 2376端口，这样在host machine command line就可以通过docker command控制daemon --optional
# docker:dind: 指定image
# --storage-driver overlay2: docker volume storage driver,更多可选的driver: https://docs.docker.com/storage/storagedriver/select-storage-driver/




# 自定义official Jenkins Docker image
# 使用附件Dockerfile来build docker image
# 其中myjenkins-blueocean名称可自定义
# 整个build过程大概持续10分钟左右，且要build多次
docker build -t myjenkins-blueocean:1.1 .




# 使用build的image运行jenkins容器

# --name jenkins-blueocean：给docker container name命名为jenkins-blueocean --optional
# --rm: 自动删除container当container停止的时候， --optional
# --detach: detach模式运行
# --network jenkins： 不再赘述
# --env DOCKER_HOST=tcp://docker:2376： 设定环境变量DOCKER_HOST，这样在前面下载的docker:dind就可以根据该环境变量接入到jenkins docker container
# --publish 8080:8080: 将本container 8080端口映射到host machine 8080端口上,第一个是host machine port，后面一个是container port
# --publish 50000:50000：将本container 50000端口映射到host machine 50000端口上 --optinal, 只有当你需要有多个jenkins agent的时候，才需要指定该选项
docker run \
  --name jenkins-blueocean \
  --rm \
  --detach \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:1.1 


docker run --name jenkins-blueocean --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:1.1


# UI访问jenkins:其中121.41.128.117是host machine的ip
http://121.41.128.117:8080/

# attach to jenkins container
docker exec -it jenkins-blueocean /bin/bash
# 获取该安全密码，该安全密码要用于网页登录
ls /var/jenkins_home/secrets/initialAdminPassword

# output: 001702e388234e17b5c1186120b6f26b

jenkins user: admin/123456
fullname: marcoroot

```

## Jenkins配置

### 添加新pipeline

![new pipeline](pic/new-task.png)

点击New Item >> Multibranch Pipeline

![new pipeline](pic/new-pipeline-2.png)

### 设置git hub credentials

![github credentials](pic/github-credentials.png)

如图，本例使用的是Github：

- 设置仓库的git地址
- 点击`添加`，选择credentials所影响的层级，根级别即整个jenkins级别，亦可选择本project级别，点击之后，将进入凭据设置页面
- 进入凭据设置页面之后，如图所示，当前对于Github，如果是public项目，可以不用设置凭据，如果是私有项目，凭据设置为：用户名(Github登录用户名)+token，其中[token创建](https://github.com/settings/tokens)，token的权限只要在`repo`即可。( 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),注意：token不能包含空格，拷贝的时候注意。
- 输入用户名+token之后，jenkins会自动验证
- 设置完毕之后，点击验证，页面中应当不提示任何错误，即可认为凭据正确。

### 设置jenkins-pr

![github pr](pic/git-pr.png)

这里选择`仅仅具有拉取请求的分支`

### 保留旧流水线天数

![old pipeline](pic/old-pipeline.png)

### Jenkinsfile

![jenkins file](pic/Jenkinsfile.png)

如图，build configuration，是指使用哪个脚本文件来指示jenkins进行流水线build作业。默认是扫描branch分支下的`Jenkinsfile`文件。所以仓库中的Jenkinsfile必须要存在

## 拓展

[docker tutorial - Use the default bridge network](https://docs.docker.com/network/network-tutorial-standalone/#use-the-default-bridge-network)
[docker - Networking with standalone containers](https://docs.docker.com/network/network-tutorial-standalone/)
[Jenkins主动拉取GitHub代码编译并发送邮件]()
[Jenkins触发远程构建]()