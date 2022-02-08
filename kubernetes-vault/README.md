
1. Установка Vault. Проверим установку

```sh
✗ helm status vault
NAME: vault
LAST DEPLOYED: Sun Jan 23 21:31:25 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing HashiCorp Vault!
```

2. Проведем инициализацию, сгенерируем токены:

```sh
kubectl exec -it vault-0 -- vault operator init --key-shares=1 --key-threshold=1

Unseal Key 1: KOL465UQ8ohd40hADI5fcjfcb2daUS4R2k5Y+y47TX4=
Initial Root Token: s.DO18VKAqs9Z0XYN0s3RYwFra

Vault initialized with 1 key shares and a key threshold of 1. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 1 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated master key. Without at least 1 keys to
reconstruct the master key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```

3. Распечатали все ноды Vault

```sh
✗ kubectl exec -it vault-0 -- vault operator unseal 'KOL465UQ8ohd40hADI5fcjfcb2daUS4R2k5Y+y47TX4='
✗ kubectl exec -it vault-0 -- vault status
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.9.2
Storage Type    consul
Cluster Name    vault-cluster-fd615f24
Cluster ID      fc1934f3-44d4-95b5-482d-d848ea5d3c60
HA Enabled      true
HA Cluster      https://vault-0.vault-internal:8201
HA Mode         active
Active Since    2022-01-23T19:07:29.516617978Z
```

4. Залогинимся

```sh
✗ kubectl exec -it vault-0 --  vault login
Token (will be hidden):
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                s.DO18VKAqs9Z0XYN0s3RYwFra

token_accessor       gQthg2LLcudzmrgmgcOulQ7x
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```

5. Повторно запросим список авторизаций

```sh
✗ kubectl exec -it vault-0 --  vault auth list
Path      Type     Accessor               Description
----      ----     --------               -----------
token/    token    auth_token_303572c4    token based credentials
```

6. Создание секретов

```sh
kubectl exec -it vault-0 -- vault secrets enable --path=otus kv
Success! Enabled the kv secrets engine at: otus/

kubectl exec -it vault-0 -- vault secrets list --detailed
Path          Plugin       Accessor              Default TTL    Max TTL    Force No Cache    Replication    Seal Wrap    External Entropy Access    Options    Description                                                UUID
----          ------       --------              -----------    -------    --------------    -----------    ---------    -----------------------    -------    -----------                                                ----
cubbyhole/    cubbyhole    cubbyhole_a7c3b708    n/a            n/a        false             local          false        false                      map[]      per-token private secret storage                           f1215b3a-ac7a-e396-b91f-57da30b4c0b3
identity/     identity     identity_60572807     system         system     false             replicated     false        false                      map[]      identity store                                             02b14e36-5bb7-c467-3083-8a1cc0179615
otus/         kv           kv_4e5d73fc           system         system     false             replicated     false        false                      map[]      n/a                                                        fa90d7ca-5f53-7ca8-84c2-c52fe1319ba6
sys/          system       system_6bd4611d       n/a            n/a        false             replicated     false        false                      map[]      system endpoints used for control, policy and debugging    b26e7d8e-1e89-2fc9-e989-e4bd789adefc


kubectl exec -it vault-0 -- vault kv put otus/otus-ro/config username='otus' password='P@ssw0rd'
Success! Data written to: otus/otus-ro/config

kubectl exec -it vault-0 -- vault kv put otus/otus-rw/config username='otus' password='P@ssw0rd'
Success! Data written to: otus/otus-rw/config

kubectl exec -it vault-0 -- vault read otus/otus-ro/config
Key                 Value
---                 -----
refresh_interval    768h
password            P@ssw0rd
username            otus

kubectl exec -it vault-0 -- vault kv get otus/otus-rw/config
====== Data ======
Key         Value
---         -----
password    P@ssw0rd
username    otus
```
7. Включим авторизацию через k8s

```sh
kubectl exec -it vault-0 --  vault auth list
Path           Type          Accessor                    Description
----           ----          --------                    -----------
kubernetes/    kubernetes    auth_kubernetes_25775c70    n/a
token/         token         auth_token_303572c4         token based credentials
```

8. Ошибки при записи вызваны тем, что политика настроена на "read", "create", "list", для того, чтоб обновить данные, нужно добавить  "update"
9. Для того, что бы отработал агент необходимо создавать секреты с движком KV2
10. Выдача сертификата

```sh
➜  ~ kubectl exec -it vault-0 -- vault write pki_int/issue/example-dot-ru common_name="gitlab.example.ru" ttl="24h"
Key                 Value
---                 -----
ca_chain            [-----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUa7cGXBZZK+csL8/tkurE+ivQwCQwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0yMjAyMDgyMjIzNThaFw0yNzAy
MDcyMjI0MjhaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMJTzkCJn1Si
eNK9C2khduwPEp6gzYRHGSEsAaiMCdDaEISUcY+kc61XagNy8YbYWwtEvAC/XiLX
FIxjEH1caR+aJnEzjAc53moR+BcOIlJNL8DGhajlD8RNJiUSuAnJi/nTRfaLWBY0
N5FUne1dJG2Atmso628TCLB4wnXwJf0RJOUAPr59FGJEdq0693MR6mUxrfk7WsLz
YONL5cSOcWyalGqfrpFZvqm+gFoDWWItqjUo5DxxhIl46rsERDZLAxAVRD4ZtH8W
AOzDE4xWsrmEhQiCoZVJakU9HJaXfzubxv7asfwIxt1ay6tRvoLzaEJvqZ7+Zd/2
xAnYEMPRDmkCAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUaf4RXkOQ0JDuP2vbQFXzc+Gnb24wHwYDVR0jBBgwFoAU
bw2IqYN6MGz+tVY7W3euCqDzpHwwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
SvvQ6anAQIJK6VORo1xJltlzCPFQQsz2ICcXJEW0qYgjkBPMe/mVADdhdDCe8czw
eQtV/WnUEQ64xbEtGobpSia/OjmTzGZQiHRNhfRFt6GXC6bv99LHk170t1JtEOk4
rRNVqVym1UIRiwmQEiMKELc/iXXPN9ZqJJHPc+2VOTuRI1ga2bOj86DjGI9RzTJI
OTyy7e1T/S2msWardUQKl71ewCmQvPrU5wYHNV2lkmt7ig8DIy4xMABt1Qg+axDv
H3O2sQyzq3ONLzpVZ/IDMlI0zreURfkezgusLsqkiOLImq1n9twCU0hHzAZ5ycvS
dj0xxWdJVZcdAR5/v0OsUA==
-----END CERTIFICATE-----]
certificate         -----BEGIN CERTIFICATE-----
MIIDZzCCAk+gAwIBAgIUV4p3oEWNV+PaLrV3nECD66OLA7gwDQYJKoZIhvcNAQEL
BQAwLDEqMCgGA1UEAxMhZXhhbXBsZS5ydSBJbnRlcm1lZGlhdGUgQXV0aG9yaXR5
MB4XDTIyMDIwODIyMzEyOVoXDTIyMDIwOTIyMzE1OVowHDEaMBgGA1UEAxMRZ2l0
bGFiLmV4YW1wbGUucnUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDr
7CgCe9jofAp+KIt05FtQCFaQ+6HdBySjwnShGru/CIh145l07fpTlP6rjoSM2BgG
xrxApx2bD0L5MjAwQPH48/4jqghQWR4J7SeyXRoqPKZCUKNdtZKxveDMG2zGgcVx
ZnyamNWKwoH2TiXHymckJYYqsnD+z/dlFYEPO7I2L7voLod7do8p+Rpmv+Hygq3k
0xS5AGBUOS6u9WrG0+0ly3yzVYgeyjE3qC6N9KKYh8arHvT7czA2aPMFSnh3fJZl
e7Z6bFeMtUBSjQo/T5ptsHWEwcB2az0/zg2u+nCjaWtCrwkBnrk+LwGhItQiCfbK
HKbeuEZ9DyoZ5tZdpF5jAgMBAAGjgZAwgY0wDgYDVR0PAQH/BAQDAgOoMB0GA1Ud
JQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjAdBgNVHQ4EFgQUoNt0aMk+fFStA7Jb
hrVxmNC9vbwwHwYDVR0jBBgwFoAUaf4RXkOQ0JDuP2vbQFXzc+Gnb24wHAYDVR0R
BBUwE4IRZ2l0bGFiLmV4YW1wbGUucnUwDQYJKoZIhvcNAQELBQADggEBAF6oP61+
tPZKQDKSdykDs1U2irs424MGByqwZXx2Hqjv/i396eyOWHUALkYe+hVYkvY0sfZ7
q+43g7YiGH5lJRNUloBOZ7+MR66YNTXs68LdtDAOu3GfzgKLdrKLTlnoThgT5wm9
z3lxmGk9DyMAvfwQV/LFPXh3C5W9qwtu80yHAxzPrtaYnJzTbCyRk9I3N9U2Ly//
DgYbmrXqlq4Aj4okL4MIXlmLVm4PbaKiWFqLuA0ZPVWqJWmlEyvgtG9tTmmkIoXq
Fjgb0I1FDUTDuhkKAPFaVwQzwLjpg+hKz5mRz0qknNB1CieW5iOKNdn7AO4dVzyA
DSG01kW4N5SGD9M=
-----END CERTIFICATE-----
expiration          1644445919
issuing_ca          -----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUa7cGXBZZK+csL8/tkurE+ivQwCQwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0yMjAyMDgyMjIzNThaFw0yNzAy
MDcyMjI0MjhaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMJTzkCJn1Si
eNK9C2khduwPEp6gzYRHGSEsAaiMCdDaEISUcY+kc61XagNy8YbYWwtEvAC/XiLX
FIxjEH1caR+aJnEzjAc53moR+BcOIlJNL8DGhajlD8RNJiUSuAnJi/nTRfaLWBY0
N5FUne1dJG2Atmso628TCLB4wnXwJf0RJOUAPr59FGJEdq0693MR6mUxrfk7WsLz
YONL5cSOcWyalGqfrpFZvqm+gFoDWWItqjUo5DxxhIl46rsERDZLAxAVRD4ZtH8W
AOzDE4xWsrmEhQiCoZVJakU9HJaXfzubxv7asfwIxt1ay6tRvoLzaEJvqZ7+Zd/2
xAnYEMPRDmkCAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUaf4RXkOQ0JDuP2vbQFXzc+Gnb24wHwYDVR0jBBgwFoAU
bw2IqYN6MGz+tVY7W3euCqDzpHwwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
SvvQ6anAQIJK6VORo1xJltlzCPFQQsz2ICcXJEW0qYgjkBPMe/mVADdhdDCe8czw
eQtV/WnUEQ64xbEtGobpSia/OjmTzGZQiHRNhfRFt6GXC6bv99LHk170t1JtEOk4
rRNVqVym1UIRiwmQEiMKELc/iXXPN9ZqJJHPc+2VOTuRI1ga2bOj86DjGI9RzTJI
OTyy7e1T/S2msWardUQKl71ewCmQvPrU5wYHNV2lkmt7ig8DIy4xMABt1Qg+axDv
H3O2sQyzq3ONLzpVZ/IDMlI0zreURfkezgusLsqkiOLImq1n9twCU0hHzAZ5ycvS
dj0xxWdJVZcdAR5/v0OsUA==
-----END CERTIFICATE-----
private_key         -----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEA6+woAnvY6HwKfiiLdORbUAhWkPuh3Qcko8J0oRq7vwiIdeOZ
dO36U5T+q46EjNgYBsa8QKcdmw9C+TIwMEDx+PP+I6oIUFkeCe0nsl0aKjymQlCj
XbWSsb3gzBtsxoHFcWZ8mpjVisKB9k4lx8pnJCWGKrJw/s/3ZRWBDzuyNi+76C6H
e3aPKfkaZr/h8oKt5NMUuQBgVDkurvVqxtPtJct8s1WIHsoxN6gujfSimIfGqx70
+3MwNmjzBUp4d3yWZXu2emxXjLVAUo0KP0+abbB1hMHAdms9P84Nrvpwo2lrQq8J
AZ65Pi8BoSLUIgn2yhym3rhGfQ8qGebWXaReYwIDAQABAoIBAHxSArdkWeYQz1qx
tONRHokrC6r03tPhWr5szxbCRqMhNP+igxAqA5qdziHLRTAPA4I2oacUKTa3sRwu
BVS4NIpy0L4scJseiwTCEwQbqZkOQrJ5Wc0czIObQmVsIkLsyYW7cvfoh8bKPr/z
aFdC4l0a5PuE8qRkJMMAtPS5CW0hcUliOjLw15ElQcmvBLkFxMy2QvOpfPoxMDxc
61uBJEVS/z39AskiP9/CvTQb2tecVE7Jod6tzN/7SqZ6j3xZ7SLMsp1d5jovAql4
PWvu4OhZulM3tXvJHTekM7ee0SOQR1jOQzwahisuvuKSDzQJYa8sWny2TXrWcbhz
fY2ssFkCgYEA9ybXE79Y/3/g3Fu+dLijB+eTbuJOLTDm+HYX4jo30uynlfv0AKTP
VCtWHJPy8Tkb1Fxs45L+gzTYdixrhLz83kC4OUaL5CXb2yUcBly+jFK8uWZXXTjq
I0p1H5GOZ2Imjtcw6L2krv1U8MxtLk0hxwqPLg8xT3HpMJo/1ZbD4m0CgYEA9F5m
ScMxwdlWa9E5dtUsx5KsvnjLl1a4VZim6w3t46HSRmhrFBpidPU7kLij779sVSQY
Rx5f/1su/YFAGqVtwxCM3H5KLLpQso5MJRXl1/IlwjSBYHqiCzGnkc4WKx0XV1/3
/d4Y7zicBTZ2UJOiGdEx2kQOuJcFPFU8I/iyQg8CgYEAyAX3K1RBgwbLxYu6qFyG
FW+mMqeU/Z4GUC/DRKQ5act+FjTDVYINCeHI33gdtnyxuTzUI5pjwWyTg5CPs+3/
+SAH+NLPhOXe+Y0fEUceMBMGCkZ5jkjxtX4dLF9xENquugwO2U4iaj088WWBN2fV
XnF9T5mcHt/iCiPMZeCOyEECgYEA4dJ2rkWerqf80Af6FZGsHwWxcxdH9SPjlt5J
qkAmDUzWd9A428wCHlkdYYDvpjd8kjWX5ejxB5apFwWhSr6Db1bVBVIDk8/dkRQk
08SnsWaJdC13PcQ2CSgq1XfgTplEn68FCmp7Gl5y9/I7Zfz4OOl0K2LnQ7fz06xk
tk011gsCgYEAhdA8OY+5xiFsOmYx2WX0/16J7CmeP2PqeD7qou3HhYn7LnBLAC2F
tEORLu1nv1dU9byWZwnrm8j64FecbxEa+uUd1WG6ucYksORzeh3fKOR9PCPXm7eL
FJeXrcexYNLg9k5+0YJTUzcyUcapvTQNZR+g8D93qUAo/0ykSqG7liA=
-----END RSA PRIVATE KEY-----
private_key_type    rsa
serial_number       57:8a:77:a0:45:8d:57:e3:da:2e:b5:77:9c:40:83:eb:a3:8b:03:b8
```

11. Отзыв сертификата

```sh
~ kubectl exec -it vault-0 -- vault write pki_int/revoke serial_number="57:8a:77:a0:45:8d:57:e3:da:2e:b5:77:9c:40:83:eb:a3:8b:03:b8"
Key                        Value
---                        -----
revocation_time            1644359579
revocation_time_rfc3339    2022-02-08T22:32:59.332853538Z
```
