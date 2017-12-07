创建 client 证书签名请求：

``` bash
$ cat kube-client-csr.json
{
  "CN": "kube-client",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```

绑定权限

``` bash
kubectl create rolebinding kube-client --clusterrole=admin --user=kube-client --namespace=default
``` bash