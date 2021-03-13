# HOMEWORKS

## kubernetes-intro

---

### Обзор demo

- Установка и знакомство с Kind
- Использование kubectl

### Установка minikube на MacOS

- Dashboard
- k9s
- Разобраться почему pod-ы восстанавливаются после удаления: kube-apiserver восстаналивается как сервис в виртуальной машине minikube в отличии от coredns контролируется ReplicaSet с запуском через runtime на базе conteinerd.

### Создание образ контейнера с nginx

- Создан образ на базе nginx:1.18.0-alpine
- Добавлен конфиг с портов 8000 и root в /app
- Опубликован в DockerHub

    ```docker push sergeykudelin/otus-k8s-web:alpine```

- Разработан манифест для него

    ```./kubernetes-intro/web-pod.yaml```

- Добавлен функционал init-контейнера

### Разработка с нуля манифест для hipster-frontend с публикацией в DockerHub

- Устранены проблемы манифеста созданого на базе ad-hoc запуска и сохранены в отдельный манифест

    ```./kubernetes-intro/frontend-pod-healthy.yaml```

## kubernetes-controllers

---

### frontend-deployment

- Добавил отсутствующий selector
- RS преобразован в Deployment
- Почему pods не заехали с новой версией, так как они уже существовали и не перезапускали, а при этом: "**НЕ проверяет соответствие запущенных Podов шаблону**"

### paymentservice-deployment

- Создан манифест с Blue-Green методом обновления
- Создан манифест с Reverse Rolling Update методом обновления

### node-exporter-daemon.yaml

- Запуск на master node-ах можно сделать через tolerations + effect, согласно документации [link](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset)

## kubernetes-security

---

### Task01

- Создать Service Account bob, дать ему роль admin в рамках всего кластера
- Создать Service Account dave без доступа к кластеру

---
    kubectl create serviceaccount bob --dry-run=client -o yaml > 01-serviceaccount-bob.yaml
    kubectl create clusterrolebinding bob-rolebinding --clusterrole=admin --serviceaccount=default:bob  --dry-run=client -o yaml > 02-rolebinding-bob.yaml
    kubectl create serviceaccount dave --dry-run=client -o yaml > 03-serviceaccount-dave.yaml

### Task02

- Создать Namespace prometheus
- Создать Service Account carol в этом Namespace
- Дать всем Service Account в Namespace prometheus возможность делать get, list, watch в отношении Pods всего кластера

---
    kubectl create ns prometheus --dry-run=client -o yaml > 01-ns-prometheus.yaml
    kubectl create serviceaccount carol --namespace prometheus --dry-run=client -o yaml > 02-serviceaccount-carol.yaml
    kubectl create clusterrole pods-viewers --verb=get,list,watch --resource=pods,pods/status --dry-run=client -o yaml > 03-clusterrole-pods-viewers.yaml
    kubectl create clusterrolebinding prometheus-pods-viewers --clusterrole=pods_viewers --group=system:serviceaccount:prometheus --dry-run=client -o yaml > 04-rolebinding-prometheus-all.yaml

### Task03

- Создать Namespace dev
- Создать Service Account jane в Namespace dev
- Дать jane роль admin в рамках Namespace dev
- Создать Service Account ken в Namespace dev
- Дать ken роль view в рамках Namespace dev

---
    kubectl create ns dev --dry-run=client -o yaml > 01-ns-dev.yaml
    kubectl create serviceaccount jane --namespace dev --dry-run=client -o yaml > 02-serviceaccount-jane.yaml
    kubectl create rolebinding jane-admin --namespace=dev --clusterrole=admin --serviceaccount=dev:jane --dry-run=client -o yaml > 05-jane-rolebinding.yaml
    kubectl create serviceaccount ken --namespace dev --dry-run=client -o yaml > 06-serviceaccount-ken.yaml
    kubectl create rolebinding ken-viewer --namespace=dev --clusterrole=view --serviceaccount=dev:ken --dry-run=client -o yaml > 07-ken-rolebinding.yaml

## kubernetes-network

---

### Базовые задачи

- Знакомство и создание service
- Мониторинг iptables
- Активация IPVS
- Установка MetalLB
- Создание минимального Ingress

### Задания со *

- Проброс CoreDNS через MetalLB
- Проброс Kubernetes Dashboard через Ingress
- Канареечный деплой с помощью Ingress на базе HEADER

## Kubernetes-volumes

---

- Созданы манифесты для использования minio

### Дополнительные задачи

- Добавлены манифесты для secret и перенастроен манифест Statefulset с их использованием

## Kubernetes-templating

---

- Resolve issue with deprecated repo

```➜  helm repo add stable https://kubernetes-charts.storage.googleapis.com```
```Error: repo "https://kubernetes-charts.storage.googleapis.com" is no longer available; try "https://charts.helm.sh/stable" instead```
```➜ helm repo add stable https://charts.helm.sh/stable```
```"stable" has been added to your repositories```

### Ingress-nginx

- Create namespace & install ingress-nginx estead deprecated nginx-ingress
```helm upgrade --install ingress-nginx stable/ingress-nginx --wait --namespace=ingress-nginx --version=3.17.0```
- Check IP
```kubectl get service --namespace ingress-nginx```

### Cert-manager

- Create namespace & install certbot v1.0
```helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.1.0 --set installCRDs=true```
- Check selfsign certs
```kubectl apply -f kubernetes-templating/cert-manager/test-resources.yaml```
- Create clusterissuer
```kubectl apply -f kubernetes-templating/cert-manager/letsencrypt-production.yaml```
```kubectl apply -f kubernetes-templating/cert-manager/letsencrypt-stage.yaml```

### Chartmuseum

- Deploy chartmuseum
```helm upgrade --install chartmuseum stable/chartmuseum --wait --namespace=chartmuseum --version=2.13.2 -f kubernetes-templating/chartmuseum/values.yaml```
- Check certs
```kubectl get certificate --namespace chartmuseum```
- Check web page

### Chartmuseum wt star

- Use plugin to push
```helm plugin install https://github.com/chartmuseum/helm-push```
- Add repo wt creds
```helm repo add --username YOUR_USERNAME --password YOUR_PSWD cm_repo http://chartmuseum.146.148.97.58.nip.io```
- For push
```helm push YOUR_CHART cm_repo```
- For search
```helm search YOUR_CHART```
- For update
```helm repo update cm_repo```

### Harbor

- Установлен и корректно сконфигурированы значение values.yaml

### Helmfile

- Создан helmfile для ingress-nginx,cert-manager,harbor

### HelmChart

- Всего по немногу, уже 1:30 не до описания

### Kubecfg

- Всего по немногу, уже 1:30 не до описания

### Kustomize

- Всего по немногу, уже 1:30 не до описания

### Конец, что это было

![I finished](./kubernetes-templating/images/531060e40ad1b80c9c79595bfa6d44c8.jpg)

## Kubernetes Operators

### Создание CRD & CR for MySQL

- Создание CRD & CR без валидации
- Добавление шаблона валидации полей to CRD и проверка
- Добавление проверки на обязательные поля to CRD и проверка

```➜  deploy git:(kubernetes-operators) ✗ kubectl apply -f cr.yml```

```The MySQL "mysql-instance" is invalid: spec.password: Required value```

### Создание своего MySQL operator

- Создание виртуального окружения для разработки
- Установка зависимостей
- Создание своего оператора mysql-operator.py
- Первый чек работоспособности

```pipenv run kopf run mysql-operator.py```

Вывод:

```[2021-01-13 00:47:55,834] kopf.objects         [INFO    ] [default/mysql-instance] Handler 'mysql_on_create' succeeded.```

```[2021-01-13 00:47:55,834] kopf.objects         [INFO    ] [default/mysql-instance] Creation event is processed: 1 succeeded; 0 failed.```

- Вопрос: Почему объект создался? Ответ: Так как контроллер не знал про него ничего, а точнее не имел подписку на него.
- Второй чек работоспособности

```pipenv run kopf run mysql-operator.py```

### Проверка создание всех объектов

- Создаем custom resource

```kubectl apply -f deploy/cr.yml```

- Проверка pvc:

```kubectl get pvc```

- Наполняем БД и чекаем ее восстановление после удаления и создания

```+----+-------------+```

```| id | name        |```

```+----+-------------+```

```|  1 | some data   |```

```|  2 | some data-2 |```

```+----+-------------+```

- Сборка образа

```docker build -t sergeykudelin/k8s-mysql-operator:0.0.1 .```

```docker push sergeykudelin/k8s-mysql-operator:0.0.1```

- Запуск оператора в kubernetes
- Задачи со звездой, спасибо но нет.

![I finished](./images/tysonreaction.gif)

## Kubernetes Monitoring

---

### Prometheus Operator

- Выбираем уровень сложности "Can i play, daddy?"

![level](./kubernetes-monitoring/images/level.png)

- Установка kube-prometheus-stack через helm3

```helm upgrade --install prometheus prometheus-community/kube-prometheus-stack -f kube-prometheus-stack/values.yaml```

- Добавление ingress domain's в /etc/hosts

```192.168.64.10 alertmanager.domain.com```
```192.168.64.10 prometheus.domain.com```
```192.168.64.10 grafana.domain.com```
```192.168.64.10 web.domain.com```

- Создание нового образа web с пробросом basic_status

```docker push sergeykudelin/otus-k8s-web:wtstatus```

- Создаем и применяем новый deployment & service & ingress & servicemonitor

```for f in $(ls | grep web); do kubectl apply -f $f; done;```

- Чекаем доступность всех сервисов через Ingress и проверяем наличие метрик
- Prometheus

![prometheus](./kubernetes-monitoring/images/prometheus.png)

- Grafana

![grafana](./kubernetes-monitoring/images/grafana.png)

Next HW
---
# Preparing cluster

## Create pools

```
➜  sergeykudelin_platform git:(kubernetes-logging) ✗ kubectl get nodesNAME                                                  STATUS   ROLES    AGE    VERSION
gke-gke-observability-hw-default-pool-65570aa0-prvg   Ready    <none>   118s   v1.16.15-gke.7801
gke-gke-observability-hw-infra-pool-7a14340e-6rz5     Ready    <none>   2m1s   v1.16.15-gke.7801
gke-gke-observability-hw-infra-pool-7a14340e-r3x6     Ready    <none>   2m     v1.16.15-gke.7801
gke-gke-observability-hw-infra-pool-7a14340e-s053     Ready    <none>   2m     v1.16.15-gke.7801
```

## Get elasticsearch on infra-pool

```
➜  kubernetes-logging git:(kubernetes-logging) ✗ kubectl get pods -n observability -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP         NODE                                     NOMINATED NODE   READINESS GATES
elasticsearch-master-0   0/1     Running   0          31s     10.4.3.2   gke-cluster-1-infra-pool-767d3a91-d6jr   <none>           <none>
elasticsearch-master-1   0/1     Running   0          116s    10.4.4.2   gke-cluster-1-infra-pool-767d3a91-sbgj   <none>           <none>
elasticsearch-master-2   0/1     Running   0          4m46s   10.4.5.2   gke-cluster-1-infra-pool-767d3a91-k60k   <none>           <none>
➜  kubernetes-logging git:(kubernetes-logging) ✗ 
```

