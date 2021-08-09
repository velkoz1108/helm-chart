[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/velkoz-helm-chart-repo)](https://artifacthub.io/packages/search?repo=velkoz-helm-chart-repo)

## 如何创建自己的chart仓库

1. 创建一个自己的github仓库，如:`helm-chart`
3. 创建一个名为`gh-pages`的分支
4. 在主分支`main`的根目录下添加一个文件夹`/charts`
5. 在主分支`main`的根目录下创建`.github/workflows/release.yml`文件，文件内容如下：
```yaml
name: Release Charts

on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: Configure Git
        run: |
          git config user.name "$GITHUB_ACTOR"
          git config user.email "$GITHUB_ACTOR@users.noreply.github.com"

      - name: Run chart-releaser
        uses: helm/chart-releaser-action@v1.1.0
        env:
          CR_TOKEN: "${{ secrets.GITHUB_TOKEN }}"

```
如果你的主分支名为`master`，将`on.push.branches`改成`master`，或者增加一个分支名,如下：
```yaml
on:
  push:
    branches:
      - main
      - master
```
5. 在`gh-pages`分支添加`README.md`，增加一些本仓库的说明

6. 在此仓库的设置页面，开启github-pages,代码分支选择`gh-pages`
  - 配置完成后可以通过[https://\<githubAccount\>.github.io/\<repoName\>](https://velkoz1108.github.io/helm-chart)可以访问到`README.md`中内容
  - githubAccount指的是你的github用户名，如：velkoz1108
  - repoName指的你的仓库名，如：helm-chart

## 如何创建应用的chart并打包

1. 首先在本地拉取此仓库`main`分支，
  ```shell
  # git clone https://github.com/velkoz1108/helm-chart.git
  ```
2. 进入charts目录，
  ```shell
  # cd helm-chart/charts
  ```
3. 通过helm命令创建chart模板， 
  ```shell
  # helm create my-app
  ```
4. 按照需要修改模板
5. 提交charts下的全部内容，
  ```shell
  # git add .
  # git commit -m 'add my-app'
  # git push
  ```
6. 切换到`gh-pages`分支，检查下`index.yaml`的更新，发现已经自动给我们生成好了
7. 通过`index.yaml`中的`urls`值来下载我们的chart的tgz包，解压后查看内容是否与我们修改的一致

`index.yaml`中`my-app`的相关信息
```yaml
  my-app:
  - apiVersion: v2
    appVersion: 1.16.0
    created: "2021-08-09T05:59:43.971215302Z"
    description: A Helm chart for Kubernetes
    digest: 66d1ac2c3f68a8fe4646b5a19badd3cb255ba3542071c132804b5d6d1b1d354b
    name: my-app
    type: application
    urls:
    - https://github.com/velkoz1108/helm-chart/releases/download/my-app-0.1.0/my-app-0.1.0.tgz
    version: 0.1.0
```

## 如何使用仓库的中的包
1. 将自己的仓库添加到本地, 
  ```shell
  # helm repo add myrepo https://velkoz1108.github.io/helm-chart
  ```
2. 更新仓库，
  ```
  # helm repo update
  ```
3. 安装`my-app`,
  ```
  # helm install my-app myrepo/my-app
  ```
安装成功后会显示如下信息：
```shell
NAME: my-app
LAST DEPLOYED: Mon Aug  9 14:03:52 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=my-app,app.kubernetes.io/instance=my-app" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
```
NOTES信息来自`/charts/my-app/templates/NOTES.txt`,你可以自行修改
```shell
1. Get the application URL by running these commands:
{{- if .Values.ingress.enabled }}
{{- range $host := .Values.ingress.hosts }}
  {{- range .paths }}
  http{{ if $.Values.ingress.tls }}s{{ end }}://{{ $host.host }}{{ .path }}
  {{- end }}
{{- end }}
{{- else if contains "NodePort" .Values.service.type }}
  export NODE_PORT=$(kubectl get --namespace {{ .Release.Namespace }} -o jsonpath="{.spec.ports[0].nodePort}" services {{ include "my-app.fullname" . }})
  export NODE_IP=$(kubectl get nodes --namespace {{ .Release.Namespace }} -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
{{- else if contains "LoadBalancer" .Values.service.type }}
     NOTE: It may take a few minutes for the LoadBalancer IP to be available.
           You can watch the status of by running 'kubectl get --namespace {{ .Release.Namespace }} svc -w {{ include "my-app.fullname" . }}'
  export SERVICE_IP=$(kubectl get svc --namespace {{ .Release.Namespace }} {{ include "my-app.fullname" . }} --template "{{"{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}"}}")
  echo http://$SERVICE_IP:{{ .Values.service.port }}
{{- else if contains "ClusterIP" .Values.service.type }}
  export POD_NAME=$(kubectl get pods --namespace {{ .Release.Namespace }} -l "app.kubernetes.io/name={{ include "my-app.name" . }},app.kubernetes.io/instance={{ .Release.Name }}" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace {{ .Release.Namespace }} $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace {{ .Release.Namespace }} port-forward $POD_NAME 8080:$CONTAINER_PORT
{{- end }}
```

## 如何将自己的仓库发布在官方的ArtifactHub上
1. 登录[ArtifactHub](https://artifacthub.io/)
2. 点击我的头像，进入`Control Panel`
3. 进入Add，输入仓库地址和名称，名称如：`velkoz-helm-chart-repo`，地址如：`https://velkoz1108.github.io/helm-chart`
4. 点击保存，稍等一会儿后就可以在首页搜索到自己的仓库了
5. 点击仓库的右上角的三个点，点击`Get Badge`获取Badge,可以将仓库的badge添加到github中，这样README.md中有了一个ArtifactHub的徽章了，请看本文的第一行

### 其他的一些命令
1. 添加成功
```
# helm repo list
NAME  	URL                                   
myrepo	https://velkoz1108.github.io/helm-chart
```

2. 搜索chart包
```
# helm search repo
NAME                              	CHART VERSION	APP VERSION	DESCRIPTION                                   
myrepo/test                       	0.1.0        	1.16.0     	A Helm chart for Kubernetes 
```

3. 打包
```
# helm package myapp --version 1.0.2
```

4. 更新index.yaml
```
# helm repo index --url https://velkoz1108.github.io/helm-chart/ .
```
