# Diplom K8s

Kubernetes-манифесты и конфиги мониторинга для дипломного проекта.

## Что внутри

- **deployment.yaml** — Deployment для приложения nginx. 2 реплики, requests 100m/128Mi.
- **service.yaml** — Service типа LoadBalancer. Открывает приложение на 80 порту снаружи.
- **atlantis/values.yaml** — конфиг для Helm-чарта Atlantis.

## Как применить

Приложение (2 реплики nginx):

```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Проверить статус:

```
kubectl get pods
kubectl get svc nginx-service
```

## Мониторинг

Мониторинг ставится через Helm-чарт `kube-prometheus-stack` в namespace `monitoring`.

Создать namespace:

```
kubectl create namespace monitoring
```

Установить чарт:

```
helm install monitoring prometheus-community/kube-prometheus-stack --namespace monitoring --timeout 20m
```

Выставить Grafana наружу (чтобы открывалась из браузера и смотрим, какой IP ей присвоился):

```
kubectl patch svc monitoring-grafana -n monitoring -p '{"spec":{"type":"LoadBalancer"}}'
kubectl get svc monitoring-grafana -n monitoring
```

**Что ставится:**
- Prometheus — сбор метрик.
- Grafana — визуализация.
- Alertmanager — оповещения.
- Node Exporter — метрики нод.
- Kube State Metrics — метрики Kubernetes-объектов.

**Дашборды Grafana** — в папке Kubernetes, например `Kubernetes / Compute Resources / Cluster`.


## Atlantis

Atlantis ставится через Helm.

Добавить репозиторий:

```
helm repo add runatlantis https://runatlantis.github.io/helm-charts
```

Установить:

```
helm install atlantis runatlantis/atlantis --namespace atlantis --create-namespace -f ~/diplom-secrets/atlantis-values.yaml --timeout 10m
```

Файл `atlantis-values.yaml` лежит **вне репозитория** — там токен GitHub и webhook secret. В репозитории лежит только шаблон `atlantis/values.yaml`

После установки — настроить webhook в GitHub (Settings → Webhooks), URL вида `http://<IP_ATLANTIS>/events`.

**про Atlantis:** описал в README репозитория `diplom-terraform`.


## Проблемы, с которыми столкнулся

**Ноды без публичного IP.** Изначально worker-ноды были без публичного IP — не могли скачивать образы ни из одного реестра. Добавил `nat = true` в `node_group.tf` в репозитории `diplom-terraform`.

**Заблокированные реестры.** `registry.k8s.io` и `ghcr.io` недоступны из России. Подменял образы через `--set` в helm.

**«Кривые» IP.** Несколько раз новые IP (мастер K8s, LoadBalancer приложения) не отвечали на TCP. Помогало пересоздание ресурса — новый IP работал.

**Одна нода не могла скачать образы с Docker Hub.** Пересоздание пода на другой ноде решило проблему.


## Ссылки

- [diplom-terraform](https://github.com/Mazaich/diplom-terraform) — инфраструктура
- [diplom-app](https://github.com/Mazaich/diplom-app) — приложение
- [diplom-notes](https://github.com/Mazaich/diplom-notes) — заметки и скриншоты
