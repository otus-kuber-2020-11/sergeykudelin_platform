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

---
# Kubernetes - Vault

## Base tasks

- Add repo with consul
```
helm repo add hashicorp https://helm.releases.hashicorp.com
```
- Update registry with new repo
```
helm repo update
```
- Install consul with default variables
```
helm install consul hashicorp/consul --set global.name=consul
```
- Install vault with values.yml
```
helm install -f kubernetes-vault/values.yaml vault hashicorp/vault
```
- Result of helm status vault
```
➜  sergeykudelin_platform git:(kubernetes-vault) ✗ helm status vault
NAME: vault
LAST DEPLOYED: Tue Apr 13 20:24:39 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/


Your release is named vault. To learn more about the release, try:

  $ helm status vault
  $ helm get manifest vault
```
- Init vault with One Unseal Key
```
kubectl exec -it vault-0 -- vault operator init --key-shares=1 --key-threshold=1
```
- Save output
```
➜  sergeykudelin_platform git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault operator init --key-shares=1 --key-threshold=1

Unseal Key 1: HXWqX2/bxQHNlVW2vUQztSt3fPU8UxKPEh01tB9iudQ=

Initial Root Token: s.Il6QUUTbZm39hDoqKr88YpqF

Vault initialized with 1 key shares and a key threshold of 1. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 1 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated master key. Without at least 1 key to
reconstruct the master key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```
- Check status Vault
```
➜  sergeykudelin_platform git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault status
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       1
Threshold          1
Unseal Progress    0/1
Unseal Nonce       n/a
Version            1.7.0
Storage Type       consul
HA Enabled         true
command terminated with exit code 2
```
- Unseal all pods
```
➜  sergeykudelin_platform git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault operator unseal 'HXWqX2/bxQHNlVW2vUQztSt3fPU8UxKPEh01tB9iudQ='
Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           1
Threshold              1
Version                1.7.0
Storage Type           consul
Cluster Name           vault-cluster-1967c9f7
Cluster ID             5f543be2-c6a9-beb6-6678-83fe9364fc73
HA Enabled             true
HA Cluster             n/a
HA Mode                standby
Active Node Address    <none>

➜  sergeykudelin_platform git:(kubernetes-vault) ✗ kubectl exec -it vault-1 -- vault operator unseal 'HXWqX2/bxQHNlVW2vUQztSt3fPU8UxKPEh01tB9iudQ='
Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           1
Threshold              1
Version                1.7.0
Storage Type           consul
Cluster Name           vault-cluster-1967c9f7
Cluster ID             5f543be2-c6a9-beb6-6678-83fe9364fc73
HA Enabled             true
HA Cluster             https://vault-0.vault-internal:8201
HA Mode                standby
Active Node Address    http://10.108.2.7:8200

➜  sergeykudelin_platform git:(kubernetes-vault) ✗ kubectl exec -it vault-2 -- vault operator unseal 'HXWqX2/bxQHNlVW2vUQztSt3fPU8UxKPEh01tB9iudQ='
Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           1
Threshold              1
Version                1.7.0
Storage Type           consul
Cluster Name           vault-cluster-1967c9f7
Cluster ID             5f543be2-c6a9-beb6-6678-83fe9364fc73
HA Enabled             true
HA Cluster             https://vault-0.vault-internal:8201
HA Mode                standby
Active Node Address    http://10.108.2.7:8200
➜  sergeykudelin_platform git:(kubernetes-vault) ✗
```
- LogIn to Vault
```
➜  sergeykudelin_platform git:(kubernetes-vault) ✗ kubectl exec -it vault-0 --  vault login
Token (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                s.Il6QUUTbZm39hDoqKr88YpqF
token_accessor       iLJh0yFYKJE0M93qSZRupjnn
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```
- Save list of auth
```
➜  sergeykudelin_platform git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault auth list                                                     
Path      Type     Accessor               Description
----      ----     --------               -----------
token/    token    auth_token_d3448f05    token based credentials
```
- Get secret
```
➜  sergeykudelin_platform git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault read otus/otus-ro/config
Key                 Value
---                 -----
refresh_interval    768h
password            asajkjkahs
username            otus

➜  sergeykudelin_platform git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault kv get otus/otus-rw/config
====== Data ======
Key         Value
---         -----
password    asajkjkahs
username    otus
```
- Add k8s auth to Vault
```
➜  sergeykudelin_platform git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault auth enable kubernetes
Success! Enabled kubernetes auth method at: kubernetes/

➜  sergeykudelin_platform git:(kubernetes-vault) ✗ kubectl exec -it vault-0 --  vault auth list
Path           Type          Accessor                    Description
----           ----          --------                    -----------
kubernetes/    kubernetes    auth_kubernetes_4d19858d    n/a
token/         token         auth_token_d3448f05         token based credentials
```
- Create ClusterRoleBing & ServiceAccount
```
➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl create serviceaccount vault-auth

serviceaccount/vault-auth created

➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl apply --filename vault-auth-service-account.yml
clusterrolebinding.rbac.authorization.k8s.io/

role-tokenreview-binding created
```
- Create variables and check
```
➜  kubernetes-vault git:(kubernetes-vault) ✗ printenv | grep 'VAULT_SA_NAME\|SA_JWT_TOKEN\|SA_CA_CRT\|K8S_HOST'                                         

VAULT_SA_NAME=vault-auth-token-lw2zn
SA_JWT_TOKEN=eyJhbGciOiJSUzI1NiIsImtpZCI6IkRhMUFsNEVWc3B5NmhFQUhrbk51MndrSjR1azFSMmc3U0VueGkwOTQwWFkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6InZhdWx0LWF1dGgtdG9rZW4tbHcyem4iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoidmF1bHQtYXV0aCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjRkNzNkMTZkLTY2MzMtNDg5Zi1hYjUxLWFjZDBiZTdhMDdhYSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OnZhdWx0LWF1dGgifQ.diTCAwx9sp6ebhpIpc6csvUIVww7F4KLZ2ElAR42s1WawdP8NIG1bHcXpJTec-EvFXwj3gIVAHo36wGQ6utvAhuiiaZqEVkOTj3XGgJSLckRY6rzjIA03tAm6rsqdPLLhwFjZfXVbRcBMBLHrQP5jWvzVKDp8d-0TtHWZSVWS1B5TByHOfIOdPjmYpSLMSGrNYCDFuRTjLZyVTGOcTUAObyIAc74Z7AfI-aV51yJHKX3ev20jhapch2Qt6PkYiaZQucuAcn1ba8yUBjDJPPKDGKKr0D6ELWp1Z6AyMDtQzHeNCJBf_9okJCpu-rYK9kwBlDOzx_izrnXA0H977xwyw
SA_CA_CRT=-----BEGIN CERTIFICATE-----
K8S_HOST=https://35.188.198.154
```
- For removing ASCI color codes using
```
sed 's/\x1b\[[0-9;]*m//g'
```
-  Add config to Vault
```
➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault write auth/kubernetes/config \
token_reviewer_jwt="$SA_JWT_TOKEN" \
kubernetes_host="$K8S_HOST" \
kubernetes_ca_cert="$SA_CA_CRT"

Success! Data written to: auth/kubernetes/config
➜  kubernetes-vault git:(kubernetes-vault) ✗ 
```
- Copy policy
```
➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl cp otus-policy.hcl vault-0:/tmp/
```
- Apply Policy
```
➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault policy write otus-policy /tmp/otus-policy.hcl

Success! Uploaded policy: otus-policy

➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault write auth/kubernetes/role/otus bound_service_account_names=vault-auth bound_service_account_namespaces=default policies=otus-policy  ttl=24h             

Success! Data written to: auth/kubernetes/role/otus
```
- Create pod and setup curl & jq and get client token
```
/ # curl --request POST  --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1605  100   666  1{0     0      0      0 --:--:-- --:--:-- --:--:--     0
0     "request_id": "56508cab-cd11-2958-d2ae-e8087913b24b",
93  "lease_id": "",
9   "renewable": false,
 12  "lease_duration": 0,
80  "data": null,
7   "wrap_info": null,
 1  "warnings": null,
80  "auth": {
57     "client_token": "s.fIaOTw1LQQy1vendJcVh2V8b",
--:    "accessor": "HfOOjqjInvvk8Q3qzdPvthxf",
```
- Check read
```
/ # curl --header "X-Vault-Token:s.fIaOTw1LQQy1vendJcVh2V8b" $VAULT_ADDR/v1/otus/otus-ro/config

{"request_id":"51c662a1-eb75-24e3-deb1-57c78a1b8561","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"password":"asajkjkahs","username":"otus"},"wrap_info":null,"warnings":null,"auth":null}

/ # curl --header "X-Vault-Token:s.fIaOTw1LQQy1vendJcVh2V8b" $VAULT_ADDR/v1/otus/otus-rw/config

{"request_id":"b29f6b1a-7dac-4613-cbf8-6c340a96b5e9","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"password":"asajkjkahs","username":"otus"},"wrap_info":null,"warnings":null,"auth":null}
```
- Check write permission and fix rights in Policy with update for present kv
```
/ # curl --request POST --data '{"bar": "baz"}'   --header "X-Vault-Token:s.fIaOTw1LQQy1vendJcVh2V8b" $VAULT_ADDR/v1/otus/otus-ro/config
{"errors":["1 error occurred:\n\t* permission denied\n\n"]}
/ # "errors":["1 error occurred:\n\t* permission denied\n\n"]}

/ # curl --request POST --data '{"bar": "baz"}'   --header "X-Vault-Token:s.fIaOTw1LQQy1vendJcVh2V8b" $VAULT_ADDR/v1/otus/otus-rw/config

/ # curl --request POST --data '{"bar": "baz"}'   --header "X-Vault-Token:s.fIaOTw1LQQy1vendJcVh2V8b" $VAULT_ADDR/v1/otus/otus-rw/config1
```
- Copy vault-agent-config
```
➜  kubernetes-vault git:(kubernetes-vault) ✗ cp ../vault-guides/identity/vault-agent-k8s-demo/configmap.yaml ./
➜  kubernetes-vault git:(kubernetes-vault) ✗ cp ../vault-guides/identity/vault-agent-k8s-demo/example-k8s-spec.yaml ./
➜  kubernetes-vault git:(kubernetes-vault) ✗ mkdir configs-k8s && mv configmap.yaml ./configs-k8s/
➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl create configmap example-vault-agent-config --from-file=./configs-k8s/
➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl get configmap example-vault-agent-config -o yaml
➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl apply -f example-k8s-spec.yml --record
- Get index.html
```
➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl exec -it vault-agent-example -- cat /usr/share/nginx/html/index.html
<html>
<body>
<p>Some secrets:</p>
<ul>
<li><pre>username: otus</pre></li>
<li><pre>password: asajkjkahs</pre></li>
</ul>

</body>
</html>
```