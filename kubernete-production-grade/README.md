# Подходы к развертыванию и обновлению production-grade кластера

## Основное ДЗ: Ручная настройка кластера

*Развернуть кластер с 1 мастер нодой и 3 worker нодами.*
 - Подготовлены 4 ВМ в облаке yandex
 - На всех ВМ настроена маршрутизация, установлены необходимые сервисы и пакеты (docker, kubadm, kubectl, etc)
 - Настроена мастер нода и worker ноды с версиеу кубера 1.17.4
 - Обновлена мастер нода до весрии 1.18.0
 - Последовательно выведены из кластера worker ноды и обновлены до версии 1.18.0

*Вывод kubctl get nodes*
```sh
kubectl get nodes -o wide
NAME         STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
k8s-master   Ready    master   30m   v1.18.0   10.128.0.20   <none>        Ubuntu 18.04.6 LTS   4.15.0-112-generic   docker://19.3.8
k8s-node-0   Ready    <none>   24m   v1.18.0   10.128.0.5    <none>        Ubuntu 18.04.6 LTS   4.15.0-112-generic   docker://19.3.8
k8s-node-1   Ready    <none>   24m   v1.18.0   10.128.0.17   <none>        Ubuntu 18.04.6 LTS   4.15.0-112-generic   docker://19.3.8
k8s-node-2   Ready    <none>   24m   v1.18.0   10.128.0.24   <none>        Ubuntu 18.04.6 LTS   4.15.0-112-generic   docker://19.3.8
```
## ДЗ со звездочкой

Развернут кластер с 3 мастер нодами и 4 worker нодами. Для установки кластера использовался [kubespray от southbridge](https://github.com/southbridgeio/kubespray)
Установлена версия k8s v1.21.1

```sh
➜  ~ k get nodes -o wide
NAME        STATUS   ROLES                  AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
i0-kub-ft   Ready    ingress                83d   v1.21.1   192.168.51.68   <none>        CentOS Linux 7 (Core)   3.10.0-1160.45.1.el7.x86_64   docker://20.10.9
m0-kub-ft   Ready    control-plane,master   83d   v1.21.1   192.168.51.53   <none>        CentOS Linux 7 (Core)   3.10.0-1160.45.1.el7.x86_64   docker://20.10.9
m1-kub-ft   Ready    control-plane,master   83d   v1.21.1   192.168.51.54   <none>        CentOS Linux 7 (Core)   3.10.0-1160.45.1.el7.x86_64   docker://20.10.9
m2-kub-ft   Ready    control-plane,master   83d   v1.21.1   192.168.51.56   <none>        CentOS Linux 7 (Core)   3.10.0-1160.45.1.el7.x86_64   docker://20.10.9
n0-kub-ft   Ready    node                   83d   v1.21.1   192.168.51.57   <none>        CentOS Linux 7 (Core)   3.10.0-1160.45.1.el7.x86_64   docker://20.10.9
n1-kub-ft   Ready    node                   83d   v1.21.1   192.168.51.58   <none>        CentOS Linux 7 (Core)   3.10.0-1160.45.1.el7.x86_64   docker://20.10.9
n2-kub-ft   Ready    node                   83d   v1.21.1   192.168.51.59   <none>        CentOS Linux 7 (Core)   3.10.0-1160.45.1.el7.x86_64   docker://20.10.9
n3-kub-ft   Ready    node                   83d   v1.21.1   192.168.51.60   <none>        CentOS Linux 7 (Core)   3.10.0-1160.45.1.el7.x86_64   docker://20.10.9
```

## PR checklist:
 - [ ] Выполнено основное ДЗ
 - [ ] Выполнено ДЗ со *
