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
