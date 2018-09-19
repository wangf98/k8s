# Ingress Controller
-  Ingress Controller将基于Ingress规则将client的request直接转发到service对应的后端endpoint（即pod）上，这样会跳过kube-proxy的转发功能。
Ingres Controller以DaemonSet的形式创建，在每个node上启动以Pod hostPort的方式一个Nginx服务。
- Ingress策略: Ingress策略定义的path需要与后端真实Service的path一致，否则将会转发到一个不存在的path上。


## Nginx Ingress Controller
### Installation
- k8s Standard
  - `kubectl apply -f controller-manifests/ic-standard-mandatory.yaml`
- k8s Ingress Controller（ic1）安装
  - `kubectl apply -f controller-manifests/ic-common.yaml`
  - `kubectl apply -f controller-manifests/ic1-mandatory.yaml`
  - `kubectl apply -f controller-manifests/ic1-svc-np.yaml`
- Helm
  - `cd installation/chart`
  - `vim values.yaml`: 
    - Controller.ingressClass: nginx
  - `helm install -n nginx-ingress-controller --namespace kube-system ./` 
- Check Installation
  - `cd ../test`
  - `vim values.yaml`
    - ingress.annotations.kubernetes.io/ingress.class: nginx
    - ingress.hosts: svc0.wukong.io 
  - `helm install -n svc0 ./`
  - `kubect get ingress -o wide`: check if the backend endpoints are bound
  - `vim /etc/hosts`: svc0.wukong.io 127.0.0.1
  - ???`curl -H 'Host:svc0.wukong.io' 127.0.0.1:`: curl doesn't work correctly
- Troubleshooting
  - `kubectl exec -it -n kube-system nginx-ingress-controller-controller-57f69dc9b9-qf6gw -- /bin/bash`
  - `cat /etc/nginx/nginx.conf`
  - `kubectl exec -it -n kube-system nginx-ingress-controller-controller-57f69dc9b9-qf6gw -- cat /etc/nginx/nginx.conf`
  - `kubectl exec -it -n kube-system nginx-ingress-controller-controller-57f69dc9b9-qf6gw -- tail /var/log/nginx/error.log`

### Examples 
- [4 Scenarios](figures/kubernetes-ingress-controller-and-ingresses.png)

#### Scenario 1: 1 Ingress Controller, 1 HTTP Ingress, 1 HTTP Service
- Install Ingress controller ic1
  - `kubectl apply -f example/controller-installation/ic-common.yaml`
  - `kubectl apply -f example/controller-installation/ic1-mandatory.yaml`
  - `kubectl apply -f example/controller-installation/ic1-svc-np.yaml`
- `helm install --name ic1-svc1 ./examples/charts/svc1`: launch ingress, service and deployment
- `curl -H 'Host:svc1.tonybai.com' http://127.0.0.1:30090`

#### Scenario 2: 1 Ingress Controller, 1 HTTP Ingress, 1 HTTPS Service
- `helm install --name ic1-svc2 ./examples/charts/svc2`: launch ingress, service and deployment
- `curl -H 'Host:svc2.tonybai.com' http://127.0.0.1:30090`

#### Scenario 3: 1 Ingress Controller, 1 HTTP Ingress, 1 TCP Service
- `helm install --name ic1-svc3 ./examples/charts/svc3`: launch ingress, service and deployment
- `vim /etc/hosts`
- `telnet svc3.tonybai.com 30070`

#### Scenario 4: 2 Ingress Controller, 6 Ingress (2 HTTP, 2 HTTPS, 2 TCP)
- Install Ingress controller ic1
  - `kubectl apply -f example/controller-installation/ic2-mandatory.yaml`
  - `kubectl apply -f example/controller-installation/ic2-svc-np.yaml`
- `helm install --name ic2-svc4 ./examples/charts/svc4`
- `curl -H 'Host:svc4.tonybai.com' http://127.0.0.1:30091`
- `helm install --name ic2-svc5 ./examples/charts/svc5`
- `curl -H 'Host:svc5.tonybai.com' http://127.0.0.1:30091`
- `helm install --name ic2-svc6 ./examples/charts/svc6`
- `vim /etc/hosts`
- `telnet svc6.tonybai.com 30071`

#### Scenario 5: 1 Ingress Controller, 1 HTTPS Ingress, 1 HTTP Service
- Install Ingress controller ic3
  - `kubectl apply -f example/controller-installation/ic3-mandatory.yaml`
  - `kubectl apply -f example/controller-installation/ic3-svc-np.yaml`
- `openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ic3.key -out ic3.crt -subj "/CN=*.tonybai.com/O=tonybai.com"`
- `kubectl create secret tls ingress-controller-demo-tls-secret --key  ic3.key --cert ic3.crt`
- `helm install --name ic3-svc7 ./examples/charts/svc7`
- `curl -k -H 'Host:svc7.tonybai.com' https://127.0.0.1:30092`

#### Scenario 6: 1 Ingress Controller, 1 HTTPS Ingress, 1 HTTPS Service
- `helm install --name ic3-svc8 ./examples/charts/svc8`
- `curl -k -H 'Host:svc8.tonybai.com' https://127.0.0.1:30092`

#### Scenario 7: 1 Ingress Controller, 1 HTTPS Ingress, 1 HTTPS Service (ssl-passthrough)
ssl-passthrough这个无法通过
- `helm install --name ic3-svc9 ./examples/charts/svc9`
- `curl -k --key ./client.key --cert ./client.crt -H 'Host:svc9.tonybai.com' https://127.0.0.1:30092`


## Troubleshooting
### 503 Service Temporarily Unavailable
- 后台deployment没起，造成service无法调用deployment


## Doc
- [实践kubernetes ingress controller的四个例子](https://tonybai.com/2018/06/21/kubernetes-ingress-controller-practice-using-four-examples/)
- [HTTPS服务的Kubernetes ingress配置实践](https://tonybai.com/2018/06/25/the-kubernetes-ingress-practice-for-https-service/)
