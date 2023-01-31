



```shell
# docker load -i image.tar.gz

kubectl api-versions

kubectl explain pod


kubectl log podname -c containername

kubectl exec podname  -c containername -it -- /bin/sh


回退 deployment



kubectl port-forward --address 0.0.0.0 pod/podname 8888:80


# 复制
kubectl cp podname:/pod/path /docker/data


wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

```

