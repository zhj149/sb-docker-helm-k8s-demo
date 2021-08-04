# Helm

## 插件的安装 (可跳过)

- Helm Skin
- Helm Intellisense
- vscode-helm

## 安装Helm3

[helm release](https://github.com/helm/helm/releases)下载helm3

```sh
curl -X GET "https://get.helm.sh/helm-v3.6.2-linux-amd64.tar.gz" -o helm-v3.6.3.tar.gz
tar -xzvf helm-v3.6.2-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/helm
```

## 创建Helm模板

```sh
# Commands:
#
#    Helm: Create Chart - Create a new chart
#    Helm: Get Release - Get a helm release from the cluster
#    Helm: Lint - Lint your chart
#    Helm: Preview Template - Open a preview window and preview how your template will render
#    Helm: Template - Run your chart through the template engine
#    Helm: Dry Run - Run a helm install --dry-run --debug on a remote cluster and get the results (NOTE: requires Tiller on the remote cluster)
#    Helm: Version - Get the Helm version
#    Helm: Insert Dependency - Insert a dependency YAML fragment
#    Helm: Dependency Update - Update a chart's dependencies
#    Helm: Package - Package a chart directory into a chart archive
#    Helm: Convert to Template - Create a template based on an existing resource or manifest
#    Helm: Convert to Template Parameter - Convert a fixed value in a template to a parameter in the values.yaml file
#

helm create sb-helm-k8s

root@marco-VirtualBox:/home/marco/workspace/helm/helm-test/sb-helm-k8s# ll
total 28
drwxr-xr-x 4 root root 4096 8月   2 14:16 ./
drwxr-xr-x 3 root root 4096 8月   2 14:16 ../
drwxr-xr-x 2 root root 4096 8月   2 14:16 charts/
-rw-r--r-- 1 root root 1147 8月   2 14:16 Chart.yaml
-rw-r--r-- 1 root root  349 8月   2 14:16 .helmignore
drwxr-xr-x 3 root root 4096 8月   2 14:16 templates/
-rw-r--r-- 1 root root 1878 8月   2 14:16 values.yaml
root@marco-VirtualBox:/home/marco/workspace/helm/helm-test/sb-helm-k8s#

# VSCode中创建helm secret：
# a. 新建secret.yaml
# b. type `secret`，弹出窗口选择secret_for_registry
# c. 格式化自动新建的片段
# d. 修改自动生成的模板，其中更改data配置项，使其指向values.yaml中的关于imagePullSecret配置
#   此步骤参考：https://helm.sh/docs/howto/charts_tips_and_tricks/#creating-image-pull-secrets
# 如果没有弹出窗口，在windows环境下 ctrl + 空格
```

查看[VS Code-Helm之于Kubernetes简约规范](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools)

- helm其实就是桥梁，一头连接着image,一头牵挂着远端的Kubernetes

## helm operation

```sh
helm list -n <k8s-namespace> -o json > installedHelm.json

APP_HELM_EXIST=$(jq '.[] | .name | contains('\"$APP_NAME\"')' installedHelm.json)

if [[ "$APP_HELM_EXIST" == "true" ]]; then
    echo "uninstall the target helm"
    helm uninstall $APP_NAME -n ${K8S_NAMESPACE} 
else
    echo "Start to install helm"
    helm install $APP_NAME "./${HELM_CHART_NAME}/" \
        -n ${K8S_NAMESPACE}  --wait \
        --set=imagePullSecret.registry="docker.io",imagePullSecret.username="archnbclub",imagePullSecret.password="test",ingress.host=${INGRESS_HOST_NAME}
fi

helm status
```

## 资料

[Helm中使用Yaml相关](https://helm.sh/docs/chart_template_guide/yaml_techniques/)
[golang template language](https://pkg.go.dev/text/template)
[Chart Development Tips and Tricks/helm creating-image-pull-secrets](https://helm.sh/docs/howto/charts_tips_and_tricks/#creating-image-pull-secrets)
[Chart template/define fullname](https://helm.sh/docs/chart_best_practices/templates/#names-of-defined-templates)
[hazelcast as helm chart reference](https://github.com/hazelcast/charts/tree/master/stable/hazelcast-enterprise)
[itnext as helm chart reference](https://itnext.io/helm-3-secrets-management-4f23041f05c3)
[werf as helm chart reference](https://werf.io/documentation/v1.2/advanced/helm/configuration/secrets.html)

[recommended label in k8s](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/)
[9 Best Practices and Examples for Working with Kubernetes Labels](https://www.replex.io/blog/9-best-practices-and-examples-for-working-with-kubernetes-labels)
