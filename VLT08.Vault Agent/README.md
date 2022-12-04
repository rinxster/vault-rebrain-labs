**VLT 08: Vault Agent**

Описание:

Прежде чем приступить к выполнению практического задания, рекомендуем Вам освежить в памяти содержание вебинара:

изучили возможности Vault Agent
написали простые шаблоны для автоматизации получения секретов
Если Вы приступили к выполнению практического задания по прошествии нескольких дней с момента вебинара, Вам будет полезно посмотреть запись и повторить пройденный материал. Также для Вашего удобства прикладываем презентацию к вебинару.


Задание:

**1.Проведите инициализацию кластера с 3 ключами, любые два из которых распечатывают Vault. Сохраните root token в файл /home/user/root_token.

`export VAULT_SKIP_VERIFY=true`

`export VAULT_ADDR=https://127.0.0.1:8200 && sudo systemctl restart vault && vault operator init -key-shares=3 -key-threshold=2 >> /home/user/vault_keys`

`touch root_token`

`echo "<put your token here from vault_keys file created above>" >> root_token`

`vault operator unseal`


2. Включите движок KVv2 по пути kv-v2.

`vault secrets enable -version=2 -path=kv-v2 kv`


3. Создайте секрет kv-v2/agent-test. Запишите в него данные password=s$cr3t.

`vault kv put kv-v2/agent-test password=s$cr3t`


4. Сделайте политику agent-read-only, которая разрешает ТОЛЬКО ЧТЕНИЕ секрета kv-v2/agent-test.

```
vault policy write -tls-skip-verify agent-read-only - << EOF

path "kv-v2/data/agent-test" {
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

```
tee /home/user/template.ctmpl <<EOF

{{ with secret "kv-v2/agent-test" }}
{{ .Data.data.password }}
{{ end }}

EOF
```

8. Сконфигурируйте и запустите vault agent. Он должен использовать AppRole для автоматической авторизации, сохраняя токен в файл /tmp/.vault-token, и записывать содержимое ключа password из секрета kv-v2/agent-test в файл /home/user/secret_auto_update, используя ранее созданный шаблон.

Используйте следюущий template_config:
```
template_config {
  static_secret_render_interval = "1s"
}
```
!!! ВНИМАНИЕ !!! При отправке задания на проверку vault agent должен быть активирован!

Решение:

Готовим кониг и сохраняем в файл.
```
tee ./agent-config.hcl <<EOF

pid_file = "./pidfile"

vault {
  address = "https://127.0.0.1:8200"
}

auto_auth {
  method {
    type = "approle"

    config = {
      role_id_file_path = "/home/user/role_id"
      secret_id_file_path = "/home/user/secret_id"
      remove_secret_id_file_after_reading = false
    }
  }

  sink {
    type = "file"

    config = {
      path = "/tmp/.vault-token"
    }
  }
}

template {
  source = "/home/user/template.ctmpl"
  destination = "home/user/secret_auto_update"
}

template_config {
  static_secret_render_interval = "1s"
}
EOF

```
далее запускаем агент:

`vault agent -config ./agent-config.hcl`
