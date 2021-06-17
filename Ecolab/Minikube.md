



```
minikube start --driver=docker \
--image-mirror-country=cn \
--image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers \
--iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.7.3.iso  \
--registry-mirror=https://hub-mirror.c.163.com
```