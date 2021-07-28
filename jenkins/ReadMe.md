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

Jenkins的配置总体要考虑Git Repo与Jenkins的搭配：

- 永远要记住：Github Repo要实现单一职责原则，即一个repo只负责一个事情，例如要构建：应用打包成docker image并上传到docker registry ==> 构建helm chart ==> 安装helm chart到k8s集群，其实此时就要考虑分拆成多个git hub repo

### 单个git hub repo

一个PR:

- Integration Test由github repo本身的action完成
- 配置Jenkins，通过github webhook，针对于该git hub repo的PR做build

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

上面设置完成之后，点击`保存`保存配置

## 设置Git端

前面我们配置完成我们的pipeline之后，在这里我们需要配置完成Github仓库webhook，以便于jenkins可以获取git相关信息，从而可以创建基于分支的管道。

### 配置github repo webhook

在Github Repo UI >> Settings

![webhook](pic/webhook.png)

![add webhook](pic/add-webhook.png)

- 在这里，payload url：jenkins host url + /github-webhook/，如图所示，`http://121.41.128.117:8080`是我的jenkins地址
- 如果是仅仅希望在push代码到仓库的时候，触发jenkins构建管道，那么选择第一项，如果要定制何时何种event触发构建管道，选择第三项，然后手动选择具体的event
- 点击`Add webhook`保存webhook

![add webhook](pic/add-webhook-2.png)

可以点击上面的webhook url进行测试，默认会获取最近一次的delivery，如果webhook没有生效，点击该链接同样可以知晓具体原因。

### 测试

提交内容到dev分支，然后新建pr从dev==>main，将会触发jenkins pipeline

![successfully build](pic/success-build.png)

## 项目的CI CD

在日常开发中存在多个CI-CD的植入点：

- 开发人员开启某个branch, -- 此处可埋点，作为一些jenkins pipeline的触发，但是需求程度以及使用率并不高
- 开发人员开发完成之后，向main发起PR, --此处可埋点，其一，通过github action/TravisCI，触发编译此branch的integration test，对当前branch进行编译，确保branch代码无误，其二，integration test完成之后，pr review流程启动，pr review通过之后，该branch merge到main，此时可继续配置github action，对main分支进行监控，有push动作发生时，对当前main分支进行编译。确保main分支代码没有问题之后，webhook触发jenkins push类型的 hook，可以做很多的事情，例如：再次对main分支进行编译

以下图为例：

![github action](ext/integration-test/pic/git-action.png)

上述，为branch设置了两个触发action的动作：a. push, b. pull request到main

更多关注ext/integration-test目录

### Git hub Repo Integration test

非常重要：关于配置Github Repo Integration test，查看ext/integration-test目录

### Github 配备Jenkins Webhook

- 同一个jenkins webhook url，一个git hub repo只能配置该jenkins url一个

更多关于[github webhook](https://docs.github.com/en/developers/webhooks-and-events/webhooks/about-webhooks)

也可以查看目录ext/面向初学者的Jenkins多分支管道xxx

## 关于Jenkins pipeline tutorial

先查看[流水线入门](https://www.jenkins.io/zh/doc/book/pipeline/getting-started/), 在本例中，采用的是源码管理系统中，即使用Jenkinsfile，在日常产品环境中，很难通过UI来管理流水线。一般都是使用Jenkinsfile来构建pipeline。

查看Jenkinsfile，可以发现有多个步骤，在Jenkins中被称为片段，而片段可以通过片段生成器来生成，即在Jenkins-url后面追加`/pipeline-syntax`，最终效果类似`http://121.41.128.117:8080/pipeline-syntax/`

![stage generator](pic/stage-generator.png)

![snippet generator](pic/snippet-generator.png)

在上面同页面中，点击`步骤参考`，可以查看所有步骤的语法

[Jenkinsfile script中脚本定义-编写](https://www.jenkins.io/doc/book/pipeline/syntax/#declarative-steps)，一般情况下结合Jenkins VSCode extension和Jenkins官网script steps来编写合适的Jenkinsfile

## Jenkins Doc Extension

在VS Code中，可以通过安装插件`Jenkins Doc - Provides Jenkins documentation and xxx`, `Jenkins in VSCode`, `Jenkinsfile Support - A大多数 syntax highlighting support xxx`来协助我们日常的Jenkinsfile开发

## 拓展

[docker tutorial - Use the default bridge network](https://docs.docker.com/network/network-tutorial-standalone/#use-the-default-bridge-network)
[docker - Networking with standalone containers](https://docs.docker.com/network/network-tutorial-standalone/)
[Jenkins主动拉取GitHub代码编译并发送邮件]()
[Jenkins触发远程构建]()
[流水线入门](https://www.jenkins.io/zh/doc/book/pipeline/getting-started/)
[流水线语法](https://www.jenkins.io/zh/doc/book/pipeline/syntax/)