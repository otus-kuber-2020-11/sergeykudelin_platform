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
```
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
- Create CA's via Vault
```
➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault secrets tune -max-lease-ttl=87600h pki
Success! Tuned the secrets engine at: pki/

➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault write -field=certificate pki/root/generate/internal common_name="exmaple.ru"  ttl=87600h > CA_cert.crt

➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault write pki/config/urls  \
issuing_certificates="http://vault:8200/v1/pki/ca"       \
crl_distribution_points="http://vault:8200/v1/pki/crl"
Success! Data written to: pki/config/urls
```
- Create intermidiate certs
```
➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault secrets enable --path=pki_int pki
Success! Enabled the pki secrets engine at: pki_int/

➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault secrets tune -max-lease-ttl=87600h pki_int
Success! Tuned the secrets engine at: pki_int/

➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault write -format=json pki_int/intermediate/generate/internal common_name="example.ru Intermediate Authority" | jq -r '.data.csr' > pki_intermediate.csr 
```
- Add intermidiate cert to Vault
```
➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl cp pki_intermediate.csr vault-0:/tmp

➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault write -format=json pki/root/sign-intermediate csr=@/tmp/pki_intermediate.csr format=pem_bundle ttl="43800h" |  jq -r '.data.certificate' > intermediate.cert.pem

➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl cp intermediate.cert.pem vault-0:/tmp

➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault write pki_int/intermediate/set-signed certificate=@/tmp/intermediate.cert.pem

Success! Data written to: pki_int/intermediate/set-signed
```
- Create role and first cert
```
➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault write pki_int/roles/example-dot-ru allowed_domains="example.ru" allow_subdomains=true   max_ttl="720h"                                                 
Success! Data written to: pki_int/roles/example-dot-ru

➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault write pki_int/issue/example-dot-ru common_name="dev.example.ru" ttl="24h"

Key                 Value
---                 -----
ca_chain            [-----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUKasF8qh//aop41yQXExTjaNbwmIwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0yMTA0MTMyMTIxNTlaFw0yNjA0
MTIyMTIyMjlaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAK7POUAQCZyC
VzW08Z7vlgdri8LHRExfeCwiLW0iEGoJmAgEOR6WtYtTgTCAIfAajhjEKb8201UJ
grx7eY+LJkNGVcfAjx4AxPKYZNXbt6NdfbCeB2lV+aZsqHvAqSrnJI+SoFXGGlXU
pqe0E73mIwWc6pwn/LcXaFLqFvBmOPifWQTk9cYbLyd0XN7svDSpyk0D7kC/cYAz
gcV0uaUuFlTgQrBFVrJSxJHaWt7JnVYaFGm32xZWWqRN8o1Hh1TIn82fsvE0QuaG
Y/317iWy+lWaQJ+Hy6IQPO99eXYa/x5CqAPqzH3dUS7ZZkhJP/zlN3nbFXScyiG2
lsEKkngPg78CAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUrnVXw9xmfQ0YANbhv480X/WdXekwHwYDVR0jBBgwFoAU
5SkChjraY0rpE5IvvNttRHrLDREwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
egk4YqY3uMBXqQRqVlSFp6EoPr/+c8zFzJAJ2q1ci50YU8MauikVX1AN4U+gXOZW
Mi/YuGhyGkcxkXxosCzhahfuqrV7Y4rSxVededmp1v9Vd9lnRLJlMP4aabO8jm2x
k1OfXaS1BDiSyavbX0flpWK4SWzxM65uYciu+TPBBAUrc2RZfxbAMzu1cvy+i6gd
NdWkTlRXYh7JHpyl9TEg/HaLQWzl/Ayi88GSWizSbBV/mvrJPFq6a3v36YD2BfUM
WzVkRJVuTLSSKYecQCJrhJj2mLMrW+b9C6SfEp3Afjkc03ssoMwg2jxxtO9Ig3ag
bi6yfg0VbpjMCCHTpX8wag==
-----END CERTIFICATE-----]
certificate         -----BEGIN CERTIFICATE-----
MIIDYTCCAkmgAwIBAgIULL8313PXK6XmUUP84QFPye9F4KYwDQYJKoZIhvcNAQEL
BQAwLDEqMCgGA1UEAxMhZXhhbXBsZS5ydSBJbnRlcm1lZGlhdGUgQXV0aG9yaXR5
MB4XDTIxMDQxMzIxMjU0MloXDTIxMDQxNDIxMjYxMlowGTEXMBUGA1UEAxMOZGV2
LmV4YW1wbGUucnUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCb9W/h
v99rSE8eDRy45S/PCS+V6AY7kgWGfsqdtst4I59EImTBXiOHoRqGWUV1z+2NavtQ
1fBMQoFLiZzEtJFzG118bOM+bw6jKRWmcnGq92kXYYhq5+gudAzGn4xBW8meeJbU
yYs993802MJP9rqeYPKSRT/RsSRBACnHJCIBDuRI7lEAY5EafR+6lS4w4tlBeRTO
RYjZNL3+dpbhVMpUp8i+BEmTJ5+0N/JsUSdsLfCTXdD1MlaVzlg3AYpgiDHDHNdF
LOHfebCjwG+87Y1k9/7uBuD3cSIyod/NidEYJ6XY9AWdYQIQWlwFz+nBdUzjCozw
hFAGirmxcUsApDEPAgMBAAGjgY0wgYowDgYDVR0PAQH/BAQDAgOoMB0GA1UdJQQW
MBQGCCsGAQUFBwMBBggrBgEFBQcDAjAdBgNVHQ4EFgQU+3qRAIpdNo47sFjhjX0b
00C63+AwHwYDVR0jBBgwFoAUrnVXw9xmfQ0YANbhv480X/WdXekwGQYDVR0RBBIw
EIIOZGV2LmV4YW1wbGUucnUwDQYJKoZIhvcNAQELBQADggEBAEK6QNYgLHsye48M
xPk2svtmXJ2IE43qgQ7mf+ke6RerqpdfJAPpGvMXj4yV0gQpq/jrxci5qewx+CwJ
RKCxGx5iT7jW5+9HIcr8HGmL0fxsKBnX2u/Hg/CsHTNmhOkILBdYreh380TOzsWQ
GMC0Lu6PFUx6Rv8kkZSp+L6XvTQO9asyjLqKJDs2OwJiXsnpAG0m8yqQq9bBj7pu
w/tz0q069vvO7KQ0g/6ABDoml7j1RX8MnzJcoCbg/LHOIFP0sc+0FX2jehSsOe/L
vJoZCHtvGk262dw36qlDXUEtPljpv4ooH1zGqm4vCSiYq0NKSVcR+288OK/JL7Sh
GH2x/hI=
-----END CERTIFICATE-----
expiration          1618435572
issuing_ca          -----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUKasF8qh//aop41yQXExTjaNbwmIwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0yMTA0MTMyMTIxNTlaFw0yNjA0
MTIyMTIyMjlaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAK7POUAQCZyC
VzW08Z7vlgdri8LHRExfeCwiLW0iEGoJmAgEOR6WtYtTgTCAIfAajhjEKb8201UJ
grx7eY+LJkNGVcfAjx4AxPKYZNXbt6NdfbCeB2lV+aZsqHvAqSrnJI+SoFXGGlXU
pqe0E73mIwWc6pwn/LcXaFLqFvBmOPifWQTk9cYbLyd0XN7svDSpyk0D7kC/cYAz
gcV0uaUuFlTgQrBFVrJSxJHaWt7JnVYaFGm32xZWWqRN8o1Hh1TIn82fsvE0QuaG
Y/317iWy+lWaQJ+Hy6IQPO99eXYa/x5CqAPqzH3dUS7ZZkhJP/zlN3nbFXScyiG2
lsEKkngPg78CAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUrnVXw9xmfQ0YANbhv480X/WdXekwHwYDVR0jBBgwFoAU
5SkChjraY0rpE5IvvNttRHrLDREwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
egk4YqY3uMBXqQRqVlSFp6EoPr/+c8zFzJAJ2q1ci50YU8MauikVX1AN4U+gXOZW
Mi/YuGhyGkcxkXxosCzhahfuqrV7Y4rSxVededmp1v9Vd9lnRLJlMP4aabO8jm2x
k1OfXaS1BDiSyavbX0flpWK4SWzxM65uYciu+TPBBAUrc2RZfxbAMzu1cvy+i6gd
NdWkTlRXYh7JHpyl9TEg/HaLQWzl/Ayi88GSWizSbBV/mvrJPFq6a3v36YD2BfUM
WzVkRJVuTLSSKYecQCJrhJj2mLMrW+b9C6SfEp3Afjkc03ssoMwg2jxxtO9Ig3ag
bi6yfg0VbpjMCCHTpX8wag==
-----END CERTIFICATE-----
private_key         -----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEAm/Vv4b/fa0hPHg0cuOUvzwkvlegGO5IFhn7KnbbLeCOfRCJk
wV4jh6EahllFdc/tjWr7UNXwTEKBS4mcxLSRcxtdfGzjPm8OoykVpnJxqvdpF2GI
aufoLnQMxp+MQVvJnniW1MmLPfd/NNjCT/a6nmDykkU/0bEkQQApxyQiAQ7kSO5R
AGORGn0fupUuMOLZQXkUzkWI2TS9/naW4VTKVKfIvgRJkyeftDfybFEnbC3wk13Q
9TJWlc5YNwGKYIgxwxzXRSzh33mwo8BvvO2NZPf+7gbg93EiMqHfzYnRGCel2PQF
nWECEFpcBc/pwXVM4wqM8IRQBoq5sXFLAKQxDwIDAQABAoIBAQCE9XmsvCd9Duhk
dklGWB2qI+qtomGt549OWkniqzRL+BKPw8KiF9+ygWZboz/UcK/VIJ+hCsMSQKB6
BZfhGw/lUi8hJLOXRpb0AtKyVF8Tolm11TC3832+HLHHo72u+tGoiKYOQsSyz41j
QGhoQ7BV1dD3YpJF8v81ay4y2FslCngq7MN5dVezkRkqmSqEfmLqZrfh5LRgJfFP
KTdZ2OYDRlaWnJ46pIZdj8rSa/5R4AOE/vfZieHzp/1NcrLmZJUSQxLY6agxIrgO
lB1cgp1f9xImyi4uMiJh0y2QGi7ScpTrmKfIQwU4XrnL5P8HjIsPZpLjsHxi1051
yhI5/QwBAoGBAMDUqXB8vbx6O/mZDVw3xuzY4CGBQn9/TIdCoZ6wWCdXy5eqbE2T
4yZP5zrogsvI4JwqleRZ+Ej9QFZlsPSJMyQBk+EzbLpF9A3Qt6o3EuC7olBdDVbO
as1fqdhLN2vHQ0nSXE1zObVmmYjsAP+L5ljKbwkpeLMitt7+BJmO9jdXAoGBAM8M
leRxyv5fQtmdV1as+NjeJKtqyqHQjKR0vvJP/vu1/FRtF01paInNx+GVvGS89+X7
GN+XaBzbEzUu7qQYUwWWUV5VzHc9GdPc8t7P5dh9QTf1uFgNBMoHgchM6pfXoVHq
0bFE74WStzYFbvmMAv3wNLkZuVuzpUvVCPRI2VkJAoGBAL7GJtRZNUXxEMEBwQwJ
Ss8sSaIcRfPpt4biTw+2m6Bg5dWpD/k4ZLSUvMm1GyIOHNmj8CO5N0DO/QX9GbL0
whnPTcSxodIwPyIj6nGGhzC7sfwb84R8N4H0MQ8Ca1RAEbxJWHRvmRp05VVnWB17
BWu2619/HiDsKUw4t8hMfh+FAoGBAJvC9RTKApOA6NK7ipP7Rq4n2GBY054OPXAP
IAM86S9FxlFhTHGBRhK9i4yK0BLdEoWidCDpT3q92OJer0slvXdrkUUtuMdPYRnA
k7nJnzlRaXoG0irziFHQefNM4gNfRc5RoHUCzkqniEsMpWL40NtnFNLXpll1eXnm
B3l3QIO5AoGAZnWVyjmduct72phHKh42JL/9nl5bvQGjoFN5j6ThN8bea02saovE
PW4qzoy2JnsAcak5q9Cv+6FfBCsgDDImzv9ydmdLVn3m356gBHmygyhtHdhyJJCN
DMvmezJSkzlC3UB+8RGL1Ch75fwKLWFZaikXO6W/0KJgqhuNybyDl9U=
-----END RSA PRIVATE KEY-----
private_key_type    rsa
serial_number       2c:bf:37:d7:73:d7:2b:a5:e6:51:43:fc:e1:01:4f:c9:ef:45:e0:a6

➜  kubernetes-vault git:(kubernetes-vault) ✗ kubectl exec -it vault-0 -- vault write pki_int/revoke serial_number="2c:bf:37:d7:73:d7:2b:a5:e6:51:43:fc:e1:01:4f:c9:ef:45:e0:a6"
Key                        Value
---                        -----
revocation_time            1618349331
revocation_time_rfc3339    2021-04-13T21:28:51.87655956Z
```
![I did it](./kubernetes-vault/images/done.jpg)

---
# Kubernetes - CSI, Storage

## Base tasks

- Activate CSI for minikube
```
➜  sergeykudelin_platform git:(kubernetes-storage) minikube start 
😄  minikube v1.19.0 on Darwin 11.2.3
❗  Both driver=hyperkit and vm-driver=hyperkit have been set.

    Since vm-driver is deprecated, minikube will default to driver=hyperkit.

    If vm-driver is set in the global config, please run "minikube config unset vm-driver" to resolve this warning.

✨  Using the hyperkit driver based on user configuration
👍  Starting control plane node minikube in cluster minikube
💾  Downloading Kubernetes v1.20.2 preload ...
    > preloaded-images-k8s-v10-v1...: 491.71 MiB / 491.71 MiB  100.00% 4.02 MiB
🔥  Creating hyperkit VM (CPUs=2, Memory=4000MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.20.2 on Docker 20.10.4 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass

❗  /usr/local/bin/kubectl is version 1.18.6, which may have incompatibilites with Kubernetes 1.20.2.
    ▪ Want kubectl v1.20.2? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
➜  sergeykudelin_platform git:(kubernetes-storage) minikube addons enable volumesnapshots    

    ▪ Using image k8s.gcr.io/sig-storage/snapshot-controller:v4.0.0
🌟  The 'volumesnapshots' addon is enabled
➜  sergeykudelin_platform git:(kubernetes-storage) minikube addons enable csi-hostpath-driver

    ▪ Using image k8s.gcr.io/sig-storage/csi-snapshotter:v4.0.0
    ▪ Using image k8s.gcr.io/sig-storage/csi-provisioner:v2.1.0
    ▪ Using image k8s.gcr.io/sig-storage/csi-external-health-monitor-controller:v0.2.0
    ▪ Using image k8s.gcr.io/sig-storage/csi-node-driver-registrar:v2.0.1
    ▪ Using image k8s.gcr.io/sig-storage/livenessprobe:v2.2.0
    ▪ Using image k8s.gcr.io/sig-storage/csi-resizer:v1.1.0
    ▪ Using image k8s.gcr.io/sig-storage/csi-attacher:v3.1.0
    ▪ Using image k8s.gcr.io/sig-storage/csi-external-health-monitor-agent:v0.2.0
    ▪ Using image k8s.gcr.io/sig-storage/hostpathplugin:v1.6.0
🔎  Verifying csi-hostpath-driver addon...
🌟  The 'csi-hostpath-driver' addon is enabled
```
- Check SC
```
➜  sergeykudelin_platform git:(kubernetes-storage) kubectl get sc
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
csi-hostpath-sc      hostpath.csi.k8s.io        Delete          Immediate           false                  4m31s
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  5m44s
```
- Create PVC and check PV
```
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl apply -f kubernetes-storage/csi-pvc.yaml
persistentvolumeclaim/csi-pvc created
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl get pvc
NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
csi-pvc   Bound    pvc-f1b92a0b-0c70-46f1-a9d2-c4ffeb69a206   1Gi        RWO            csi-hostpath-sc   8s
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl get pv 
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS      REASON   AGE
pvc-f1b92a0b-0c70-46f1-a9d2-c4ffeb69a206   1Gi        RWO            Delete           Bound    default/csi-pvc   csi-hostpath-sc            11s
```
- Check properties of PVC
```
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl describe pvc csi-pvc
Name:          csi-pvc
Namespace:     default
StorageClass:  csi-hostpath-sc
Status:        Bound
Volume:        pvc-f1b92a0b-0c70-46f1-a9d2-c4ffeb69a206
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: hostpath.csi.k8s.io
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Mounted By:    <none>
Events:
...
```
- Create pod and create file
```
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl apply -f kubernetes-storage/csi-app.yaml 
pod/sci-pod created
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl exec -it sci-pod -- /bin/sh
/ # echo "File on CSI-volume" >> /data/file.txt
/ # ls /data/file.txt
/data/file.txt
/ # exit
```
- Check file on host
```
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ minikube ssh
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ sudo find / -name file.txt
/mnt/vda1/var/lib/kubelet/pods/91006633-e0a0-4601-809d-53abca9a40d8/volumes/kubernetes.io~csi/pvc-f1b92a0b-0c70-46f1-a9d2-c4ffeb69a206/mount/file.txt
/var/lib/csi-hostpath-data/04f425fe-aa42-11eb-9f84-0242ac110005/file.txt
/var/lib/kubelet/pods/91006633-e0a0-4601-809d-53abca9a40d8/volumes/kubernetes.io~csi/pvc-f1b92a0b-0c70-46f1-a9d2-c4ffeb69a206/mount/file.txt
```
- Create snapshot
```
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl apply -f kubernetes-storage/csi-pvc-snapshot.yaml 
volumesnapshot.snapshot.storage.k8s.io/snapshot-app created
```
- Check snapshot
```
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl get volumesnapshot
NAME           READYTOUSE   SOURCEPVC   SOURCESNAPSHOTCONTENT   RESTORESIZE   SNAPSHOTCLASS            SNAPSHOTCONTENT                                    CREATIONTIME   AGE
snapshot-app   true         csi-pvc                             1Gi           csi-hostpath-snapclass   snapcontent-5960342c-c735-434f-86ef-bceae3a8f98b   59s            59s
```
- Remove POD & PVC wt PV
```
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl delete -f kubernetes-storage/csi-app.yaml    
pod "csi-pod" deleted
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl delete -f kubernetes-storage/csi-pvc.yaml 
persistentvolumeclaim "csi-pvc" deleted
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl get pv
No resources found in default namespace.
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ 
```
- Restore PVC
```
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl apply -f kubernetes-storage/csi-pvc-restore.yaml 
persistentvolumeclaim/csi-pvc-restore created
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl get pvc
NAME              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
csi-pvc-restore   Bound    pvc-9df7cdca-c1a5-4571-9b8e-6841bddcf256   1Gi        RWO            csi-hostpath-sc   11s
```
- Create pod with new name PVC
```
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl apply -f kubernetes-storage/csi-pvc-restore.yaml 
persistentvolumeclaim/csi-pvc-restore created
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl get pvc

NAME              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
csi-pvc-restore   Bound    pvc-9df7cdca-c1a5-4571-9b8e-6841bddcf256   1Gi        RWO            csi-hostpath-sc   11s
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl apply -f kubernetes-storage/csi-app.yaml 
pod/sci-pod created
```
- Check file
```
➜  sergeykudelin_platform git:(kubernetes-storage) ✗ kubectl exec -it csi-pod -- /bin/sh
/ # ls /data
file.txt
/ # 
```

![The End](./kubernetes-storage/images/shakil_oneill.gif)
---
# Kubernetes - Debug

## Install 
- Install cli
```
brew install aylei/tap/kubectl-debug
```
- Установил daemonset с версией 0.1.1, так как версия 0.0.1 имела особенность, а именно не включенный SYS_PTRACE по умолчанию, но определена в более поздней версии. Более новые версии тоже несут определенные проблемы, например с pull образа агента.
```
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl apply -f kubernetes-debug/strace/kubectl-debug.yaml
daemonset.apps/debug-agent created
```
-  Проверил возможность strace
```
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl-debug frontend --agentless=false
Forwarding from 127.0.0.1:10027 -> 10027
Forwarding from [::1]:10027 -> 10027
Handling connection for 10027
                             pulling image docker.io/nicolaka/netshoot:latest... 
latest: Pulling from nicolaka/netshoot
ba3557a56b15: Pull complete 
b6950ffaa29e: Pull complete 
f4e19f280112: Pull complete 
98080a8ada1b: Pull complete 
0a2f2053602a: Pull complete 
ab1b1d3ce981: Pull complete 
8ff9b7faf304: Pull complete 
a226db7afe28: Pull complete 
1110ea2ea993: Pull complete 
1d5fe12c8162: Pull complete 
Digest: sha256:f31f63ead49ba5bf47213ee213713bd81fdb4e1e72f25d87cbaaff7ba8ab2398
Status: Downloaded newer image for nicolaka/netshoot:latest
starting debug container...
container created, open tty...
bash-5.1# strace -c -p1
strace: Process 1 attached
```
- Для следующего задания переподниму minikube c включенным calico
```
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ minikube start --network-plugin=cni --cni=calico --kubernetes-version v1.16.0
😄  minikube v1.19.0 on Darwin 11.3
❗  Both driver=hyperkit and vm-driver=hyperkit have been set.

    Since vm-driver is deprecated, minikube will default to driver=hyperkit.

    If vm-driver is set in the global config, please run "minikube config unset vm-driver" to resolve this warning.

✨  Using the hyperkit driver based on user configuration
❗  With --network-plugin=cni, you will need to provide your own CNI. See --cni flag as a user-friendly alternative
👍  Starting control plane node minikube in cluster minikube
🔥  Creating hyperkit VM (CPUs=2, Memory=4000MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.16.0 on Docker 20.10.4 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring Calico (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass

❗  /usr/local/bin/kubectl is version 1.18.6, which may have incompatibilites with Kubernetes 1.16.0.
    ▪ Want kubectl v1.16.0? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ watch kubectl get pods -l k8s-app=calico-node -n kube-system

➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl get pods -l k8s-app=calico-node -n kube-system 

NAME                READY   STATUS    RESTARTS   AGE
calico-node-mg2kw   1/1     Running   0          98s
```
- Копируем манифесты с официального репозитория kit/deploy
- Применяем манифесты
```
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl apply -f kubernetes-debug/kit/deploy/crd.yaml 
customresourcedefinition.apiextensions.k8s.io/netperfs.app.example.com created
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl apply -f kubernetes-debug/kit/deploy/rbac.yaml 
role.rbac.authorization.k8s.io/netperf-operator created
rolebinding.rbac.authorization.k8s.io/default-account-netperf-operator created
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl apply -f kubernetes-debug/kit/deploy/operator.yaml 
deployment.apps/netperf-operator created
```
- Проверяем работоспособность
```
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl apply -f kubernetes-debug/kit/deploy/cr.yaml 
netperf.app.example.com/example created
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
netperf-client-48028c2ea67f         1/1     Running   0          3s
netperf-operator-85569b59dd-226wv   1/1     Running   0          70s
netperf-server-48028c2ea67f         1/1     Running   0          10s
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl describe netperf.app.example.com/example
Name:         example
Namespace:    default
Labels:       <none>
Annotations:  API Version:  app.example.com/v1alpha1
Kind:         Netperf
Metadata:
  Creation Timestamp:  2021-05-05T18:55:21Z
  Generation:          4
  Resource Version:    1088
  Self Link:           /apis/app.example.com/v1alpha1/namespaces/default/netperfs/example
  UID:                 539154c2-a89c-4571-899c-48028c2ea67f
Spec:
  Client Node:  
  Server Node:  
Status:
  Client Pod:          netperf-client-48028c2ea67f
  Server Pod:          netperf-server-48028c2ea67f
  Speed Bits Per Sec:  11152.91
  Status:              Done
Events:                <none>
```
- Применяем политику для calico и видим что тест не проходит
```
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl apply -f kubernetes-debug/kit/netperf-calico-policy.yaml 
networkpolicy.crd.projectcalico.org/netperf-calico-policy created
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl delete -f kubernetes-debug/kit/deploy/cr.yaml && kubectl apply -f kubernetes-debug/kit/deploy/cr.yaml 
netperf.app.example.com "example" deleted
netperf.app.example.com/example created
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl describe netperf.app.example.com/example                Name:         example
Namespace:    default
Labels:       <none>
Annotations:  API Version:  app.example.com/v1alpha1
Kind:         Netperf
Metadata:
  Creation Timestamp:  2021-05-05T19:01:44Z
  Generation:          3
  Resource Version:    1378
  Self Link:           /apis/app.example.com/v1alpha1/namespaces/default/netperfs/example
  UID:                 960f614c-7ab6-40d8-be05-7908f4d4cb7e
Spec:
  Client Node:  
  Server Node:  
Status:
  Client Pod:          netperf-client-7908f4d4cb7e
  Server Pod:          netperf-server-7908f4d4cb7e
  Speed Bits Per Sec:  0
  Status:              Started test
Events:                <none>
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ 
```
- Что бы понять что происходит, нам бы как то отслеживать drop пакетов, для этого будем использовать tailer, ставим ...
```
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl apply -f kubernetes-debug/kit/kit-serviceaccount.yaml 
serviceaccount/kube-iptables-tailer created
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl apply -f kubernetes-debug/kit/kit-clusterrole.yaml   
clusterrole.rbac.authorization.k8s.io/kube-iptables-tailer created
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl apply -f kubernetes-debug/kit/kit-clusterrolebinding.yaml 
clusterrolebinding.rbac.authorization.k8s.io/kube-iptables-tailer created
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl apply -f kubernetes-debug/kit/kit-daemonset.yaml         
daemonset.apps/kube-iptables-tailer created
```
- Что бы daemonset заработал нормально пришлось немного затюнить minikube
```
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ minikube ssh
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ sudo mkdir /var/log/journal
$ sudo systemd-tmpfiles --create --prefix /var/log/journal
$ sudo systemctl restart systemd-journald
```
- Проверяем корректный старт
```
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl delete pod kube-iptables-tailer-j8rsx -n kube-system
pod "kube-iptables-tailer-j8rsx" deleted
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl get pod -n kube-system                              
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-77d6cbc65f-89wsc   1/1     Running   0          49m
calico-node-mg2kw                          1/1     Running   0          49m
coredns-5644d7b6d9-hwrm6                   1/1     Running   0          49m
etcd-minikube                              1/1     Running   0          48m
kube-apiserver-minikube                    1/1     Running   0          48m
kube-controller-manager-minikube           1/1     Running   0          48m
kube-iptables-tailer-qfbsz                 1/1     Running   0          54s
kube-proxy-6xgjw                           1/1     Running   0          49m
kube-scheduler-minikube                    1/1     Running   0          48m
storage-provisioner                        1/1     Running   1          49m
```
- Попробуем посмотреть что поменяет по отношению описание pod в котором происходит тест
```
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl delete -f kubernetes-debug/kit/deploy/cr.yaml && kubectl apply -f kubernetes-debug/kit/deploy/cr.yaml
netperf.app.example.com "example" deleted
netperf.app.example.com/example created
```
- Check event about DROP package
```
➜  sergeykudelin_platform git:(kubernetes-debug) ✗ kubectl describe pod netperf-server-cf23c6780c65
...
Events:
  Type    Reason     Age        From                    Message
  ----    ------     ----       ----                    -------
  Normal  Scheduled  <unknown>  default-scheduler       Successfully assigned default/netperf-server-cf23c6780c65 to minikube
  Normal  Pulled     31s        kubelet, minikube       Container image "tailoredcloud/netperf:v2.7" already present on machine
  Normal  Created    31s        kubelet, minikube       Created container netperf-server-cf23c6780c65
  Normal  Started    31s        kubelet, minikube       Started container netperf-server-cf23c6780c65
  Warning PacketDrop 31s        kube-iptables-tailer    Packet dropped when receiving traffic from client (10.244.120.90)
```


# Kubernetes - Production Cluster

## Manual Mode

- Create instances
```
➜  sergeykudelin_platform git:(kubernetes-production-clusters) for i in master worker0 worker1 worker2; do gcloud compute instances create $i --zone=us-west2-c --image-project=ubuntu-os-cloud --image-family=ubuntu-minimal-1804-lts --machine-type=n1-standard-2; done
Created [https://www.googleapis.com/compute/v1/projects/my-project-1512478941742/zones/us-west2-c/instances/master].
NAME    ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
master  us-west2-c  n1-standard-2               10.168.0.2   35.236.48.122  RUNNING
Created [https://www.googleapis.com/compute/v1/projects/my-project-1512478941742/zones/us-west2-c/instances/worker0].
NAME     ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP   STATUS
worker0  us-west2-c  n1-standard-2               10.168.0.3   34.94.13.206  RUNNING
Created [https://www.googleapis.com/compute/v1/projects/my-project-1512478941742/zones/us-west2-c/instances/worker1].
NAME     ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP   STATUS
worker1  us-west2-c  n1-standard-2               10.168.0.4   34.94.58.121  RUNNING
Created [https://www.googleapis.com/compute/v1/projects/my-project-1512478941742/zones/us-west2-c/instances/worker2].
NAME     ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
worker2  us-west2-c  n1-standard-2               10.168.0.5   34.94.219.110  RUNNING
```
- Configure instances

```
# swap off
root@master:/home/sergeykudelin# swapoff -a
# route
root@master:/home/sergeykudelin# cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
> net.bridge.bridge-nf-call-iptables = 1
> net.ipv4.ip_forward = 1
> net.bridge.bridge-nf-call-ip6tables = 1
> EOF
root@master:/home/sergeykudelin# sysctl --system | grep 99-kubernetes
* Applying /etc/sysctl.d/99-kubernetes-cri.conf ...
# Установка Docker
apt update && apt-get install -y apt-transport-https ca-certificates curl software-properties-common gnupg2
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - 
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt update && apt-get install -y \
          containerd.io=1.2.13-1 \
          docker-ce=5:19.03.8~3-0~ubuntu-$(lsb_release -cs) \
          docker-ce-cli=5:19.03.8~3-0~ubuntu-$(lsb_release -cs)
# Конфигурируем docker:
cat > /etc/docker/daemon.json <<EOF
{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m"
    },
    "storage-driver": "overlay2"
}
EOF
# Добавляем как сервис
mkdir -p /etc/systemd/system/docker.service.d && systemctl daemon-reload && systemctl restart docker
```

- Установка kubeadm, kubelet and kubectl

```
# kube repo
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
# Xenial because repo doesn't have bionic release
# k8s == 1.17
apt update && apt install -y kubelet=1.17.4-00 kubeadm=1.17.4-00 kubectl=1.17.4-00
```

- Конфигурируем кластер

```
root@master:/home/sergeykudelin# kubeadm init --pod-network-cidr=192.168.0.0/24
root@master:/home/sergeykudelin# mkdir -p $HOME/.kube
root@master:/home/sergeykudelin# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
root@master:/home/sergeykudelin# sudo chown $(id -u):$(id -g) $HOME/.kube/config
root@master:/home/sergeykudelin# kubectl get nodes
NAME     STATUS     ROLES    AGE   VERSION
master   NotReady   master   13m   v1.17.4

# Calico setup
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

 - Добавляем worker node-ы

```
kubeadm join 10.168.0.2:6443 --token djgvl8.fzynwwoxjxycqaby \
    --discovery-token-ca-cert-hash sha256:81ec79a819d05487891d76e29bef5b885d605c3bbdaac6a929f9359b327d9697

#root@master:/home/sergeykudelin# kubectl get nodes
master    Ready    master   19m   v1.17.4
worker0   Ready    <none>   48s   v1.17.4
worker1   Ready    <none>   48s   v1.17.4
worker2   Ready    <none>   48s   v1.17.4
```

- Создаем Deployment
```
root@master:/home/sergeykudelin# kubectl apply -f deployment.yaml
deployment.apps/nginx-deployment created
root@master:/home/sergeykudelin# kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-c8fd555cc-r7mv9   1/1     Running   0          64s
nginx-deployment-c8fd555cc-rhz87   1/1     Running   0          64s
nginx-deployment-c8fd555cc-vpdpw   1/1     Running   0          64s
nginx-deployment-c8fd555cc-zhpt6   1/1     Running   0          64s
```

- Обновляем кластер
```
root@master:/home/sergeykudelin# apt update && apt-get install -y kubeadm=1.18.0-00 kubelet=1.18.0-00 kubectl=1.18.0-00
Hit:1 https://download.docker.com/linux/ubuntu bionic InRelease
Hit:2 http://us-west2.gce.archive.ubuntu.com/ubuntu bionic InRelease                                                                                      
Get:4 http://us-west2.gce.archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]                                                                    
Get:5 http://us-west2.gce.archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]                          
Hit:3 https://packages.cloud.google.com/apt kubernetes-xenial InRelease                                   
Get:6 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]                               
Fetched 252 kB in 1s (314 kB/s)                                
Reading package lists... Done
Building dependency tree       
Reading state information... Done
6 packages can be upgraded. Run 'apt list --upgradable' to see them.
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages will be upgraded:
  kubeadm kubectl kubelet
3 upgraded, 0 newly installed, 0 to remove and 3 not upgraded.
Need to get 36.4 MB of archives.
After this operation, 2612 kB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.18.0-00 [19.4 MB]
Get:2 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubectl amd64 1.18.0-00 [8822 kB]
Get:3 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.18.0-00 [8163 kB]
Fetched 36.4 MB in 2s (15.3 MB/s) 
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 52866 files and directories currently installed.)
Preparing to unpack .../kubelet_1.18.0-00_amd64.deb ...
Unpacking kubelet (1.18.0-00) over (1.17.4-00) ...
Preparing to unpack .../kubectl_1.18.0-00_amd64.deb ...
Unpacking kubectl (1.18.0-00) over (1.17.4-00) ...
Preparing to unpack .../kubeadm_1.18.0-00_amd64.deb ...
Unpacking kubeadm (1.18.0-00) over (1.17.4-00) ...
Setting up kubelet (1.18.0-00) ...
Setting up kubectl (1.18.0-00) ...
Setting up kubeadm (1.18.0-00) ...
root@master:/home/sergeykudelin# kubectl get nodes
NAME      STATUS   ROLES    AGE   VERSION
master    Ready    master   25m   v1.18.0
worker0   Ready    <none>    6m   v1.17.4
worker1   Ready    <none>    6m   v1.17.4
worker2   Ready    <none>    6m   v1.17.4
```
- Обновляем компоненты
```
root@master:/home/sergeykudelin# kubeadm upgrade plan
...
root@master:/home/sergeykudelin# kubeadm upgrade apply v1.18.0
...
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.18.0". Enjoy!
```

- Обновляем node-ы
```
root@master:/home/sergeykudelin# kubectl drain worker0 --ignore-daemonsets
root@master:/home/sergeykudelin# kubectl get nodes
 NAME      STATUS                     ROLES    AGE   VERSION
 master    Ready                      master   26m   v1.18.0
 worker0   Ready,SchedulingDisabled   <none>    7m   v1.17.4
 worker1   Ready                      <none>    7m   v1.17.4
 worker2   Ready                      <none>    7m   v1.17.4

apt install -y kubelet=1.18.0-00 kubeadm=1.18.0-00
systemctl restart kubelet
kubectl uncordon worker0


root@master:/home/sergeykudelin# kubectl get nodes
NAME      STATUS   ROLES    AGE   VERSION
master    Ready    master   28m   v1.18.0
worker0   Ready    <none>    9m   v1.18.0
worker1   Ready    <none>    9m   v1.17.4
worker2   Ready    <none>    9m   v1.17.4
```
## Kubespray Way

- Создаем instance-ы

```
➜  sergeykudelin_platform git:(kubernetes-production-clusters) ✗ for i in master0 master1 master2 worker0 worker1; do gcloud compute instances create $i --zone=us-west2-c --image-project=ubuntu-os-cloud --image-family=ubuntu-minimal-1804-lts --machine-type=n1-standard-2; done 
Created [https://www.googleapis.com/compute/v1/projects/my-project-1512478941742/zones/us-west2-c/instances/master0].
NAME     ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP   STATUS
master0  us-west2-c  n1-standard-2               10.168.0.6   34.94.58.121  RUNNING
Created [https://www.googleapis.com/compute/v1/projects/my-project-1512478941742/zones/us-west2-c/instances/master1].
NAME     ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP   STATUS
master1  us-west2-c  n1-standard-2               10.168.0.7   34.94.13.206  RUNNING
Created [https://www.googleapis.com/compute/v1/projects/my-project-1512478941742/zones/us-west2-c/instances/master2].
NAME     ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
master2  us-west2-c  n1-standard-2               10.168.0.8   35.236.48.122  RUNNING
Created [https://www.googleapis.com/compute/v1/projects/my-project-1512478941742/zones/us-west2-c/instances/worker0].
NAME     ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
worker0  us-west2-c  n1-standard-2               10.168.0.9   34.94.219.110  RUNNING
Created [https://www.googleapis.com/compute/v1/projects/my-project-1512478941742/zones/us-west2-c/instances/worker1].
NAME     ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
worker1  us-west2-c  n1-standard-2               10.168.0.10  35.236.108.245  RUNNING
```

- Добавим свой ssh

```
➜  sergeykudelin_platform git:(kubernetes-production-clusters) ✗ gcloud beta compute ssh --zone "us-west2-c" "master0"       
Warning: Permanently added 'compute.6207459233293891678' (ECDSA) to the list of known hosts.
***
sergeykudelin@master0:~$ exit
logout
Connection to 34.94.58.121 closed.
```

- Конфигурация kubespray
```
➜  sergeykudelin_platform git:(kubernetes-production-clusters) ✗ git clone https://github.com/kubernetes-sigs/kubespray.git

Cloning into 'kubespray'...
remote: Enumerating objects: 53743, done.
remote: Counting objects: 100% (288/288), done.
remote: Compressing objects: 100% (203/203), done.
remote: Total 53743 (delta 109), reused 202 (delta 65), pack-reused 53455
Receiving objects: 100% (53743/53743), 15.63 MiB | 757.00 KiB/s, done.
Resolving deltas: 100% (30250/30250), done.

➜  sergeykudelin_platform git:(kubernetes-production-clusters) ✗ ansible-playbook --version
ansible-playbook 2.9.5
  config file = None
  configured module search path = ['/Users/sergeykudelin/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages/ansible
  executable location = /Library/Frameworks/Python.framework/Versions/3.7/bin/ansible-playbook
  python version = 3.7.6 (v3.7.6:43364a7ae0, Dec 18 2019, 14:18:50) [Clang 6.0 (clang-600.0.57)]

➜  sergeykudelin_platform git:(kubernetes-production-clusters) ✗ pipenv --python 3.7
Creating a virtualenv for this project...
Pipfile: /Users/sergeykudelin/GIT/otus/sergeykudelin_platform/Pipfile
Using /Library/Frameworks/Python.framework/Versions/3.7/bin/python3.7m (3.7.6) to create virtualenv...
⠙ Creating virtual environment...created virtual environment CPython3.7.6.final.0-64 in 761ms
  creator CPython3Posix(dest=/Users/sergeykudelin/.local/share/virtualenvs/sergeykudelin_platform-aLsAXhC8, clear=False, global=False)
  seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=/Users/sergeykudelin/Library/Application Support/virtualenv)
    added seed packages: pip==21.0.1, setuptools==53.0.0, wheel==0.36.2
  activators BashActivator,CShellActivator,FishActivator,PowerShellActivator,PythonActivator,XonshActivator

✔ Successfully created virtual environment! 
Virtualenv location: /Users/sergeykudelin/.local/share/virtualenvs/sergeykudelin_platform-aLsAXhC8
Creating a Pipfile for this project...

➜  sergeykudelin_platform git:(kubernetes-production-clusters) ✗ pipenv run pip install -r kubespray/requirements.txt 
Collecting ansible==2.9.20
  Downloading ansible-2.9.20.tar.gz (14.3 MB)
     |████████████████████████████████| 14.3 MB 2.3 MB/s 
Collecting cryptography==2.8
  Downloading cryptography-2.8-cp34-abi3-macosx_10_6_intel.whl (1.6 MB)
     |████████████████████████████████| 1.6 MB 3.1 MB/s 
Collecting jinja2==2.11.3
  Downloading Jinja2-2.11.3-py2.py3-none-any.whl (125 kB)
     |████████████████████████████████| 125 kB 1.3 MB/s 
Collecting netaddr==0.7.19
  Downloading netaddr-0.7.19-py2.py3-none-any.whl (1.6 MB)
     |████████████████████████████████| 1.6 MB 1.9 MB/s 
Collecting pbr==5.4.4
  Downloading pbr-5.4.4-py2.py3-none-any.whl (110 kB)
     |████████████████████████████████| 110 kB 2.6 MB/s 
Collecting jmespath==0.9.5
  Downloading jmespath-0.9.5-py2.py3-none-any.whl (24 kB)
Collecting ruamel.yaml==0.16.10
  Downloading ruamel.yaml-0.16.10-py2.py3-none-any.whl (111 kB)
     |████████████████████████████████| 111 kB 169 kB/s 
Collecting MarkupSafe==1.1.1
  Downloading MarkupSafe-1.1.1-cp37-cp37m-macosx_10_9_x86_64.whl (16 kB)
Collecting PyYAML
  Downloading PyYAML-5.4.1-cp37-cp37m-macosx_10_9_x86_64.whl (249 kB)
     |████████████████████████████████| 249 kB 2.2 MB/s 
Collecting six>=1.4.1
  Downloading six-1.16.0-py2.py3-none-any.whl (11 kB)
Collecting cffi!=1.11.3,>=1.8
  Downloading cffi-1.14.5-cp37-cp37m-macosx_10_9_x86_64.whl (176 kB)
     |████████████████████████████████| 176 kB 2.7 MB/s 
Collecting ruamel.yaml.clib>=0.1.2
  Downloading ruamel.yaml.clib-0.2.2-cp37-cp37m-macosx_10_9_x86_64.whl (147 kB)
     |████████████████████████████████| 147 kB 2.8 MB/s 
Collecting pycparser
  Downloading pycparser-2.20-py2.py3-none-any.whl (112 kB)
     |████████████████████████████████| 112 kB 3.4 MB/s 
Building wheels for collected packages: ansible
  Building wheel for ansible (setup.py) ... done
  Created wheel for ansible: filename=ansible-2.9.20-py3-none-any.whl size=16203439 sha256=eef5a30a32c623db592ed7b2191efdd537d523e52c453179ffbeb9fcc4030113
  Stored in directory: /private/var/folders/kp/_1hzq2bx4cb2w_g4h52s9xj00000gp/T/pip-ephem-wheel-cache-qq2xp30u/wheels/55/6c/ef/50eac044302d863dba57c8199e98bf5a7a241678b057649831
Successfully built ansible
Installing collected packages: pycparser, six, MarkupSafe, cffi, ruamel.yaml.clib, PyYAML, jinja2, cryptography, ruamel.yaml, pbr, netaddr, jmespath, ansible
Successfully installed MarkupSafe-1.1.1 PyYAML-5.4.1 ansible-2.9.20 cffi-1.14.5 cryptography-2.8 jinja2-2.11.3 jmespath-0.9.5 netaddr-0.7.19 pbr-5.4.4 pycparser-2.20 ruamel.yaml-0.16.10 ruamel.yaml.clib-0.2.2 six-1.16.0

