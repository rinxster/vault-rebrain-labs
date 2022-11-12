# VLT 02: KV secret engine
## Описание
Прежде чем приступить к выполнению практического задания, рекомендуем Вам освежить в памяти темы вебинара. На уроке мы с Вами разобрали следующие темы:

Key-Value Secret Engine (версии и отличия) — выполним практику в текущем задании
Token (виды и особенности) — практика в следующем задании
ACL: пишем политики — практика в следующем задании
Если Вы приступили к выполнению практического задания по прошествии нескольких дней с момента вебинара, Вам будет полезно посмотреть запись и повторить пройденный материал.

Также для Вашего удобства прикладываем презентацию к вебинару (обратите внимание, что информация в ней понадобится для выполнения текущего и следующего практического задания).

## Подготовка к практике
Для начала работы с Vault выполните:
`sudo systemctl restart vault`

## Задание:
1. Проведите инициализацию кластера с 3 ключами, любые два из которых распечатывают Vault. Сохраните root token в файл /home/user/root_token.
```
export VAULT_ADDR=https://127.0.0.1:8200

sudo systemctl restart vault

vault operator init
export VAULT_ADDR=https://127.0.0.1:8200 && sudo systemctl restart vault && vault operator init -key-shares=3 -key-threshold=2 >> /home/user/vault_keys

export VAULT_TOKEN=<define your token here>

echo $VAULT_TOKEN >> /home/user/root_token

```
2. Создайте три secret-engine kv version2 с путями kv-production/, kv-staging/, kv-development/. Для kv-production/ включите обязательное использование флага cas, используя curl-запрос к API вместо CLI.
```
vault secrets enable -version=2 -path=kv-production/ kv

vault secrets enable -version=2 -path=kv-development/ kv

vault secrets enable -version=2 -path=kv-staging/ kv


vault read kv-production/config

vault write kv-production/config cas_required=true

(https://learn.hashicorp.com/tutorials/vault/versioned-kv)

CURL EXAMPLE

 tee payload-cas.json<<EOF
{
  "cas_required": true
}
EOF


curl --header "X-Vault-Token: $VAULT_TOKEN" \
    --request POST \
    --data @payload-cas.json \
    $VAULT_ADDR/v1/kv-production/config

vault read kv-production/config
```
3. В каждом secret-engine создайте секрет по пути db/mysql с ключами username=service password=passw. Внимание, проверяется первая версия.
```
vault kv put kv-development/db/mysql username=service password=passw

vault kv put kv-staging/db/mysql username=service password=passw

vault kv put -cas=0  kv-production/db/mysql username=service password=passw
```
4. В каждом secret-engine создайте секрет по пути db/mysql с ключами username=service2 password=passw2. Внимание, будет проверяться именно вторая версия ключей.
```
(((vault kv put kv-development/username username=service2 password=passw2
vault kv put kv-development/password username=service2 password=passw2)))

vault kv put kv-development/db/mysql username=service2 password=passw2

vault kv put kv-staging/db/mysql username=service2 password=passw2

vault kv put -cas=1  kv-production/db/mysql username=service2 password=passw2
```

5. Откатите версию db/mysql в kv-production до первой. Внимание, будет проверяться именно третья версия.
```
vault kv rollback -mount=kv-production/ -version=1 db/mysql
```
***

Перед проверкой оставьте Vault в состоянии unseal.