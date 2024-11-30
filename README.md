# Домашнее задание к занятию Troubleshooting

### Цель задания

Устранить неисправности при деплое приложения.

### Задание. При деплое приложение web-consumer не может подключиться к auth-db. Необходимо это исправить

1. Установить приложение по команде:
```shell
kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
```
2. Выявить проблему и описать.
3. Исправить проблему, описать, что сделано.
4. Продемонстрировать, что проблема решена.

Пытаюсь запустить приложение:
```bash
user@microk8s:~/kuber-homeworks-3.5$ kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
Error from server (NotFound): error when creating "https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml": namespaces "web" not found
Error from server (NotFound): error when creating "https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml": namespaces "data" not found
Error from server (NotFound): error when creating "https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml": namespaces "data" not found
```
Необходимо создать  namespases:
```bash
user@microk8s:~/kuber-homeworks-3.5$ kubectl create namespace web
namespace/web created
user@microk8s:~/kuber-homeworks-3.5$ kubectl create namespace data
namespace/data created

```
Пробую еще раз:
```bash
user@microk8s:~/kuber-homeworks-3.5$ kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
deployment.apps/web-consumer created
deployment.apps/auth-db created
service/auth-db created
user@microk8s:~/kuber-homeworks-3.5$ kubectl get all -n data
NAME                           READY   STATUS    RESTARTS   AGE
pod/auth-db-79c4894db7-zhprl   1/1     Running   0          44m

NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/auth-db   ClusterIP   10.152.183.148   <none>        80/TCP    44m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/auth-db   1/1     1            1           44m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/auth-db-79c4894db7   1         1         1       44m
user@microk8s:~/kuber-homeworks-3.5$ kubectl get all -n web
NAME                               READY   STATUS    RESTARTS   AGE
pod/web-consumer-6ccf95f84-4hxtj   1/1     Running   0          44m
pod/web-consumer-6ccf95f84-xdwdc   1/1     Running   0          44m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web-consumer   2/2     2            2           44m

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/web-consumer-6ccf95f84   2         2         2       44m
```
Вроде создалось, посмотрим логи:
```bash
user@microk8s:~/kuber-homeworks-3.5$ kubectl logs deployments/web-consumer 
Found 2 pods, using pod/web-consumer-6ccf95f84-xdwdc
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'

user@microk8s:~/kuber-homeworks-3.5$ kubectl logs deployments/auth-db -n data
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
192.168.0.105 - - [30/Nov/2024:12:03:44 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/8.5.0" "-"
```
Проблема в деплойменте web-consumer , не резолвится имя 'auth-db'.   
Посмотрим что в yaml деплоймента:
```bash
user@microk8s:~/kuber-homeworks-3.5$ kubectl get deployments/web-consumer -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"web-consumer","namespace":"web"},"spec":{"replicas":2,"selector":{"matchLabels":{"app":"web-consumer"}},"template":{"metadata":{"labels":{"app":"web-consumer"}},"spec":{"containers":[{"command":["sh","-c","while true; do curl auth-db; sleep 5; done"],"image":"radial/busyboxplus:curl","name":"busybox"}]}}}}
  creationTimestamp: "2024-11-30T12:00:18Z"
  generation: 1
  name: web-consumer
  namespace: web
  resourceVersion: "1470784"
  uid: b37f2936-80dd-4a5b-8571-2a1e45e4cc7e
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: web-consumer
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web-consumer
    spec:
      containers:
      - command:
        - sh
        - -c
        - while true; do curl auth-db; sleep 5; done
        image: radial/busyboxplus:curl
        imagePullPolicy: IfNotPresent
        name: busybox
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 2
  conditions:
  - lastTransitionTime: "2024-11-30T12:00:26Z"
    lastUpdateTime: "2024-11-30T12:00:26Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2024-11-30T12:00:18Z"
    lastUpdateTime: "2024-11-30T12:00:26Z"
    message: ReplicaSet "web-consumer-6ccf95f84" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 2
  replicas: 2
  updatedReplicas: 2

```

Вижу что есть строка __- while true; do curl auth-db; sleep 5; done__   
Посмотрю сервисы:
```bash
user@microk8s:~/kuber-homeworks-3.5$ kubectl get service -A
NAMESPACE     NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
data          auth-db                     ClusterIP   10.152.183.148   <none>        80/TCP                   53m
default       kubernetes                  ClusterIP   10.152.183.1     <none>        443/TCP                  20d
kube-system   dashboard-metrics-scraper   ClusterIP   10.152.183.144   <none>        8000/TCP                 20d
kube-system   kube-dns                    ClusterIP   10.152.183.10    <none>        53/UDP,53/TCP,9153/TCP   20d
kube-system   kubernetes-dashboard        ClusterIP   10.152.183.205   <none>        443/TCP                  20d
kube-system   metrics-server              ClusterIP   10.152.183.174   <none>        443/TCP                  20d
```
Сервис auth-db есть, его ip-адрес 10.152.183.148    
Зайду в контейнер:
```bash
[ root@web-consumer-6ccf95f84-xdwdc:/ ]$ curl auth-db
curl: (6) Couldn't resolve host 'auth-db'
[ root@web-consumer-6ccf95f84-xdwdc:/ ]$ curl 10.152.183.148
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
Доступ по ip есть.
Пробую отрезолвить имя по ip:
```bash
[ root@web-consumer-6ccf95f84-xdwdc:/ ]$ dig 10.152.183.148
sh: dig: not found
[ root@web-consumer-6ccf95f84-xdwdc:/ ]$ nslookup 10.152.183.148
Server:    10.152.183.10
Address 1: 10.152.183.10 kube-dns.kube-system.svc.cluster.local

Name:      10.152.183.148
Address 1: 10.152.183.148 auth-db.data.svc.cluster.local
[ root@web-consumer-6ccf95f84-xdwdc:/ ]$ 
```
Необходимо поменять в манифесте __do curl auth-db_ на  __do curl auth-db.data.svc.cluster.local__   
Скачиваю манифест:
```bash
user@microk8s:~/kuber-homeworks-3.5$ wget  https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
--2024-11-30 13:07:16--  https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.110.133, 185.199.108.133, 185.199.111.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.110.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 937 [text/plain]
Saving to: ‘task.yaml’

task.yaml                                 100%[===================================================================================>]     937  --.-KB/s    in 0s      

2024-11-30 13:07:17 (81.9 MB/s) - ‘task.yaml’ saved [937/937]
```
Меняю значения в манифесте и проверяю:
```bash
user@microk8s:~/kuber-homeworks-3.5$ kubectl apply -f task.yaml 
deployment.apps/web-consumer created
deployment.apps/auth-db created
service/auth-db created
user@microk8s:~/kuber-homeworks-3.5$ kubectl get deployments.apps -A
NAMESPACE     NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
data          auth-db                     1/1     1            1           43s
kube-system   calico-kube-controllers     1/1     1            1           20d
kube-system   coredns                     1/1     1            1           20d
kube-system   dashboard-metrics-scraper   1/1     1            1           20d
kube-system   hostpath-provisioner        1/1     1            1           20d
kube-system   kubernetes-dashboard        1/1     1            1           20d
kube-system   metrics-server              1/1     1            1           20d
web           web-consumer                2/2     2            2           43s
```
Проверяю логи проблемного деплоймента  и вижу что ошибка ушла: 
```bash
user@microk8s:~/kuber-homeworks-3.5$ kubectl logs deployments/web-consumer 
Found 2 pods, using pod/web-consumer-58847d6585-264n7
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0curl: (7) Failed to connect to auth-db.data.svc.cluster.local port 80: Connection refused
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
100   612  100   612    0     0   323k      0 --:--:-- --:--:-- --:--:--  597k
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   363k      0 --:--:-- --:--:-- --:--:--  597k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

```
