# Выполнено ДЗ kubernetes-templating

## Основное ДЗ

1. Настроен cert manager на выпуск сертификатов
2. Подготовлен chartmuseum и выпущен сертификат. Доступен по url https://chartmuseum.otus.fortislink.ru
3. Подготовлен harbor и выпущен сертификат. Доступен по url https://harbor.otus.fortislink.ru
4. Frontend выделен в отдельный helm chart, прикручен сертификат, доступен по URL https://shop.otus.fortislink.ru
5. payment service и shipping service установлены через kubecfg
6. kustomize настроен и выполняется kubectl apply -k kubernetes-templating/kustomize/overrides/<Название окружения>/


## PR checklist:
 - [ ] Выставлен label с темой домашнего задания
 - [ ] Основное ДЗ
