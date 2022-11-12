
# VLT 06: Secret Engines. DB

Описание:
Прежде чем приступить к выполнению практического задания, рекомендуем Вам освежить в памяти темы вебинара:

изучили динамические секреты на примере движка секретов Database
поговорили про систему плагинов
написали политику для настройки сложности пароля
Если Вы приступили к выполнению практического задания по прошествии нескольких дней с момента вебинара, Вам будет полезно посмотреть запись и повторить пройденный материал.

Также для Вашего удобства прикладываем презентацию к вебинару.

Задание:
* 1. В данном задании Вам предоставляется две машины. На первой установлен Vault, а второй — PostgreSQL и MongoDB.

Логины и пароли для Postgres:
```
pgadmin : pgpass

```
```
Логины и пароли для Mongo:

mongouser : mongopass
vault : vaultpass
```

* 1. Проведите инициализацию кластера с 3 ключами, любые два из которых распечатывают Vault. Сохраните root token в файл /home/user/root_token.

export VAULT_ADDR=https://127.0.0.1:8200 && sudo systemctl restart vault && vault operator init -key-shares=1 -key-threshold=1 >> /home/user/vault_keys

touch root_token

echo "hvs.7xSKW0pn2rRCRCPyxEApGt0L" >> root_token

export VAULT_SKIP_VERITY=true

* 2. Включите SecretEngine database c mount point=database

https://developer.hashicorp.com/vault/docs/secrets/databases/mongodb

```

 vault secrets enable -path=database database

 vault secrets enable database

 vault write mongodb/config/mongo-test \
      plugin_name=mongodb-database-plugin \
      allowed_roles="tester" \
      connection_url="mongodb://{{username}}:{{password}}@$MONGODB_URL/admin?tls=false" \
      username="mdbadmin" \
      password="hQ97T9JJKZoqnFn2NXE"

```

* 3. Включить AuthMethod userpass с mount point db-users
```
vault auth enable -path="db-users" userpass 
```



* 4. Создайте пользователя mongo-dev с политикой mongo-dev и паролем password-mongo-dev.
```
vault write auth/db-users/users/mongo-dev password="password-mongo-dev" policies="mongo-dev" 
```

* 5. Создайте пользователя postgres-dev с политикой postgres-dev и паролем password-postgres-dev.
```
vault write auth/db-users/users/postgres-dev password="password-postgres-dev" policies="postgres-dev"
```
* 6. Сконфигурируйте подключение к MongoDB в mount-point database с названием mongo-dev и следующими параметрами:

connection string - mongodb://{{username}}:{{password}}@[ип_баз_данных]:27017/admin?tls=false
username - mongouser
password - mongouser
allowed roles - mongo-dev-role

```

vault write database/config/mongo-dev plugin_name=mongodb-database-plugin \
allowed_roles="mongo-dev-role" connection_url='mongodb://{{username}}:{{password}}@158.160.12.32:27017/admin?tls=false' \
username="mongouser" \
password="mongopass"
```
check config after
```
vault read database/config/mongo-dev

```


* 7. Сконфигурируйте подключение к Postgres в mount-point database с названием postgres-dev и следующими параметрами:
connection string - postgresql://{{username}}:{{password}}@[ваш_хостнейм_postgresql]:5432/postgres?sslmode=disable
username - vault
password - vaultpass
allowed roles - postgres-dev-role
```
vault write database/config/postgres-dev plugin_name=postgresql-database-plugin \
connection_url='postgresql://{{username}}:{{password}}@158.160.12.32:5432/postgres?sslmode=disable' \
allowed_roles='postgres-dev-role' \
username='vault' \
password='vaultpass'
```
check how it is configured
```
vault read database/config/postgres-dev
```

* 8. Сконфигурируйте роль postgres-dev-role для инстанса postgres-dev
creation_statements="CREATE ROLE "{{name}}" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT USAGE ON SCHEMA stage_schema TO "{{name}}"; GRANT SELECT, UPDATE, INSERT ON ALL TABLES IN SCHEMA stage_schema TO "{{name}}";" \
default_ttl=30m
max_ttl=1h

```
vault write database/roles/postgres-dev-role db_name=postgres-dev \
creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}';
GRANT USAGE ON SCHEMA stage_schema TO \"{{name}}\";
GRANT  SELECT, UPDATE, INSERT ON ALL TABLES IN SCHEMA stage_schema TO \"{{name}}\";" \
default_ttl=30m \
max_ttl=1h


```

how to check role afterwards
```
vault read database/roles/postgres-dev-role

```



* 9. Сконфигурируйте роль mongo-dev-role для инстанса mongo-dev
creation-statement = '{ "db": "admin", "roles": [{"role": "readWrite", "db": "dev-app"}] }'
default_ttl=30m
max_ttl=1h


How to config role
```
vault write database/roles/mongo-dev-role db_name="mongo-dev" \
creation_statements='{"db":"admin","roles":[{"role":"readWrite", "db": "dev-app"}] }' \
default_ttl="30m" \
max_ttl="1h"
```
how to check role afterwards
```
vault read database/roles/mongo-dev-role

```

* 10. Напишите политику с именем mongo-dev, которая будет позволять читать креды для mongo с помощью роли mongo-dev-role.

```

 vault policy write -tls-skip-verify mongo-dev - << EOF

path "database/creds/mongo-dev-role" {
  capabilities = ["read"]
}

EOF
```

* 11. Напишите политику с именем postgres-dev, которая будет позволять читать креды для postgres с помощью роли postgres-dev-role.


```

 vault policy write -tls-skip-verify postgres-dev - << EOF

path "database/creds/postgres-dev-role" {
  capabilities = ["read"]
}

EOF
```

