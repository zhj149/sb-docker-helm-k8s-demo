# 关乎于Docker

## 目标

- 构建docker image，推送到docker registry
- 构建docker image，推入CICD
- 为helm chart准备image

## 打包镜像

参照[smaple-sb-docker-app](https://github.com/HuangMarco/sample-sb-docker-app.git)，在该应用中使用fabric8io maven plugin来完成docker image构建



## 调试

```sh
# 如果是以docker镜像方式运行Jenkins
docker exec -it <container-id> /bin/sh
cd /var/jenkins_home/workspace/
# 回退：ctrl + backspace
```
