### Kubernetes logging

Запущен кластер в яндексе, где 3 ноды с tain `node-role=infra:NoSchedule` и 2 ноды без taint
На пул с 3 нодами повесил метки `cloud.google.com/gke-nodepool=infra-pool`, на пул с 2 нодами `cloud.google.com/gke-nodepool=deault-pool` для максимального приближения к требованиям ДЗ


1\. Установим hipstershop

```bash
$ kubectl create ns microservices-demo
$ kubectl apply -f https://raw.githubusercontent.com/express42/otus-platformsnippets/master/Module-02/Logging/microservices-demo-without-resources.yaml -n microservices-demo
```


2\. Установим ElasticSearch с кастомными настройками в elasticsearch.values.yaml

```bash
$ kubectl create ns observability
$ helm upgrade --install elasticsearch elastic/elasticsearch \
    --namespace observability \
    -f elasticsearch.values.yaml
```

3\. Устанавливаем ingress

```bash
$ kubectl create ns nginx-ingress
$ helm upgrade --install nginx-ingress stable/nginx-ingress --namespace=nginx-ingress -f nginx-ingress.values.yaml
```

4\. Установим kibana с кастомными настройками в kibana.values.yaml

```bash
$ helm upgrade --install kibana elastic/kibana --namespace observability -f kibana.values.yaml
```

5\. Установим fluent-bit с кастомными настройками в fluent-bit.values.yaml

```bash
$ helm upgrade --install fluent-bit stable/fluent-bit --namespace observability -f fluent-bit.values.yaml
```

5\. Установим prometheus-operator

```bash
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
$ helm install prometheus --namespace observability prometheus-community/kube-prometheus-stack
```

6\. Устанавливаем prometheus exporter для мониторинга ElasticSearch

```bash
$ helm upgrade --install elasticsearch-exporter stable/elasticsearch-exporter --set es.uri=http://elasticsearch-master:9200 --set serviceMonitor.enabled=true --namespace=observability
```

7\. Создаем дашборд:

 - Создаем index pattern
 - Настраиваем nginx на выгрузку логов в json
 - создаем визуализацию

8\. Разворачиваем Loki

```bash
$ helm repo add loki https://grafana.github.io/loki/charts
$ helm upgrade --install loki loki/loki-stack --namespace=observability  -f loki.yaml
```

10\. Создаем дашборду для Nginx в графане
