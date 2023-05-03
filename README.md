# devops-DZ12.5-K8S-net2

# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 2»

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к двум приложениям снаружи кластера по разным путям.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Service.
3. [Описание](https://kubernetes.io/docs/concepts/services-networking/ingress/) Ingress.
4. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment приложений backend и frontend

1. Создать Deployment приложения _frontend_ из образа nginx с количеством реплик 3 шт.
2. Создать Deployment приложения _backend_ из образа multitool.
3. Добавить Service, которые обеспечат доступ к обоим приложениям внутри кластера.
4. Продемонстрировать, что приложения видят друг друга с помощью Service.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

------

### Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера

1. Включить Ingress-controller в MicroK8S.
2. Создать Ingress, обеспечивающий доступ снаружи по IP-адресу кластера MicroK8S так, чтобы при запросе только по адресу открывался _frontend_ а при добавлении /api - _backend_.
3. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
4. Предоставить манифесты и скриншоты или вывод команды п.2.

------

### Правила приема работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------

## Ответ

Проверим установку MicroK8S

```bash
root@vkvm:/home/vk# kubectl get nodes
NAME   STATUS   ROLES    AGE   VERSION
vkvm   Ready    <none>   26d   v1.26.3
root@vkvm:/home/vk# kubectl get pod -A
NAMESPACE     NAME                                        READY   STATUS    RESTARTS        AGE
kube-system   calico-node-tl7rj                           1/1     Running   8 (3m48s ago)   26d
kube-system   kubernetes-dashboard-dc96f9fc-m949s         1/1     Running   9               26d
kube-system   dashboard-metrics-scraper-7bc864c59-7dspr   1/1     Running   8               26d
kube-system   coredns-6f5f9b5d74-c6l72                    1/1     Running   5               14d
kube-system   calico-kube-controllers-79568db7f8-jk9ck    1/1     Running   8               26d
kube-system   metrics-server-6f754f88d-qlllr              1/1     Running   9               26d
```

## Задание 1

### 1. Создать Deployment приложения _frontend_ из образа nginx с количеством реплик 3 шт

Создадим `deploy-front.yml` с развёртыванием _frontend_ , кол-во реплик = 3

<details><summary>Посмотреть YAML...</summary>

```yml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: deploy-front
  name: deploy-front
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: deploy-front
  template:
    metadata:
      labels:
        app: deploy-front
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
```

</details>

[Ссылка на deploy-front.yml](deploy-front.yml)

Запустим развёртывание  приложения _frontend_

```bash
root@vkvm:/home/vk# kubectl create -f deploy-front.yml
deployment.apps/deploy-front created
```

Проверим состояние подов  

```bash
root@vkvm:/home/vk# kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
deploy-front-65d5548d5b-f68g9   1/1     Running   0          28s
deploy-front-65d5548d5b-ql5sc   1/1     Running   0          28s
deploy-front-65d5548d5b-qnkt7   1/1     Running   0          28s
```

### 2. Создать Deployment приложения _backend_ из образа multitool

Создадим `deploy-back.yml` с развёртыванием _multitool_

<details><summary>Посмотреть YAML...</summary>

```yml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: deploy-back
  name: deploy-back
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deploy-back
  template:
    metadata:
      labels:
        app: deploy-back
    spec:
      containers:
        - name: multitool
          image: wbitt/network-multitool
          ports:
            - name: http-8080
              containerPort: 8080
              protocol: TCP
          env:
            - name: HTTP_PORT
              value: "8080"
            - name: HTTPS_PORT
              value: "11443"
```
</details>

[Ссылка на deploy-back.yml](deploy-back.yml)

Запустим развёртывание  приложения _backend_

```bash
root@vkvm:/home/vk# kubectl create -f deploy-back.yml
deployment.apps/deploy-back created
```

Проверим состояние подов

```bash
root@vkvm:/home/vk# kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
deploy-front-65d5548d5b-f68g9   1/1     Running   0          2m23s
deploy-front-65d5548d5b-ql5sc   1/1     Running   0          2m23s
deploy-front-65d5548d5b-qnkt7   1/1     Running   0          2m23s
deploy-back-65458fdbb5-g6pn2    1/1     Running   0          63s
```

### 3. Добавить Service, которые обеспечат доступ к обоим приложениям внутри кластера

Создадим `service-front.yml` с развёртыванием сервиса для _frontend_

<details><summary>Посмотреть YAML...</summary>

```yml
---
apiVersion: v1
kind: Service
metadata:
  name: service-front
spec:
  selector:
    app: deploy-front
  ports:
    - name: nginx-http
      port: 9001
      protocol: TCP
      targetPort: 80
```

</details>

[Ссылка на service-front.yml](service-front.yml)

Запустим развёртывание  сервиса 

```bash
root@vkvm:/home/vk# kubectl create -f service-front.yml
service/service-front created
```

Создадим файл `service-back.yml` с развёртыванием сервиса для _backend_

<details><summary>Посмотреть YAML...</summary>

```yml
---
apiVersion: v1
kind: Service
metadata:
  name: service-back
spec:
  selector:
    app: deploy-back
  ports:
    - name: multitool-http
      port: 9002
      protocol: TCP
      targetPort: 8080
```
</details>

[ССылка на service-back.yml](service-back.yml)

Запустим развёртывание

```bash
root@vkvm:/home/vk# kubectl create -f service-back.yml
service/service-back created
```

Проверим состояние сервисов

```bash
service/service-back created
root@vkvm:/home/vk# kubectl get service
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes      ClusterIP   10.152.183.1     <none>        443/TCP    26d
service-front   ClusterIP   10.152.183.244   <none>        9001/TCP   57s
service-back    ClusterIP   10.152.183.98    <none>        9002/TCP   17s
```

### 4. Продемонстрировать, что приложения видят друг друга с помощью Service

Проверим запуск `curl` из сервисов

```bash
root@vkvm:/home/vk# kubectl exec -it service/service-front -- curl --silent -i service-back.default.svc.cluster.local:9002 | grep Server
Server: nginx/1.20.2
root@vkvm:/home/vk# kubectl exec -it service/service-front -- curl --silent -i service-back:9002 | grep Server
Server: nginx/1.20.2
root@vkvm:/home/vk# kubectl exec -it service/service-back -- curl --silent -i service-front.default.svc.cluster.local:9001 | grep Server
Server: nginx/1.23.4
root@vkvm:/home/vk# kubectl exec -it service/service-back -- curl --silent -i service-front:9001 | grep Server
Server: nginx/1.23.4
```

Убедились, что доступ по относительному и абсолютному DNS имени сервиса работает из другого сервиса

### 5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4

## Задание 2

### 1. Включить Ingress-controller в MicroK8S

Включим ingress контроллер

```bash
root@vkvm:/home/vk# microk8s enable ingress
Infer repository core for addon ingress
Enabling Ingress
ingressclass.networking.k8s.io/public created
ingressclass.networking.k8s.io/nginx created
namespace/ingress created
serviceaccount/nginx-ingress-microk8s-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-microk8s-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-microk8s-role created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
configmap/nginx-load-balancer-microk8s-conf created
configmap/nginx-ingress-tcp-microk8s-conf created
configmap/nginx-ingress-udp-microk8s-conf created
daemonset.apps/nginx-ingress-microk8s-controller created
Ingress is enabled
```

### 2. Создать Ingress, обеспечивающий доступ снаружи по IP-адресу кластера MicroK8S так, чтобы при запросе только по адресу открывался _frontend_ а при добавлении /api - _backend_

Создадим `ingress.yml`

<details><summary>Посмотреть YAML...</summary>

```yml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service-front
            port:
              number: 9001
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: service-back
            port:
              number: 9002
```

</details>

[Ссылка на ingress.yml](ingress.yml)

Аннотация `nginx.ingress.kubernetes.io/rewrite-target: /` нужна для перезаписи URL с `/api` на `/` перед передачей запроса на внутренний сервис. Это позволяет внутренней службе обработать запросы без специальной реализации логики для `/api`.    Если удалить `rewrite-target`, то контроллер Ingress не будет переписывать путь, а будет пересылать запросы на backend сервис с префиксом `/api`. Backend сервис должен знать об этом префиксе и корректно обрабатывать запрос, иначе запрос приведёт к ошибке `404`.

Запускаем развёртывание

```bash
root@vkvm:/home/vk# kubectl create -f ingress.yml
ingress.networking.k8s.io/ingress created
```

Проверяем состояние _Ingress_

```bash
root@vkvm:/home/vk# kubectl describe ingress
Name:             ingress
Labels:           <none>
Namespace:        default
Address:          127.0.0.1
Ingress Class:    public
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /      service-front:9001 (10.1.101.247:80,10.1.101.248:80,10.1.101.249:80)
              /api   service-back:9002 (10.1.101.250:8080)
Annotations:  nginx.ingress.kubernetes.io/rewrite-target: /
Events:
  Type    Reason  Age               From                      Message
  ----    ------  ----              ----                      -------
  Normal  Sync    6s (x2 over 18s)  nginx-ingress-controller  Scheduled for sync
```

### 3. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера

Проверим IP-адреса нод

```bash
root@vkvm:/home/vk# kubectl get node -o yaml | grep IPv4Address
      projectcalico.org/IPv4Address: 172.20.0.1/16
```

Проверим запуск curl из локального компьютера

```bash
root@vkvm:/home/vk# curl  --silent -i 172.20.0.1 | grep title
<title>Welcome to nginx!</title>
root@vkvm:/home/vk# curl  --silent -i 172.20.0.1/api | grep WB
WBITT Network MultiTool (with NGINX) - deploy-back-65458fdbb5-g6pn2 - 10.1.101.250 - HTTP: 8080 , HTTPS: 11443 . (Formerly praqma/network-multitool)
```

Убедились, что доступ по адресу ноды в зависимости от префикса URL подключается либо к _frontend_ сервису, либо к _backend_ сервису

### 4. Предоставить манифесты и скриншоты или вывод команды п.2

Удалим развёрнутые ресурсы

```bash
root@vkvm:/home/vk# kubectl delete -f deploy-front.yml -f deploy-back.yml -f service-front.yml -f service-back.yml -f ingress.yml
deployment.apps "deploy-front" deleted
deployment.apps "deploy-back" deleted
service "service-front" deleted
service "service-back" deleted
ingress.networking.k8s.io "ingress" deleted
```
