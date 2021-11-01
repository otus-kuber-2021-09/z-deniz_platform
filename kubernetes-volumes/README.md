# Выполнено ДЗ №3 Kubernetes Volumes

## Основное ДЗ

1. Подготовлен Stateful Set для minio
2. Подготовлен Headless серивис для minio

## Задание со *: Payment Service

1. Помещены данные в secrets и настроена конфигурация на их использование.

## Как запустить проект:
- ``` minio-secret.yaml ``` - для загрузки секретов
- ``` kubectl apply -f minio-statefulset.yaml ``` - запуск основоного ДЗ
- ``` minio-headless-service.yaml``` - для запуска сервиса

## PR checklist:
 - [ ] Выставлен label с темой домашнего задания
 - [ ] Основное ДЗ
 - [ ] Задание со *
 
