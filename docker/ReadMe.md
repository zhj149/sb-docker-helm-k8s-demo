# 关乎于Docker

## 目标

- 构建docker image，推送到docker registry
- 构建docker image，推入CICD
- 为helm chart准备image

## 打包镜像

参照[smaple-sb-docker-app](https://github.com/HuangMarco/sample-sb-docker-app.git)，在该应用中使用fabric8io maven plugin来完成docker image构建

## 上传镜像到registry

上一步骤中，`sample-sb-docker-app`通过maven plugin已经将其应用打包成image,通过`docker image ls`命令查看到其image，下一步骤我们需要将该image tag之后提交到我们的docker registry中

本文利用的是docker hub public repository.其上传镜像不是很合理。

```sh
#   docker hub public repo: https://hub.docker.com/repository/docker/anbclub/cicdsbk8s
#   docker-hub-user: archnbclub
#   repository name: cicdsbk8s
#   docker hub push image to repo within an organization: https://docs.docker.com/docker-hub/repos/
#   docker hub是通过命名规则来匹配具体image会push到哪里，以样例为例：anbclub/cicdsbk8s，其中anbclub是organization, cicdsbk8s是repository name
# a. 先将本地镜像安装docker hub要求的格式打包：
# b. 将本地的image重新打上tag

$ docker tag demo:latest anbclub/cicdsbk8s:latest
$ docker image ls
REPOSITORY             TAG            IMAGE ID       CREATED        SIZE
cicdsbk8s              latest         02488bd57da9   18 hours ago   181MB
demo                   latest         02488bd57da9   18 hours ago   181MB
anbclub/cicdsbk8s      v1             02488bd57da9   18 hours ago   181MB
archnbclub/cicdsbk8s   v1             02488bd57da9   18 hours ago   181MB
busybox                latest         69593048aa3a   7 weeks ago    1.24MB
java                   8-jdk-alpine   3fd9dd82815c   4 years ago    145MB

$ docker push anbclub/cicdsbk8s:v1
The push refers to repository [docker.io/anbclub/cicdsbk8s]
b2a8b42ce4e4: Mounted from archnbclub/cicdsbk8s
a1e7033f082e: Mounted from archnbclub/cicdsbk8s
78075328e0da: Mounted from archnbclub/cicdsbk8s
9f8566ee5135: Mounted from archnbclub/cicdsbk8s
v1: digest: sha256:9e7c761956b399b6ebce7d907248046be4311f3a8ce589cc36b98e9b74be07c5 size: 1159
$

# 再次查看docker hub中的organization/repository中发现有新的v1 tag被打上标签
```

## 调试

```sh
# 如果是以docker镜像方式运行Jenkins
docker exec -it <container-id> /bin/sh
cd /var/jenkins_home/workspace/
# 回退：ctrl + backspace
```

## 参考资料

[google cloud about how to push and pull image from regsitry](https://cloud.google.com/container-registry/docs/pushing-and-pulling)
