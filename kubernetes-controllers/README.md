# Выполнено ДЗ №3 Kubernetes Controllers

## Основное ДЗ

1. Подготовлен Replica Set для frontend и paymentservice
2. Подготовлен Deployment для frontend и paymentservice

- Обновление ReplicaSet не повлекло обновление запущенных pod потому, что обновление подов выполняется в deployment
- Для запуска node-exporter на мастер ноде необходимо добавить 
## Задание со *: Payment Service

1. Подготовлены манифеста для Blue-Green и Reverse update

## Как запустить проект:
- ``` kubectl apply -f web-pod.yaml ``` - запуск основоного ДЗ

- ``` kubectl apply -f frontend-deployment.yaml``` - запуск основоного ДЗ
- ``` kubectl apply -f frontend-replicaset.yaml``` - запуск основоного ДЗ
- ``` kubectl apply -f paymentservice-deployment.yaml``` - запуск основоного ДЗ
- ``` kubectl apply -f paymentservice-replicaset.yaml``` - запуск основоного ДЗ

- ``` kubectl apply -f paymentservice-deployment-bg.yaml ``` - для запуска задания со *
- ``` kubectl apply -f paymentservice-deployment-reverse.yaml ``` - для запуска задания со *

- ``` kubectl apply -f node-exporter-daemonset.yaml ``` - для запуска задания с **

## Как проверить работоспособность:
 - - ``` kubectl port-forward <Имя пода node-exporter>9100:9100 ``` - для доступа по url http://localhost:9100 до контента pod дополнительного ДЗ с **

## PR checklist:
 - [ ] Выставлен label с темой домашнего задания
 - [ ] Основное ДЗ
 - [ ] Задание со *
 - [ ] Послед удаления подов в namespace kube-system они пересоздаются kublet-ом, агентом, работающим на каждом узле в кластере
