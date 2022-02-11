Kubernetes-Operators
 - Создаем CustomResource, CustomResourceDefinition
 - Деплой оператора. Применяем манифесты:
    - role-binding.yml.yml
    - cr.yml
    - crd.yml
    - deploy-operator.yml
    - role.yml
    - service-account.yml
 - Заполним базу созданного mysql-instance
 - Удаляем mysql-instance
 - Создадим заново mysql-instance
 - Проверяем, что данные восстановились.

 ```sh
 kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database
 +----+-------------+
 | id | name        |
 +----+-------------+
 |  1 | some data   |
 |  2 | some data-2 |
 +----+-------------+

kubectl get jobs
 NAME                         COMPLETIONS   DURATION   AGE
 backup-mysql-instance-job    1/1           2s         2m22s
 restore-mysql-instance-job   1/1           52s        80s

 ```
