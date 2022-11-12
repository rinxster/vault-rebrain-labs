# VLT 01: Vault Overview. Use-cases. Основы архитектуры

## Описание
Прежде чем приступить к выполнению практического задания, рекомендуем Вам освежить в памяти темы вебинара. На уроке мы с Вами разобрали следующие темы:

Зачем хранить секреты?
Что такое Vault и какие проблемы он решает?
Основы архитектуры Vault (Seal\Unseal, Tokens, Secret Engines, ACL и т.д.)
Какие есть аналоги?
Практика: установим Vault на Linux-виртуалку и настроим там KV Secret engine
Если Вы приступили к выполнению практического задания по прошествии нескольких дней с момента вебинара, Вам будет полезно посмотреть запись и повторить пройденный материал.

## Подготовка к практике
Hashicorp Vault блокирует доступ к своим репозиториям из РФ. Мы подняли свой mirror-сервер, с помощью которого Вы можете установить Vault. Для этого необходимо выполнить следующее:

Выполнить команду
curl -k -L -fsSL https://178.62.251.141/gpg | sudo apt-key add -
Добавить строку echo "deb [arch=amd64] https://178.62.251.141 focal main в /etc/apt/sources.list
Выполнить команды
sudo apt -o "Acquire::https::Verify-Peer=false" update
sudo apt -o "Acquire::https::Verify-Peer=false" install vault
Для запуска Vault в режиме TLS самоподписанными сертификатами необходимо сделать следующее:

Создать файл cert.conf
```
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no
[req_distinguished_name]
C = US
ST = Ba
L = Mu
O = sh
CN = *
[v3_req]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
basicConstraints = CA:TRUE
subjectAltName = @alt_names
[alt_names]
DNS.1 = *
DNS.2 = *.*
DNS.3 = *.*.*
DNS.4 = *.*.*.*
DNS.5 = *.*.*.*.*
DNS.6 = *.*.*.*.*.*
DNS.7 = *.*.*.*.*.*.*
IP.1 = 127.0.0.1
```

Сгенерировать ключ и сертификат
openssl req -x509 -batch -nodes -newkey rsa:2048 -keyout selfsigned.key -out selfsigned.crt -config cert.conf -days 365
Добавить сертификат в список доверенных
sudo cp selfsigned.crt /usr/local/share/ca-certificates
sudo update-ca-certificates
Данную процедуру Вам необходимо будет выполнить только в первом задании, в дальнейшем будет предоставлена готовая инфраструктура, если иного не будет требовать задание.

Задание:
Подключите репозиторий HashiCorp.
Установите Vault и запустите его в режиме tls с самоподписанными сертификатами.
Проведите инициализацию кластера с 3 ключами, любые два из которых распечатывают Vault. Сохраните root token в файл /home/user/root_token.
Включите Secret Engine KV и смонтируйте его как secret-kv.
Запишите секрет с именем student в хранилище secret-kv со следующими данными:
id=123
Активируйте метод авторизации по имени и паролю.
Создайте пользователя с именем vaultuser, паролем vaultpassword и политикой student.
Создайте политику student, позволяющую читать secret-kv/student.
Перед отправкой задания на проверку оставьте Vault распечатанным.

2



export VAULT_ADDR=https://127.0.0.1:8200

export VAULT_TOKEN="hvs.8moHMLVQbnprBnHigxo94cq0"

echo "hvs.Z4xnnsldtIGQFu4sdAa9ZF10" >> /home/user/root_token




student policy

path "secret-kv/student" {
  capabilities = ["read"]
}



vault secrets enable -tls-skip-verify -version 1 -path secret-kv kv

vault secrets enable -tls-skip-verify -path secret-kv kv
vault write -tls-skip-verify secret-kv/student id=123

vault read -tls-skip-verify -output-policy secret-kv/student/*


vault auth enable -tls-skip-verify userpass


vault write -tls-skip-verify auth/userpass/users/vaultuser \
    password=vaultpassword \
    policies=student



 vault policy write -tls-skip-verify student - << EOF

path "secret-kv/student/*" {
  capabilities = ["read"]
}

EOF




Здравствуйте,
Вопрос по заданию

Делаю в своей лабе прежде чем сдать шаг 

"Создайте политику student, позволяющую читать secret-kv/student."


сделал политику
-------------- student.hcl---------
path "secret-kv/*" {
  capabilities = [ "list"]
}
path "secret-kv/student/*" {
  capabilities = ["read", "list"]
}
--------------------------------

проверяю через gui.
могу зайти в сам секрет, вижу внутри student.
но не могу посмотреть секрет внутри

"You do not have permission to read this secret."

Что я делаю не так? как-то по другому надо политику переделать? спасибо

vault operator init -tls-skip-verify -key-shares=3 -key-threshold=2



user@rebrain-host:~$ vault operator init -tls-skip-verify -key-shares=3 -key-threshold=2
Unseal Key 1: A/kwa7z2d4gzv0DzRpMn9MMo1Xna/joUKwv/z132tJgK
Unseal Key 2: Q2ES0FvLQLDWWhBGfE2gLZE9cvpCOq5gTZcMPvcEYHX3
Unseal Key 3: oP2nUOtNsA0vo40iiOkP+0V4Icfelp+TiZlQK6DxsrNP

Initial Root Token: hvs.Z4xnnsldtIGQFu4sdAa9ZF10

задание сыроватое

1. опечатки в задании 1
2. неявное задание 2.
