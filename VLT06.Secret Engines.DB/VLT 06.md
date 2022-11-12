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

pgadmin : pgpass
Логины и пароли для Mongo:

mongouser : mongopass
vault : vaultpass

* 2. Проведите инициализацию кластера с 3 ключами, любые два из которых распечатывают Vault. Сохраните root token в файл /home/user/root_token.

* 3. Включите SecretEngine database c mount point=database

https://developer.hashicorp.com/vault/docs/secrets/databases/mongodb

```

 vault secrets enable -path=database database

 vault write mongodb/config/mongo-test \
      plugin_name=mongodb-database-plugin \
      allowed_roles="tester" \
      connection_url="mongodb://{{username}}:{{password}}@$MONGODB_URL/admin?tls=false" \
      username="mdbadmin" \
      password="hQ97T9JJKZoqnFn2NXE"

```

* 4. Включить AuthMethod userpass с mount point db-users



* 5. Создайте пользователя mongo-dev с политикой mongo-dev и паролем password-mongo-dev.

* 6. Создайте пользователя postgres-dev с политикой postgres-dev и паролем password-postgres-dev.

* 7. Сконфигурируйте подключение к MongoDB в mount-point database с названием mongo-dev и следующими параметрами:

connection string - mongodb://{{username}}:{{password}}@[ип_баз_данных]:27017/admin?tls=false
username - mongouser
password - mongouser
allowed roles - mongo-dev-role
Сконфигурируйте подключение к Postgres в mount-point database с названием postgres-dev и следующими параметрами:
connection string - postgresql://{{username}}:{{password}}@[ваш_хостнейм_postgresql]:5432/postgres?sslmode=disable
username - vault
password - vaultpass
allowed roles - postgres-dev-role

* 8. Сконфигурируйте роль postgres-dev-role для инстанса postgres-dev
creation_statements="CREATE ROLE "{{name}}" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT USAGE ON SCHEMA stage_schema TO "{{name}}"; GRANT SELECT, UPDATE, INSERT ON ALL TABLES IN SCHEMA stage_schema TO "{{name}}";" \
default_ttl=30m
max_ttl=1h

* 9. Сконфигурируйте роль mongo-dev-role для инстанса mongo-dev
creation-statement = '{ "db": "admin", "roles": [{"role": "readWrite", "db": "dev-app"}] }'
default_ttl=30m
max_ttl=1h

* 10. Напишите политику с именем mongo-dev, которая будет позволять читать креды для mongo с помощью роли mongo-dev-role.

* 11. Напишите политику с именем postgres-dev, которая будет позволять читать креды для postgres с помощью роли postgres-dev-role.

