# helm-chart

## helm的chart仓库地址为：https://velkoz1108.github.io/helm-chart

## 本Chart仓库的使用方法

1、添加chart仓库
```
# helm repo add myrepo https://velkoz1108.github.io/helm-chart
```

2、添加成功
```
# helm repo list
NAME  	URL                                   
myrepo	https://velkoz1108.github.io/helm-chart
```

3、搜索chart包
```
# helm search repo
NAME                              	CHART VERSION	APP VERSION	DESCRIPTION                                   
myrepo/test                       	0.1.0        	1.16.0     	A Helm chart for Kubernetes 
```

4、安装chart包
```
# helm install xxx myrepo/test
```

5、创建新的chart模板
```
# helm create myapp
```

6、打包
```
# helm package myapp --version 1.0.2
```

7、更新index.yaml
```
# helm repo index --url https://velkoz1108.github.io/helm-chart/ .
```

8、提交index.yaml和tgz包
```
# git add index.yaml myapp-1.0.2.tgz
# git commit -m 'upload myapp-1.0.2 chart'
# git push origin main
```
