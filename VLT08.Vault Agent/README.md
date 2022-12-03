**п.1 Скачать репозиторий hydra и зайти в него**

`git clone --branch v1.11.10 --depth 1 https://github.com/ory/hydra.git`

**п.2 Изменить в файле contrib/quickstart/5-min/hydra.yml поле issuer на домен машины**

```
serve:
  cookies:
    same_site_mode: Lax

urls:
  self:
    issuer: http://xbslz.rbrvault.com:4444
  consent: http://127.0.0.1:3000/consent
  login: http://127.0.0.1:3000/login
  logout: http://127.0.0.1:3000/logout

secrets:
  system:
    - youReallyNeedToChangeThis

oidc:
  subject_identifiers:
    supported_types:
      - pairwise
      - public
    pairwise:
      salt: youReallyNeedToChangeThis
```

**п.3 Запустить сервис**

`sudo docker-compose -f quickstart.yml -f quickstart-postgres.yml -f quickstart-jwt.yml up --build -d`


**п.4 Создать пользователя**

```
sudo docker-compose -f quickstart.yml \
exec hydra hydra clients create \
--endpoint http://84.252.128.77:4445/ \
--id rebrain \
--secret secret \
-g client_credentials
```

**п.5 Получить JWT**

```
curl -s -k -X POST    -H "Content-Type: application/x-www-form-urlencoded"    -d grant_type=client_credentials    -u 'rebrain:secret'    http://84.252.128.77:4444/oauth2/token
```

**п.6 Провести инициализацию кластера с 3 ключами, любые два из которых распечатывают Vault.Сохранить root token в файл /home/user/root_token**

`export VAULT_SKIP_VERIFY=true`

`vault operator init -key-shares=3 -key-threshold=2`

`vault operator unseal`

`export VAULT_TOKEN=`

**п.7 Активировать авторизацию по JWT**

`vault auth enable jwt`

**п.8 Создать роль test для авторизации по JWT, токены должны иметь время жизни 2 часа и использовать политику default-jwt.Разрешить логиниться только токенам, у которых client_id (Поле из JWT) = rebrain.**

`vault write auth/jwt/config jwks_url="http://xbslz.rbrvault.com:4444/.well-known/jwks.json" bound_issuer="http://xbslz.rbrvault.com:4444/"`

```
vault write auth/jwt/role/test - <<EOF
{
  "role_type": "jwt",
  "policies": ["default-jwt"],
  "user_claim": "client_id",
  "bound_claims_type": "string",
  "bound_claims": {
    "client_id": "rebrain"
  },
  "token_ttl": "2h"
}
EOF
```

**п.9 Активировать авторизацию по AppRole**

`vault auth enable approle`

**п.10 Создать AppRole dev-partner с биндом на политику partner, роль должна генерировать периодические токены с периодом 3 часа**

`vault write auth/approle/role/dev-partner token_policies=partner token_period=3h`

**п.11 Сгенерировать role_id dev-partner и сохраните значение в файл /home/user/role_id**

`vault read auth/approle/role/dev-partner/role-id`

`echo "7f2785cd-4a08-d060-f0f9-d104d276d24f" > role_id`

**п.12 Создать wrapped token сроком жизни 120 минут для dev-partner и сохраните его в файл /home/user/wrapped_token**

`vault write -f -wrap-ttl=120m auth/approle/role/dev-partner/secret-id`

`echo "<wrapping_token>" > wrapped_token`


*VLT 08: Vault Agent
Описание:

Прежде чем приступить к выполнению практического задания, рекомендуем Вам освежить в памяти содержание вебинара:

изучили возможности Vault Agent
написали простые шаблоны для автоматизации получения секретов
Если Вы приступили к выполнению практического задания по прошествии нескольких дней с момента вебинара, Вам будет полезно посмотреть запись и повторить пройденный материал. Также для Вашего удобства прикладываем презентацию к вебинару.

Задание:

**1.Проведите инициализацию кластера с 3 ключами, любые два из которых распечатывают Vault. Сохраните root token в файл /home/user/root_token.

`export VAULT_SKIP_VERIFY=true`

`vault operator init -key-shares=3 -key-threshold=2`

`vault operator unseal`

`export VAULT_TOKEN=`

`export VAULT_ADDR=https://127.0.0.1:8200 && sudo systemctl restart vault && vault operator init -key-shares=3 -key-threshold=2 >> /home/user/vault_keys`

`touch root_token`

`echo "<put your token here from vault_keys file created above>" >> root_token`


2. Включите движок KVv2 по пути kv-v2.

`vault secrets enable -version=2 -path=kv-v2 kv`


3. Создайте секрет kv-v2/agent-test. Запишите в него данные password=s$cr3t.

`vault kv put kv-v2/agent-test password=s$cr3t`


4. Сделайте политику agent-read-only, которая разрешает ТОЛЬКО ЧТЕНИЕ секрета kv-v2/agent-test.

```
vault policy write -tls-skip-verify agent-read-only - << EOF

path "kv-v2/agent-test" {
  capabilities = ["read"]
}

EOF
```

5. Создайте AppRole с именем agent-role c политикой agent-read-only. Время жизни secret id — 1 час, время жизни токена - 1 минута, максимальное время жизни - 10 минут.

```
vault auth enable approle

vault write auth/approle/role/agent-role policies="agent-read-only" \
    secret_id_ttl=1h \
    token_ttl=1m \
    token_max_ttl=10m
```

6. Получите AppRole secret id и role id и запишите их в файлы /home/user/secret_id и /home/user/role_id соответственно.

`vault read auth/approle/role/agent-role/role-id`

`vault write -f auth/approle/role/agent-role/secret-id`


7. Сделайте шаблон, который будет выводить содержимое ключа password из секрета kv-v2/agent-test. Шаблон сохраните в файл /home/user/template.ctmpl

touch /home/user/template.ctmpl

{{ with secret "kv-v2/agent-test" }}
{{ .Data.data.password }}
{{ end }}

8. Сконфигурируйте и запустите vault agent. Он должен использовать AppRole для автоматической авторизации, сохраняя токен в файл /tmp/.vault-token, и записывать содержимое ключа password из секрета kv-v2/agent-test в файл /home/user/secret_auto_update, используя ранее созданный шаблон.

Используйте следюущий template_config:

template_config {
  static_secret_render_interval = "1s"
}
!!! ВНИМАНИЕ !!! При отправке задания на проверку vault agent должен быть активирован!

3
0