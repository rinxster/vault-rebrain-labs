# VLT 03: ACL
***
## Описание
Прежде чем приступить к выполнению практического задания, рекомендуем Вам освежить в памяти темы предыдущего вебинара. На уроке мы с Вами разобрали следующие темы:

Key-Value Secret Engine (версии и отличия) — эта практика была в прошлом задании
Token (виды и особенности)
ACL: пишем политики
Если Вы приступили к выполнению практического задания по прошествии нескольких дней с момента вебинара, Вам будет полезно посмотреть запись и повторить пройденный материал.
***
## Подготовка к практике
Для начала работы с Vault выполните:

`sudo systemctl restart vault`

***
## Задание:
### 1. Проведите инициализацию кластера с 3 ключами, любые два из которых распечатывают Vault. Сохраните root token в файл /home/user/root_token.
```
export VAULT_ADDR=https://127.0.0.1:8200 && sudo systemctl restart vault && vault operator init -key-shares=3 -key-threshold=2 >> /home/user/vault_keys


export VAULT_TOKEN=hvs.SR6BNLMuIzptLsMWew30a9G3 && echo $VAULT_TOKEN >> /home/user/root_token

```

### 2. Включите AuthMethod userpass по пути corp-auth. Установите максимальное время жизни токена - 5 часов, TTL - 1 час.
```
vault auth enable -max-lease-ttl="5h" -default-lease-ttl="1h" -path corp-auth userpass 
```

### 3. Напишите общую политику user, которая разрешает каждому пользователю СОЗДАВАТЬ, ОБНОВЛЯТЬ, ИЗМЕНЯТЬ, ЧИТАТЬ, УДАЛЯТЬ и просматривать списком секреты в secret engine kv-development в каталоге, равным его ID. У пользователя не должно быть прав на удаление своего каталога, только секретов в нем.

```
vault policy write -tls-skip-verify user - << EOF

path "kv-development/{{identity.entity.id}}" {
  capabilities = ["create", "read", "update", "patch", "list"]
}

path "kv-development/{{identity.entity.id}}/*" {
  capabilities = ["create", "delete", "read",]
}

EOF
```
```
vault policy write -tls-skip-verify user - << EOF


path "kv-development/{{identity.entity.id}}/*" {
  capabilities = ["create", "delete", "read", "update"]
}

EOF
```
### 4. Напишите политику dev-junior, которая разрешает ТОЛЬКО ЧТЕНИЕ ДАННЫХ секрета db/mysql в kv-development/ и kv-staging/ без доступа к метаданным.

```
vault policy write -tls-skip-verify dev-junior - << EOF

path "kv-development/db/mysql" {
  capabilities = ["read", "list"]
}

path "kv-staging/db/mysql" {
  capabilities = ["read", "list"]
}

path "kv-development/metadata/db/mysql" {
  capabilities = ["deny"]
}

path "kv-staging/metadata/db/mysql" {
  capabilities = ["deny"]
}
EOF
```
вариант
```

vault policy write -tls-skip-verify dev-junior - << EOF

path "kv-development/db/mysql" {
  capabilities = ["read"]
}

path "kv-staging/db/mysql" {
  capabilities = ["read"]
}

path "kv-development/metadata/*" {
  capabilities = ["deny"]
}

path "kv-staging/metadata/*" {
  capabilities = ["deny"]
}
EOF

```

### 5. Напишите политику devops-junior, которая разрешает ЧТЕНИЕ И ИЗМЕНЕНИЕ данных db/mysql в kv-development/ и kv-staging/. Полная перезапись должна быть запрещена.

```
vault policy write -tls-skip-verify devops-junior - << EOF

path "kv-development/db/mysql" {
  capabilities = ["read", "list", "update"]
}

path "kv-staging/db/mysql" {
  capabilities = ["read", "list", "update"]
}
EOF
```
новый вариант
```
vault policy write -tls-skip-verify devops-junior - << EOF

path "kv-development/data/db/mysql" {
   capabilities = ["read", "patch"]
}

path "kv-staging/data/db/mysql" {
   capabilities = ["read", "patch"]
}
EOF
```
### 6. Напишите политику dev-senior, которая разрешает ТОЛЬКО ЧТЕНИЕ данных секрета db/mysql в kv-development/, kv-staging/, kv-production/.

```
vault policy write -tls-skip-verify dev-senior - << EOF


path "kv-production/data/db/mysql" {
  capabilities = ["read"]
}


path "kv-development/data/db/mysql" {
  capabilities = ["read"]
}

path "kv-staging/data/db/mysql" {
  capabilities = ["read"]
}

EOF
```

### 7. Напишите политику devops-senior, которая разрешает ЧТЕНИЕ, ИЗМЕНЕНИЕ, ПЕРЕЗАПИСЬ и УДАЛЕНИЕ данных секрета db/mysql в kv-development/, kv-staging/, kv-production/.
```
vault policy write -tls-skip-verify devops-senior - << EOF

path "kv-production/data/db/mysql" {
  capabilities = ["create", "update", "patch", "read", "delete"]
}


path "kv-development/data/db/mysql" {
  capabilities = ["create", "update", "patch", "read", "delete"]
}

path "kv-staging/data/db/mysql" {
  capabilities = ["create", "update", "patch", "read", "delete"]
}

EOF
```
новый вариант
```
vault policy write -tls-skip-verify devops-senior - << EOF
path "kv-development/data/db/mysql" {
   capabilities = ["read", "patch", "update", "delete"]
}

path "kv-staging/data/db/mysql" {
   capabilities = ["read", "patch", "update", "delete"]
}

path "kv-production/data/db/mysql" {
   capabilities = ["read", "patch", "update", "delete"]
}
EOF
```

### 8. Создайте пользователей devops-junior, devops-senior, dev-senior, dev-junior и назначьте им соответствующую политику, а также политику user. Для dev-junior и devops-junior - установите максимальное время жизни токена 3 часа и TTL - 40 минут. Для всех пользователей установите пароль в формате password-[username], например для dev-junior: password-dev-junior

```
vault write auth/corp-auth/users/devops-junior policies=devops-junior policies=user  password=password-devops-junior 
vault write auth/corp-auth/users/devops-senior policies=devops-senior policies=user password=password-devops-senior 
vault write auth/corp-auth/users/dev-senior policies=dev-senior policies=user password=password-dev-senior
vault write auth/corp-auth/users/dev-junior policies=dev-junior policies=user password=password-dev-junior


vault write auth/corp-auth/users/devops-junior ttl=40m max_ttl=3h 
vault write auth/corp-auth/users/dev-junior ttl=40m max_ttl=3h
```

### 9. Создайте политику с именем partner, которая разрешает только чтение секрета db/mysql в kv-staging .
```

vault policy write -tls-skip-verify partner - << EOF

path "kv-staging/db/mysql" {
  capabilities = ["read", "list"]
}

EOF
```

### 10. Создайте token-роль с именем partner, которая разрешает использование ТОЛЬКО политики partner. Сделайте так, чтобы роль выпускала периодические токены с периодом, равным 3 часам.
```

vault write auth/token/roles/partner allowed_policies="partner" period=3h
```
***
Перед проверкой оставьте Vault в состоянии unseal.
