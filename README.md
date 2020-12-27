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

### Task04

#### Базовые задачи

- Знакомство и создание service
- Мониторинг iptables
- Активация IPVS
- Установка MetalLB
- Создание минимального Ingress

#### Задания со *

- Проброс CoreDNS через MetalLB
- Проброс Kubernetes Dashboard через Ingress
- Канареечный деплой с помощью Ingress на базе HEADER

### Task05 - Kubernetes Volumes

- Созданы манифесты для использования minio

#### Дополнительные задачи

- Добавлены манифесты для secret и перенастроен манифест Statefulset с их использованием

### Task06

- Resolve issue with deprecated repo

```➜  helm repo add stable https://kubernetes-charts.storage.googleapis.com```
```Error: repo "https://kubernetes-charts.storage.googleapis.com" is no longer available; try "https://charts.helm.sh/stable" instead```
```➜ helm repo add stable https://charts.helm.sh/stable```
```"stable" has been added to your repositories```

#### Chartmuseum

- Create namespace & install ingress-nginx estead deprecated nginx-ingress
- Check IP
```kubectl get service --namespace ingress-nginx```
- Create namespace & install certbot v1.0
```helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.1.0 --set installCRDs=true```
- Check selfsign certs
```kubectl apply -f cert-manager/test-resources.yaml```
- Create clusterissuer
```kubectl apply -f cert-manager/letsencrypt-production.yaml```
- Deploy chartmuseum
```helm upgrade --install chartmuseum stable/chartmuseum --wait --namespace=chartmuseum --version=2.13.2 -f chartmuseum/values.yaml```
- Check certs
```kubectl get certificate --namespace chartmuseum```
- Check web page

#### Chartmuseum wt star

- Use plugin to push
```helm plugin install https://github.com/chartmuseum/helm-push```
- Add repo wt creds
```helm repo add --username YOUR_USERNAME --password YOUR_PSWD cm_repo http://chartmuseum.35.202.229.251.nip.io```
- For push
```helm push YOUR_CHART cm_repo```
- For search
```helm search YOUR_CHART```
- For update
```helm repo update cm_repo```
