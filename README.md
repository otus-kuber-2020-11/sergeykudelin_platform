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


---
# Kubernetes - Logging

## Summary

- Get nodes in ns observability

```
➜  sergeykudelin_platform git:(kubernetes-logging) ✗ kubectl get pods -n observability         
NAME                                                     READY   STATUS    RESTARTS   AGE
alertmanager-prom-stack-kube-prometheus-alertmanager-0   2/2     Running   0          92m
elasticsearch-exporter-5b6cc9b94d-fp96r                  1/1     Running   0          126m
elasticsearch-master-0                                   1/1     Running   0          15h
elasticsearch-master-1                                   1/1     Running   0          15h
elasticsearch-master-2                                   1/1     Running   0          15h
fluent-bit-25pn5                                         1/1     Running   0          70m
fluent-bit-gkc88                                         1/1     Running   0          70m
fluent-bit-st4lh                                         1/1     Running   0          70m
fluent-bit-x9shb                                         1/1     Running   0          70m
k8s-event-logger-dd4647944-dvndv                         1/1     Running   0          13m
kibana-kibana-687f987f84-m9lnv                           1/1     Running   0          15h
loki-0                                                   1/1     Running   0          54m
loki-promtail-dt6xn                                      1/1     Running   0          54m
loki-promtail-hcvrz                                      1/1     Running   0          54m
loki-promtail-p7g2t                                      1/1     Running   0          54m
loki-promtail-vg7ft                                      1/1     Running   0          54m
prom-stack-grafana-6b48d76d64-nhg4b                      2/2     Running   0          109m
prom-stack-kube-prometheus-operator-5b8c688c7b-kd6zj     1/1     Running   0          92m
prom-stack-kube-state-metrics-6b64fdd9d9-xvgx9           1/1     Running   0          109m
prom-stack-prometheus-node-exporter-csgz2                1/1     Running   0          109m
prom-stack-prometheus-node-exporter-dfhmk                1/1     Running   0          109m
prom-stack-prometheus-node-exporter-gbk47                1/1     Running   0          109m
prom-stack-prometheus-node-exporter-phfqk                1/1     Running   0          109m
prometheus-prom-stack-kube-prometheus-prometheus-0       2/2     Running   1          92m
```
## Step by step

- Created cluster with 2 pool
```
➜  sergeykudelin_platform git:(kubernetes-logging) ✗ kubectl get nodes
NAME                                                  STATUS   ROLES    AGE   VERSION
gke-gke-observability-hw-default-pool-65570aa0-prvg   Ready    <none>   18h   v1.16.15-gke.7801
gke-gke-observability-hw-infra-pool-7a14340e-6rz5     Ready    <none>   18h   v1.16.15-gke.7801
gke-gke-observability-hw-infra-pool-7a14340e-r3x6     Ready    <none>   18h   v1.16.15-gke.7801
gke-gke-observability-hw-infra-pool-7a14340e-s053     Ready    <none>   18h   v1.16.15-gke.7801
```
- Deployed microservices-demo
```
➜  sergeykudelin_platform git:(kubernetes-logging) ✗ kubectl get pods -n microservices-demo
NAME                                     READY   STATUS    RESTARTS   AGE
adservice-cb695c556-mfh7k                1/1     Running   0          18h
cartservice-f4677b75f-b9w9b              1/1     Running   2          18h
checkoutservice-664f865b9b-ffp7r         1/1     Running   0          18h
currencyservice-bb9d998bd-bfbtq          1/1     Running   0          18h
emailservice-6756967b6d-jqgjz            1/1     Running   0          18h
frontend-766587959d-ghh6l                1/1     Running   0          18h
loadgenerator-9f854cfc5-m9qkh            1/1     Running   4          18h
paymentservice-57c87dc78b-8dvbb          1/1     Running   0          18h
productcatalogservice-9f5d68b54-szw4b    1/1     Running   0          18h
recommendationservice-57c49756fd-jfm59   1/1     Running   0          18h
redis-cart-5f75fbd9c7-482xq              1/1     Running   0          18h
shippingservice-689c6457cd-8cbld         1/1     Running   0          18h
```
- Created namespace observability
- Installed EFK
    - ElasticSearch in infra-poll with ingress
    - Kibana with ingress
    ```http://kibana.35.192.50.47.xip.io```
    - Fluent-bit
- Task #1 wt star - Didn't
- Prometheus Stack for EFK
- Check alert for elasticsearch over drain-mode
- Changed format logs for ingress-nginx
- Create dashboard for ingress
```./kubernetes-loggins/export.ndjson```
- Deployed Loki
- Move promtail for infra-pool
- Install k8s-event-logger
```helm install -n observability k8s-event-logger ./kubernetes-logging/k8s-event-logger/chart```
- Task #2 wt star - Didn't
- Task #3 wt star
    ```
    systemd:
      enabled: true
      maxEntries: 1000
      readFromTail: true
      stripUnderscores: false
      tag: host.*
    ```
## Screenshots
```./kubernetes-logging/screenshots```

---
# Kubernetes - GitOps

## Summary

- Created GKE cluster
- Created my own-repo - http://gitlab.com/sergeykudelin/microservices-demo
- Built images of microservices-demo

```
export TAG=v0.0.4 && export REPO_PREFIX=sergeykudelin && bash ./hack/make-docker-images.sh
```

- Installed FluxCD

```
➜  kubernetes-gitops git:(kubernetes-gitops) helm upgrade --install flux fluxcd/flux -f flux.values.yaml --namespace flux
```

- Installed Helm-Operator

```
➜  kubernetes-gitops git:(kubernetes-gitops) ✗ helm upgrade --install helm-operator fluxcd/helm-operator -f helm-operator.values.yaml --namespace flux
```

- Fixed error with add CRDs of Prometheus
```
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/release-0.43/example/prometheus-operator-crd/monitoring.coreos.com_alertmanagerconfigs.yaml
```

- Install PromStack

```
helm upgrade --install prometheus prometheus-community/kube-prometheus-stack -f kube-prometheus-stack/values.yaml
```

- Fix error https://github.com/prometheus-community/helm-charts/issues/579
- Installed Flagger

```
helm repo add flagger https://flagger.app

kubectl apply -f https://raw.githubusercontent.com/weaveworks/flagger/master/artifacts/flagger/crd.yaml

helm upgrade --install flagger flagger/flagger \\
--namespace=istio-system \\
--set crd.create=false \\
--set meshProvider=istio \\
--set metricsServer=http://prometheus:9090
```

- Get canaries

```
➜  kubernetes-gitops git:(kubernetes-gitops) ✗ kubectl get canaries -n microservices-demo

NAME       STATUS   WEIGHT   LASTTRANSITIONTIME
frontend   Failed   0        2021-04-08T19:26:57Z
```

- Fix canary with upgrade k8s + install istio via istioctl
- Check

```
➜  kubernetes-gitops git:(kubernetes-gitops) ✗ kubectl get canaries -n microservices-demo

NAME  STATUS   WEIGHT   LASTTRANSITIONTIME
frontend   Succeeded   0        2021-04-08T20:58:57Z
```

- Get description

```
➜  kubernetes-gitops git:(kubernetes-gitops) ✗ kubectl  describe canary frontend -n microservices-demo

Name: frontend 
Namespace: microservices-demo 
Labels: 
Annotations: helm.fluxcd.io/antecedent: microservices-demo:helmrelease/frontend 
API Version: flagger.app/v1beta1 
Kind: Canary 
Metadata: 
Creation Timestamp: 2021-04-08T21:32:13Z 
Generation: 1 
Resource Version: 18778 
Self Link: /apis/flagger.app/v1beta1/namespaces/microservices-demo/canaries/frontend 
UID: 540f68a3-1783-3533-b311-34ab884fcd89 
Spec: 
Analysis: 
Interval: 1m 
Max Weight: 100 
Metrics: 
Interval: 1m 
Name: request-success-rate 
Threshold: 99 
Step Weight: 50 

... устал форматировать лог

microservices-demo canary weight 100 Normal Synced 5m2s flagger Copying frontend.microservices-demo template spec to frontend-primary.microservices-demo Normal Synced 4m2s flagger Routing all traffic to primary Normal Synced 3m2s flagger Promotion completed! 
Scaling down frontend.microservices-demo
```