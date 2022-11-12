


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
