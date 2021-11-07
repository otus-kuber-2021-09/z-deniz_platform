# Выполнено ДЗ №5 Kubernetes Security

## Основное ДЗ

1. Задание №1

- Создан Service Account bob, ему выдана роль admin в рамках всего кластера
- Создан Service Account dave без доступа к кластеру

2. Задание №2

- Создан Namespace prometheus
- Создан Service Account carol в этом Namespace
- Выданы всем Service Account в Namespace prometheus права делать get, list, watch в отношении Pods всего кластера

3. Задание №3

- Создан Namespace dev
- Создан Service Account jane в Namespace dev
- Выдана роль роль admin Service Account jane в рамках Namespace dev
- Создан Service Account ken в Namespace dev
- Выдана роль роль view Service Account ken в рамках Namespace dev

## Как запустить проект:

- ``` kubectl apply -f task01/. ``` - запустить Задание №1
- ``` kubectl apply -f task02/. ``` - запустить Задание №2
- ``` kubectl apply -f task03/. ``` - запустить Задание №3

## PR checklist:
 - [ ] Выставлен label с темой домашнего задания
 - [ ] Основное ДЗ
