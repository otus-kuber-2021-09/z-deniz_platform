
1. Установка Vault. Проверим установку

✗ helm status vault
NAME: vault
LAST DEPLOYED: Sun Jan 23 21:31:25 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing HashiCorp Vault!

2. Проведем инициализацию, сгенерируем токены:
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

3. Распечатали все ноды Vault

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

4. Залогинимся
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

5. Повторно запросим список авторизаций
✗ kubectl exec -it vault-0 --  vault auth list
Path      Type     Accessor               Description
----      ----     --------               -----------
token/    token    auth_token_303572c4    token based credentials

6. Создание секретов

kubectl exec -it vault-0 -- vault secrets enable --path=otus kv
Success! Enabled the kv secrets engine at: otus/

kubectl exec -it vault-0 -- vault secrets list --detailed
Path          Plugin       Accessor              Default TTL    Max TTL    Force No Cache    Replication    Seal Wrap    External Entropy Access    Options    Description                                                UUID
----          ------       --------              -----------    -------    --------------    -----------    ---------    -----------------------    -------    -----------                                                ----
cubbyhole/    cubbyhole    cubbyhole_a7c3b708    n/a            n/a        false             local          false        false                      map[]      per-token private secret storage                           f1215b3a-ac7a-e396-b91f-57da30b4c0b3
identity/     identity     identity_60572807     system         system     false             replicated     false        false                      map[]      identity store                                             02b14e36-5bb7-c467-3083-8a1cc0179615
otus/         kv           kv_4e5d73fc           system         system     false             replicated     false        false                      map[]      n/a                                                        fa90d7ca-5f53-7ca8-84c2-c52fe1319ba6
sys/          system       system_6bd4611d       n/a            n/a        false             replicated     false        false                      map[]      system endpoints used for control, policy and debugging    b26e7d8e-1e89-2fc9-e989-e4bd789adefc


kubectl exec -it vault-0 -- vault kv put otus/otus-ro/config username='otus' password='asajkjkahs'
Success! Data written to: otus/otus-ro/config

kubectl exec -it vault-0 -- vault kv put otus/otus-rw/config username='otus' password='asajkjkahs'
Success! Data written to: otus/otus-rw/config

kubectl exec -it vault-0 -- vault read otus/otus-ro/config
Key                 Value
---                 -----
refresh_interval    768h
password            asajkjkahs
username            otus

kubectl exec -it vault-0 -- vault kv get otus/otus-rw/config
====== Data ======
Key         Value
---         -----
password    asajkjkahs
username    otus

7. Включим авторизацию черерз k8s
kubectl exec -it vault-0 --  vault auth list
Path           Type          Accessor                    Description
----           ----          --------                    -----------
kubernetes/    kubernetes    auth_kubernetes_25775c70    n/a
token/         token         auth_token_303572c4         token based credentials
