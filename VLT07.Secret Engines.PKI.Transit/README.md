# VLT 07: Secret Engines. PKI, Transit
*** 
## Описание:
Прежде чем приступить к выполнению практического задания, рекомендуем Вам освежить в памяти содержание вебинара:

разобрались с PKI: Public Key Infrastructure
сконфигурировали PKI Secret Engine
разобрались с Encryption as Service и Transit Secret Engine
Если Вы приступили к выполнению практического задания по прошествии нескольких дней с момента вебинара, Вам будет полезно посмотреть запись и повторить пройденный материал. Также для Вашего удобства прикладываем презентацию к вебинару.

## Перед практикой:
Прежде чем приступить к выполнению задания, создайте в каталоге /opt/certs файл bundle.pem со следующим содержимым.
```
-----BEGIN CERTIFICATE-----
MIIEHzCCAwegAwIBAgIUMjpzvgqs7MP8+HmsvUc5Hn4zj7IwDQYJKoZIhvcNAQEL
BQAwgZExCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQH
Ew1TYW4gRnJhbmNpc2NvMRYwFAYDVQQKEw1FeGFtcGxlLCBJbmMuMR0wGwYDVQQL
ExRSZWJyYWluIFZhdWx0IENvdXJzZTEeMBwGA1UEAxMVUmVicmFpbiBWYXVsdCBS
b290IENBMB4XDTIyMTAxNjEyMDMwMFoXDTQyMTAxNjEyMDMwMFowgZgxCzAJBgNV
BAYTAlJVMRYwFAYDVQQIEw1Nb3Njb3cgcmVnaW9uMQ8wDQYDVQQHEwZNb3Njb3cx
FzAVBgNVBAoTDkV4YW1wbGUgLCBJbmMuMScwJQYDVQQLEx5SZWJyYWluIENvdXJz
ZSBJbnRlcm1lZGlhdGUgQ0ExHjAcBgNVBAMTFVZhdWx0IEludGVybWVkaWF0ZSBD
QTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAJ4XtGtXNiziEzMdQNGF
8GfSnp/+g0U7JTvzBCcBbLDSVhdjKkuPU3V5jYMsW7IY4jayqB7cPs0OZEGrA97O
eVshezFaXwOgrEuGUQXluV9VONuCzJmrjMQAjydessV9Z/ko/0UQ4PKF/j4XpmBZ
MdfVGry7cP6oyrWDGkXa3kucBBb/sckf5QSAkjMp68in7JWJeA5BGqvooKW69bzy
INRne+btbz0d8R88etn2UXs2RVm91v/KQ21R/fbDSAS+P6+xBhblmv6vCxkmb5QX
tpIZQ/op6dgtG6K5yL1iDRNJHn4Z7R9zdc3n1UbD5y4JsRRJ+VWDI9fieo3vZH6D
PCkCAwEAAaNmMGQwDgYDVR0PAQH/BAQDAgEGMBIGA1UdEwEB/wQIMAYBAf8CAQAw
HQYDVR0OBBYEFEg3T7qbQeuUPjNUjGQtluIXLYNuMB8GA1UdIwQYMBaAFK2ZXrA5
0maoozwNrcQqtFkkEgwiMA0GCSqGSIb3DQEBCwUAA4IBAQAcbLF9o5xggGll8b8u
irlwbG73h6IrQuCqpzB8OtygLPQaozWTKfnLHfdnFXZAXvcGh9gDiM+IuJ5/4jXn
fBV/GxkjhGSH2/1fymsADJ9DrZFMcdwqsWWMW6tI87+kDb4Z2yimzD20l+XpDOQu
8sCz/rRZhNB4S75W2HKtPVcA2PdH6ph39HyzzJ5CLsWGo4vv8RXMXz/AgzHDHkVF
zej7JSQF5E04GaUbypeEsvsYVs3yEiFTOqm9Tekv4keNWll3sweShOTI891Njmvw
04yQu2WsFeoni/Tb+dZgOtb/zuxuqqpemgrHDMmSFYGZmVYYtviLWP2JqSb0tPso
ygHx
-----END CERTIFICATE-----
```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAnhe0a1c2LOITMx1A0YXwZ9Ken/6DRTslO/MEJwFssNJWF2Mq
S49TdXmNgyxbshjiNrKoHtw+zQ5kQasD3s55WyF7MVpfA6CsS4ZRBeW5X1U424LM
mauMxACPJ16yxX1n+Sj/RRDg8oX+PhemYFkx19UavLtw/qjKtYMaRdreS5wEFv+x
yR/lBICSMynryKfslYl4DkEaq+igpbr1vPIg1Gd75u1vPR3xHzx62fZRezZFWb3W
/8pDbVH99sNIBL4/r7EGFuWa/q8LGSZvlBe2khlD+inp2C0bornIvWINE0kefhnt
H3N1zefVRsPnLgmxFEn5VYMj1+J6je9kfoM8KQIDAQABAoIBAHhJnwxhUiY6adNl
ebEyUSYd+nXQCH9/rif8Eve+vL2ZfMnUuRS+3AixUPwynx5WkqB9tS+t8tbBEYVp
oss/nNS7F+oIUe0HrrDUZQewsCgaRuW2kwiFn9hueH3DLxDXB2psSDZ7zjyZuUXz
ZrM+io8nZW2ezS3mrj4Hn9Dw5FzwCXa5V/FDXuJf7DLZM090BAuPm1Du3f3dgm2O
RmM8T/yIjSb+8Cv66Lf2VmL5l15Q2IIaRPKdLGSgtZKVumEh90tLFpQeW17QMFqC
FUaAUL1tebSw4w3JGXPcueZ3qEfQlSwDSzVN4ey1Rc57v4r3QekF7egZr1ExCM4A
VqL3v3UCgYEAxXEFSv+AfpqBL99eajzWodTAfE/t0nubLWQPODw9Z91Puytoa9Wm
l60kPaNDuBXP/3sFzpD9Zttb1Syi8aYr5RV3GCqhV67WPWTP1ow6gJQ87+CQZago
5YrFOUZOFwQGao0Kidw8GY8hlcOdDtgu5PnThO+NG/mnzPZqW1Ve9s8CgYEAzPsS
p9SbhfKRzyQEZg+t2bPVipsUlnC8LQPDXddE/4/NRw1ScvQAB5zS0ofJf+WMwywj
fqKsElPtt95nTYzXZvQD+8Hy1Q5kdYgsr6CBlJwq1GeUeelIBKOoM1jnRa7x5A7q
j7lPXCZXYoV4KlB4ZF377Nf519InDnZn0NH024cCgYAgOsTMa0zEXeA8uk+lM+0t
WZdaM4n00+yOykiZu2uiqsO7H+jZwXSCSecikKYbRKRBZgmaoJxcz+37rF+k5qU/
rfNU5JCVyZp7RxuOQDHEj24rEhNAJOUYI0DyioFwzF1nw0I3ItZErdKjqdzXcX6m
LgnTJ293Y5d6o7bU1ei8jQKBgBZsLJlBT5Xyd/LBzN1hP7I90tErr6/ZOyxtafSc
9MZD87+e/HLosAwlIoa3JdqgwKok7OkQYGRM3AcuA/zeuD1h2gGzMJ4Pyft1XvYD
R8l639CGWB6R3zfqsx6SzhG4VmuNGimIqt64rvxu/zsZvGG2SjWZVpI+Qdl6KFcW
cIOHAoGBAJ9MIDt5cGWa1LDdzK9Xf1V7bhv1TOX/bseDtt+1qoKCLcQOzWw2W1v/
CxzAa39lzTqcFdriANUiAOsf37VHkTNFE39b0A6/rIhUW44vOkTNZH6MniCIRqgw
GFlCoJYukGBTqgMg+P/5OHXkQEY/rQKqpCEcf6e64yZMb6TjIcUD
-----END RSA PRIVATE KEY-----

***
Задание:

### 1. Проведите инициализацию кластера с 3 ключами, любые два из которых распечатывают Vault. Сохраните root token в файл /home/user/root_token.

### 2. Включите PKI SecretEngine по пути rebrain-pki/ с максимальным сроком жизни сертификата 1 год (8760 часов).

### 3. Импортируйте ранее созданный bundle.pem в секрет rebrain-pki/config/ca.

### 4. Создайте роль с именем local-certs, которая разрешит выпуск сертификатов сроком жизни 24 часа для всех поддоменов домена rebrain-vault.local (включая сам этот домен). Роль также должна ЗАПРЕЩАТЬ выпускать сертификаты для localhost, IP-адресов, а также wildcard-сертификаты.

### 5. Сделайте политику с именем cert-issue-policy, которая позволяет выпускать сертификаты с ролью local-certs.

### 6. Выпустите токен с этой политикой (срок жизни и другие параметры не важны) и запишите токен в файл /home/user/cert_issuer_token.

### 7. Выпустите сертификат с common_name="rebrain-vault.local" alt_names="random.rebrain-vault.local". Сохраните его в файл /home/user/cert.pem.

### 8. Запустите dev-server Vault на 9200/TCP порту в бэкграунде любым удобным способом (screen, nohup и т.д.). Используйте команду vault server -dev -dev-root-token-id=1 -dev-listen-address=0.0.0.0:9200. Она запустит dev-сервер с root-токеном "1", который необходимо использовать для дальнейшей настройки.

### 9. В dev-сервере смонтируйте SecretEngine Transit (не забудьте поменять env VAULT_ADDR и VAULT_TOKEN) как transit-autounseal и создайте ключ vault-autounseal.

### 10. В dev-сервере создайте политику transit-autounseal со следующим содержимым:
```
path "transit-autounseal/encrypt/vault-autounseal" {
   capabilities = [ "update" ]
}

path "transit-autounseal/decrypt/vault-autounseal" {
   capabilities = [ "update" ]
}
```
### 10 .Создайте периодический токен для этой политики с периодом 24 часа.

### 11. Добавьте конфигурацию seal в основной инстанс. Подставьте Ваши значения токена в соотвествующие поля.
```
seal "transit" {
  address            = ""
  token              = ""
  key_name           = "vault-autounseal"
  mount_path         = "transit-autounseal"
  tls_skip_verify    = "true"
}
```
Перезапустите vault и выполните команду vault operator unseal -migrate.
***
ВАЖНО! Перед отправкой задания на проверку перезагрузите основной инстанс vault-сервера (systemctl restart vault). Убедитесь, что unseal происходит автоматически!






---

sudo mkdir /opt/certs

sudo tee /opt/certs/bundle.pem << EOF 
-----BEGIN CERTIFICATE-----
MIIEHzCCAwegAwIBAgIUMjpzvgqs7MP8+HmsvUc5Hn4zj7IwDQYJKoZIhvcNAQEL
BQAwgZExCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQH
Ew1TYW4gRnJhbmNpc2NvMRYwFAYDVQQKEw1FeGFtcGxlLCBJbmMuMR0wGwYDVQQL
ExRSZWJyYWluIFZhdWx0IENvdXJzZTEeMBwGA1UEAxMVUmVicmFpbiBWYXVsdCBS
b290IENBMB4XDTIyMTAxNjEyMDMwMFoXDTQyMTAxNjEyMDMwMFowgZgxCzAJBgNV
BAYTAlJVMRYwFAYDVQQIEw1Nb3Njb3cgcmVnaW9uMQ8wDQYDVQQHEwZNb3Njb3cx
FzAVBgNVBAoTDkV4YW1wbGUgLCBJbmMuMScwJQYDVQQLEx5SZWJyYWluIENvdXJz
ZSBJbnRlcm1lZGlhdGUgQ0ExHjAcBgNVBAMTFVZhdWx0IEludGVybWVkaWF0ZSBD
QTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAJ4XtGtXNiziEzMdQNGF
8GfSnp/+g0U7JTvzBCcBbLDSVhdjKkuPU3V5jYMsW7IY4jayqB7cPs0OZEGrA97O
eVshezFaXwOgrEuGUQXluV9VONuCzJmrjMQAjydessV9Z/ko/0UQ4PKF/j4XpmBZ
MdfVGry7cP6oyrWDGkXa3kucBBb/sckf5QSAkjMp68in7JWJeA5BGqvooKW69bzy
INRne+btbz0d8R88etn2UXs2RVm91v/KQ21R/fbDSAS+P6+xBhblmv6vCxkmb5QX
tpIZQ/op6dgtG6K5yL1iDRNJHn4Z7R9zdc3n1UbD5y4JsRRJ+VWDI9fieo3vZH6D
PCkCAwEAAaNmMGQwDgYDVR0PAQH/BAQDAgEGMBIGA1UdEwEB/wQIMAYBAf8CAQAw
HQYDVR0OBBYEFEg3T7qbQeuUPjNUjGQtluIXLYNuMB8GA1UdIwQYMBaAFK2ZXrA5
0maoozwNrcQqtFkkEgwiMA0GCSqGSIb3DQEBCwUAA4IBAQAcbLF9o5xggGll8b8u
irlwbG73h6IrQuCqpzB8OtygLPQaozWTKfnLHfdnFXZAXvcGh9gDiM+IuJ5/4jXn
fBV/GxkjhGSH2/1fymsADJ9DrZFMcdwqsWWMW6tI87+kDb4Z2yimzD20l+XpDOQu
8sCz/rRZhNB4S75W2HKtPVcA2PdH6ph39HyzzJ5CLsWGo4vv8RXMXz/AgzHDHkVF
zej7JSQF5E04GaUbypeEsvsYVs3yEiFTOqm9Tekv4keNWll3sweShOTI891Njmvw
04yQu2WsFeoni/Tb+dZgOtb/zuxuqqpemgrHDMmSFYGZmVYYtviLWP2JqSb0tPso
ygHx
-----END CERTIFICATE-----
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAnhe0a1c2LOITMx1A0YXwZ9Ken/6DRTslO/MEJwFssNJWF2Mq
S49TdXmNgyxbshjiNrKoHtw+zQ5kQasD3s55WyF7MVpfA6CsS4ZRBeW5X1U424LM
mauMxACPJ16yxX1n+Sj/RRDg8oX+PhemYFkx19UavLtw/qjKtYMaRdreS5wEFv+x
yR/lBICSMynryKfslYl4DkEaq+igpbr1vPIg1Gd75u1vPR3xHzx62fZRezZFWb3W
/8pDbVH99sNIBL4/r7EGFuWa/q8LGSZvlBe2khlD+inp2C0bornIvWINE0kefhnt
H3N1zefVRsPnLgmxFEn5VYMj1+J6je9kfoM8KQIDAQABAoIBAHhJnwxhUiY6adNl
ebEyUSYd+nXQCH9/rif8Eve+vL2ZfMnUuRS+3AixUPwynx5WkqB9tS+t8tbBEYVp
oss/nNS7F+oIUe0HrrDUZQewsCgaRuW2kwiFn9hueH3DLxDXB2psSDZ7zjyZuUXz
ZrM+io8nZW2ezS3mrj4Hn9Dw5FzwCXa5V/FDXuJf7DLZM090BAuPm1Du3f3dgm2O
RmM8T/yIjSb+8Cv66Lf2VmL5l15Q2IIaRPKdLGSgtZKVumEh90tLFpQeW17QMFqC
FUaAUL1tebSw4w3JGXPcueZ3qEfQlSwDSzVN4ey1Rc57v4r3QekF7egZr1ExCM4A
VqL3v3UCgYEAxXEFSv+AfpqBL99eajzWodTAfE/t0nubLWQPODw9Z91Puytoa9Wm
l60kPaNDuBXP/3sFzpD9Zttb1Syi8aYr5RV3GCqhV67WPWTP1ow6gJQ87+CQZago
5YrFOUZOFwQGao0Kidw8GY8hlcOdDtgu5PnThO+NG/mnzPZqW1Ve9s8CgYEAzPsS
p9SbhfKRzyQEZg+t2bPVipsUlnC8LQPDXddE/4/NRw1ScvQAB5zS0ofJf+WMwywj
fqKsElPtt95nTYzXZvQD+8Hy1Q5kdYgsr6CBlJwq1GeUeelIBKOoM1jnRa7x5A7q
j7lPXCZXYoV4KlB4ZF377Nf519InDnZn0NH024cCgYAgOsTMa0zEXeA8uk+lM+0t
WZdaM4n00+yOykiZu2uiqsO7H+jZwXSCSecikKYbRKRBZgmaoJxcz+37rF+k5qU/
rfNU5JCVyZp7RxuOQDHEj24rEhNAJOUYI0DyioFwzF1nw0I3ItZErdKjqdzXcX6m
LgnTJ293Y5d6o7bU1ei8jQKBgBZsLJlBT5Xyd/LBzN1hP7I90tErr6/ZOyxtafSc
9MZD87+e/HLosAwlIoa3JdqgwKok7OkQYGRM3AcuA/zeuD1h2gGzMJ4Pyft1XvYD
R8l639CGWB6R3zfqsx6SzhG4VmuNGimIqt64rvxu/zsZvGG2SjWZVpI+Qdl6KFcW
cIOHAoGBAJ9MIDt5cGWa1LDdzK9Xf1V7bhv1TOX/bseDtt+1qoKCLcQOzWw2W1v/
CxzAa39lzTqcFdriANUiAOsf37VHkTNFE39b0A6/rIhUW44vOkTNZH6MniCIRqgw
GFlCoJYukGBTqgMg+P/5OHXkQEY/rQKqpCEcf6e64yZMb6TjIcUD
-----END RSA PRIVATE KEY-----
EOF
`sudo chmod 644 /opt/certs/bundle.pem`



1. Проведите инициализацию кластера с 3 ключами, любые два из которых распечатывают Vault. Сохраните root token в файл /home/user/root_token.


`export VAULT_SKIP_VERIFY=true && export VAULT_ADDR=https://127.0.0.1:8200 && sudo systemctl restart vault && vault operator init -key-shares=1 -key-threshold=1 >> /home/user/vault_keys`

`cat /home/user/vault_keys`

`vault operator unseal`

`export VAULT_TOKEN="hvs.IQAqzl5PdqRt9Qw0oL2y8k2y" && echo $VAULT_TOKEN > /home/user/root_token`



2.Включите PKI SecretEngine по пути rebrain-pki/ с максимальным сроком жизни сертификата 1 год (8760 часов)

`vault secrets enable -path=rebrain-pki -max-lease-ttl=8760h pki`

3.Импортируйте ранее созданный bundle.pem в секрет rebrain-pki/config/ca.

`vault write rebrain-pki/config/ca pem_bundle=@/opt/certs/bundle.pem`

`vault write rebrain-pki/import/bundle pem_bundle=@/opt/certs/bundle.pem`

vault write rebrain-pki/config/urls issuing_certificates="http://127.0.0.1:8200/v1/pki/ca" crl_distribution_point="http://127.0.0.1:8200/v1/pki/crl"

https://gruchalski.com/posts/2020-09-09-multi-tenant-vault-pki-with-custom-root-pem-bundle/

4.Создайте роль с именем local-certs, которая разрешит выпуск сертификатов сроком жизни 24 часа для всех поддоменов домена rebrain-vault.local (включая сам этот домен). Роль также должна ЗАПРЕЩАТЬ выпускать сертификаты для localhost, IP-адресов, а также wildcard-сертификаты.
```
vault write rebrain-pki/roles/local-certs \
allowed_domains="rebrain-vault.local" \
allow_subdomains=true \
allow_wildcard_certificates=false \
allow_localhost=false \
allow_glob_domains=true \
enforce_hostnames=true \
allow_ip_sans=false \
allow_client=true \
period="24h" \
allow_server=false \
enforce_hostname=false \
allow_client=true \
allow_any_name=true \
allow_bare_domains=true
```
5.Сделайте политику с именем cert-issue-policy, которая позволяет выпускать сертификаты с ролью local-certs

// `vault policy write -tls-skip-verify cert-issue-policy - << EOF
//path "rebrain-pki/roles/local-certs/*" 
//{ 
//   capabilities = ["list", "read", "create", "update", "delete", "sudo"] 
//   } 
//EOF`

//echo 'path "rebrain-pki/*" {
//  capabilities = ["list", "sudo", "read","create","update"]
//}' | vault policy write cert-issue-policy -


echo 'path "rebrain-pki/issue/local-certs" {
  capabilities = ["create", "update"]
}' | vault policy write cert-issue-policy -


//echo 'path "rebrain-pki/issue/*" {
//  capabilities = ["list", "sudo", "read","create","update"]
//}
//path "rebrain-pki/certs" {
//  capabilities = ["list", "sudo", "read","create","update"]
//}
//path "rebrain-pki/revoke" {
//      capabilities = ["create", "update"]
//    }
//path "rebrain-pki/tidy" {
//      capabilities = ["create", "update"]
//    }
//path "rebrain-pki/cert/ca" {
//      capabilities = ["read"]
//    }
//path "auth/token/renew" {
//      capabilities = ["update"]
//    }
//path "auth/token/renew-self" {
//      capabilities = ["update"]
//    }' | vault policy write cert-issue-policy -

https://medium.com/hashicorp-engineering/pki-as-a-service-with-hashicorp-vault-a8d075ece9a
https://www.ibm.com/docs/en/cloud-private/3.2.0?topic=manager-using-vault-issue-certificates

https://www.hashicorp.com/blog/certificate-management-with-vault

6.Выпустите токен с этой политикой (срок жизни и другие параметры не важны) и запишите токен в файл /home/user/cert_issuer_token

`vault token create -policy=cert-issue-policy`

`echo hvs.CAESID0z7vxBXFezHQkWozfNlJiKOipjA6boiNi57_eMviYOGh4KHGh2cy5Ld1JvME9MZ3FPbE0xTmJIVE1TUnYzMU4 > /home/user/cert_issuer_token`

7. Выпустите сертификат с common_name="rebrain-vault.local" alt_names="random.rebrain-vault.local". Сохраните его в файл /home/user/cert.pem

//`cat <> payload-7.json { "common_name": "rebrain-vault.local", "alt_names": "random.rebrain-vault.local" } EOT`

// `curl --header "X-Vault-Token: ..." --request POST --data @payload-7.json http://127.0.0.1:8200/v1/rebrain-pki/issue/local-certs`

// curl --header "X-Vault-Token: hvs.CAESIIi4g-i-pz8bQwrwta323FZvqus7fto34Z6uuXBDpoJuGh4KHGh2cy5yRHd1QUxwYXBibUJOT1BVbnhveTFnSHo" --request POST --data @payload-7.json http://127.0.0.1:8200/v1/rebrain-pki/issue/local-certs

`CERT > /home/user/cert.pem` 

 curl \
    --header "X-Vault-Token: hvs.CAESID0z7vxBXFezHQkWozfNlJiKOipjA6boiNi57_eMviYOGh4KHGh2cy5Ld1JvME9MZ3FPbE0xTmJIVE1TUnYzMU4" \
    --request POST \
    --data  '{ "common_name": "rebrain-vault.local", "alt_names": "random.rebrain-vault.local" }' \
    https://127.0.0.1:8200/v1/rebrain-pki/issue/local-certs


// vault write rebrain-pki/issue/local-certs common_name=rebrain-vault.local alt_names=random.rebrain-vault.local

// vault write rebrain-pki/roles/local-certs allow_server=false enforce_hostnames=false allow_client=true allow_any_name=true allow_bare_domains=true

got bug:
https://github.com/issacg/vault-pki-client



8.Запустите dev-server Vault на 9200/TCP порту в бэкграунде любым удобным способом (screen, nohup и т.д.). Используйте команду vault server -dev -dev-root-token-id=1 -dev-listen-address=0.0.0.0:9200. Она запустит dev-сервер с root-токеном "1", который необходимо использовать для дальнейшей настройки.

`vault server -dev -dev-root-token-id=1 -dev-listen-address=0.0.0.0:9200`

9. В dev-сервере смонтируйте SecretEngine Transit (не забудьте поменять env VAULT_ADDR и VAULT_TOKEN) как transit-autounseal и создайте ключ vault-autounseal.

10. В dev-сервере создайте политику transit-autounseal со следующим содержимым:
```
path "transit-autounseal/encrypt/vault-autounseal" {
   capabilities = [ "update" ]
}

path "transit-autounseal/decrypt/vault-autounseal" {
   capabilities = [ "update" ]
}
```

11. Создайте периодический токен для этой политики с периодом 24 часа.

12. Добавьте конфигурацию seal в основной инстанс. Подставьте Ваши значения токена в соотвествующие поля.
```
seal "transit" {
  address            = ""
  token              = ""
  key_name           = "vault-autounseal"
  mount_path         = "transit-autounseal"
  tls_skip_verify    = "true"
}
```

13. Перезапустите vault и выполните команду vault operator unseal -migrate.

ВАЖНО! Перед отправкой задания на проверку перезагрузите основной инстанс vault-сервера (systemctl restart vault). Убедитесь, что unseal происходит автоматически!





{"request_id":"748e39b8-c995-c17b-98a0-889b21034faa","lease_id":"","renewable":false,"lease_duration":0,"data":{"ca_chain":["-                                 ----BEGIN CERTIFICATE-----\nMIIEHzCCAwegAwIBAgIUMjpzvgqs7MP8+HmsvUc5Hn4zj7IwDQYJKoZIhvcNAQEL\nBQAwgZExCzAJBgNVBAYTAlVTMRMwEQYD                                 VQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQH\nEw1TYW4gRnJhbmNpc2NvMRYwFAYDVQQKEw1FeGFtcGxlLCBJbmMuMR0wGwYDVQQL\nExRSZWJyYWluIFZhdWx0IENvdX                                 JzZTEeMBwGA1UEAxMVUmVicmFpbiBWYXVsdCBS\nb290IENBMB4XDTIyMTAxNjEyMDMwMFoXDTQyMTAxNjEyMDMwMFowgZgxCzAJBgNV\nBAYTAlJVMRYwFAYDVQQI                                 Ew1Nb3Njb3cgcmVnaW9uMQ8wDQYDVQQHEwZNb3Njb3cx\nFzAVBgNVBAoTDkV4YW1wbGUgLCBJbmMuMScwJQYDVQQLEx5SZWJyYWluIENvdXJz\nZSBJbnRlcm1lZG                                 lhdGUgQ0ExHjAcBgNVBAMTFVZhdWx0IEludGVybWVkaWF0ZSBD\nQTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAJ4XtGtXNiziEzMdQNGF\n8GfSnp/+                                 g0U7JTvzBCcBbLDSVhdjKkuPU3V5jYMsW7IY4jayqB7cPs0OZEGrA97O\neVshezFaXwOgrEuGUQXluV9VONuCzJmrjMQAjydessV9Z/ko/0UQ4PKF/j4XpmBZ\nMd                                 fVGry7cP6oyrWDGkXa3kucBBb/sckf5QSAkjMp68in7JWJeA5BGqvooKW69bzy\nINRne+btbz0d8R88etn2UXs2RVm91v/KQ21R/fbDSAS+P6+xBhblmv6vCxkmb5                                 QX\ntpIZQ/op6dgtG6K5yL1iDRNJHn4Z7R9zdc3n1UbD5y4JsRRJ+VWDI9fieo3vZH6D\nPCkCAwEAAaNmMGQwDgYDVR0PAQH/BAQDAgEGMBIGA1UdEwEB/wQIMAYB                                 Af8CAQAw\nHQYDVR0OBBYEFEg3T7qbQeuUPjNUjGQtluIXLYNuMB8GA1UdIwQYMBaAFK2ZXrA5\n0maoozwNrcQqtFkkEgwiMA0GCSqGSIb3DQEBCwUAA4IBAQAcbL                                 F9o5xggGll8b8u\nirlwbG73h6IrQuCqpzB8OtygLPQaozWTKfnLHfdnFXZAXvcGh9gDiM+IuJ5/4jXn\nfBV/GxkjhGSH2/1fymsADJ9DrZFMcdwqsWWMW6tI87+k                                 Db4Z2yimzD20l+XpDOQu\n8sCz/rRZhNB4S75W2HKtPVcA2PdH6ph39HyzzJ5CLsWGo4vv8RXMXz/AgzHDHkVF\nzej7JSQF5E04GaUbypeEsvsYVs3yEiFTOqm9Te                                 kv4keNWll3sweShOTI891Njmvw\n04yQu2WsFeoni/Tb+dZgOtb/zuxuqqpemgrHDMmSFYGZmVYYtviLWP2JqSb0tPso\nygHx\n-----END CERTIFICATE-----"                                 ],"certificate":"-----BEGIN CERTIFICATE-----\nMIIEMTCCAxmgAwIBAgIUZPB9HnPg31OMP+T1AT/Mqk6LGLcwDQYJKoZIhvcNAQEL\nBQAwgZgxCzAJBg                                 NVBAYTAlJVMRYwFAYDVQQIEw1Nb3Njb3cgcmVnaW9uMQ8wDQYD\nVQQHEwZNb3Njb3cxFzAVBgNVBAoTDkV4YW1wbGUgLCBJbmMuMScwJQYDVQQLEx5S\nZWJyYWlu                                 IENvdXJzZSBJbnRlcm1lZGlhdGUgQ0ExHjAcBgNVBAMTFVZhdWx0IElu\ndGVybWVkaWF0ZSBDQTAeFw0yMjEyMDQxOTQwNDZaFw0yMzAxMDUxOTQxMTVaMB4x\nHD                                 AaBgNVBAMTE3JlYnJhaW4tdmF1bHQubG9jYWwwggEiMA0GCSqGSIb3DQEBAQUA\nA4IBDwAwggEKAoIBAQDTFmNMPqHzBQQHPUBwoHA1zPpZ3MTdW2FbMrlZIwiwwR                                 0X\nx1uAhtQiK5+3UVIY++AY/EQaj3UaS4rv0xwQYBSRDqBhuHpiw5YmWnPIIMCxmE4l\nF49+jf8NnZg8LeoGKWX7ATtMFMc1mxK/NugkaSusIFwmZ6RAUqnWkwzV                                 E5HueF3k\nCjUcmpHvZp8IvKwglkTJ+ED21dePgZDxQD/iCpPZWmL8Db135PGYUFuQ4jzedU6N\nq32+j3qYzrEBEvJmvQsIqM3hHh/8Y59QcKzYbtibtx0nVmo9Yv                                 a7mdrcKSY+GuP3\nXSiw4YrVFjeIFKC2xKZhuDHwHG38rm0wRGQKQVGLAgMBAAGjgeswgegwDgYDVR0P\nAQH/BAQDAgOoMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggr                                 BgEFBQcDAjAdBgNVHQ4E\nFgQUMuA/UmbDe3Iu/BllOr+GxiO/9DkwHwYDVR0jBBgwFoAUSDdPuptB65Q+M1SM\nZC2W4hctg24wOwYIKwYBBQUHAQEELzAtMCsGCC                                 sGAQUFBzAChh9odHRwOi8vMTI3\nLjAuMC4xOjgyMDAvdjEvcGtpL2NhMDoGA1UdEQQzMDGCGnJhbmRvbS5yZWJyYWlu\nLXZhdWx0LmxvY2FsghNyZWJyYWluLXZh                                 dWx0LmxvY2FsMA0GCSqGSIb3DQEBCwUA\nA4IBAQBZxm5Q9I0AUrgU022teo+Hi6JbThCfGhu7v+NSC8KCAkQ8FxPNNovp9QwX\n1TeKYWWiB1dPzR2c/kt75zFaSm                                 X5NsPsxC4fUvGyifKqkHlvg165243y72cKg4XG\n255boTBThKjYLC2TrQCOxIt02XfHf0Y3K3XxX1x7ky3ZRO2iubmHbx9KRfI2McAk\nQ4j5k2SfLmJiHnOUXGw/                                 j1IJ8ELfzUxHsP+0qE0FKS/iDtNffb4CKbdrDz/9Pxa+\nVcet+scowwA2ULGTM7z/tkUuINFm7affqzcjTXRHprdHE4XACW2U5alA3dEY+rK6\n5YTIEWQ7JU/hlF                                 GL3hz22uhVIz23\n-----END CERTIFICATE-----","expiration":1672947675,"issuing_ca":"-----BEGIN CERTIFICATE-----\nMIIEHzCCAwegAwIB                                 AgIUMjpzvgqs7MP8+HmsvUc5Hn4zj7IwDQYJKoZIhvcNAQEL\nBQAwgZExCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQH\nEw1TYW4gRn                                 JhbmNpc2NvMRYwFAYDVQQKEw1FeGFtcGxlLCBJbmMuMR0wGwYDVQQL\nExRSZWJyYWluIFZhdWx0IENvdXJzZTEeMBwGA1UEAxMVUmVicmFpbiBWYXVsdCBS\nb290                                 IENBMB4XDTIyMTAxNjEyMDMwMFoXDTQyMTAxNjEyMDMwMFowgZgxCzAJBgNV\nBAYTAlJVMRYwFAYDVQQIEw1Nb3Njb3cgcmVnaW9uMQ8wDQYDVQQHEwZNb3Njb3cx                                 \nFzAVBgNVBAoTDkV4YW1wbGUgLCBJbmMuMScwJQYDVQQLEx5SZWJyYWluIENvdXJz\nZSBJbnRlcm1lZGlhdGUgQ0ExHjAcBgNVBAMTFVZhdWx0IEludGVybWVkaW                                 F0ZSBD\nQTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAJ4XtGtXNiziEzMdQNGF\n8GfSnp/+g0U7JTvzBCcBbLDSVhdjKkuPU3V5jYMsW7IY4jayqB7c                                 Ps0OZEGrA97O\neVshezFaXwOgrEuGUQXluV9VONuCzJmrjMQAjydessV9Z/ko/0UQ4PKF/j4XpmBZ\nMdfVGry7cP6oyrWDGkXa3kucBBb/sckf5QSAkjMp68in7J                                 WJeA5BGqvooKW69bzy\nINRne+btbz0d8R88etn2UXs2RVm91v/KQ21R/fbDSAS+P6+xBhblmv6vCxkmb5QX\ntpIZQ/op6dgtG6K5yL1iDRNJHn4Z7R9zdc3n1UbD                                 5y4JsRRJ+VWDI9fieo3vZH6D\nPCkCAwEAAaNmMGQwDgYDVR0PAQH/BAQDAgEGMBIGA1UdEwEB/wQIMAYBAf8CAQAw\nHQYDVR0OBBYEFEg3T7qbQeuUPjNUjGQtlu                                 IXLYNuMB8GA1UdIwQYMBaAFK2ZXrA5\n0maoozwNrcQqtFkkEgwiMA0GCSqGSIb3DQEBCwUAA4IBAQAcbLF9o5xggGll8b8u\nirlwbG73h6IrQuCqpzB8OtygLPQa                                 ozWTKfnLHfdnFXZAXvcGh9gDiM+IuJ5/4jXn\nfBV/GxkjhGSH2/1fymsADJ9DrZFMcdwqsWWMW6tI87+kDb4Z2yimzD20l+XpDOQu\n8sCz/rRZhNB4S75W2HKtPV                                 cA2PdH6ph39HyzzJ5CLsWGo4vv8RXMXz/AgzHDHkVF\nzej7JSQF5E04GaUbypeEsvsYVs3yEiFTOqm9Tekv4keNWll3sweShOTI891Njmvw\n04yQu2WsFeoni/Tb                                 +dZgOtb/zuxuqqpemgrHDMmSFYGZmVYYtviLWP2JqSb0tPso\nygHx\n-----END CERTIFICATE-----","private_key":"-----BEGIN RSA PRIVATE KEY--                                 ---\nMIIEpAIBAAKCAQEA0xZjTD6h8wUEBz1AcKBwNcz6WdzE3VthWzK5WSMIsMEdF8db\ngIbUIiuft1FSGPvgGPxEGo91GkuK79McEGAUkQ6gYbh6YsOWJlpzyCD                                 AsZhOJReP\nfo3/DZ2YPC3qBill+wE7TBTHNZsSvzboJGkrrCBcJmekQFKp1pMM1ROR7nhd5Ao1\nHJqR72afCLysIJZEyfhA9tXXj4GQ8UA/4gqT2Vpi/A29d+Txm                                 FBbkOI83nVOjat9\nvo96mM6xARLyZr0LCKjN4R4f/GOfUHCs2G7Ym7cdJ1ZqPWL2u5na3CkmPhrj910o\nsOGK1RY3iBSgtsSmYbgx8Bxt/K5tMERkCkFRiwIDAQA                                 BAoIBAQDOiU1HQNE8819p\npejzSkgAnDsoyfZlkA/GJ+9q4/iQ2aMZrRo+u628cWqo94yYnXo7eDk6s7skq12a\nIrmG3DvDYshSVSqKkEzN4hr/aeyg2CE98buZX                                 F5+eACIgXRF6yO5YQ8f9gSk0sKZ\nDaQ+XBk7Jb6EZUw1E6zSIreflLJo3N0o4CYsOyJgGLqAlqzvpkG92su4VX1IahhK\nwfemU/M6R2kD9YH046S+9Gk6pxHqtb7                                 iIYHYlwzXbyXWzX45Kr3zNGd89VjGfvCQ\nevDlklFdtqry8gINFdPbOQhjhDEfNWu3tEKSbLOea3qzd4SUZMiGT3iYic3P47hQ\nbZxa6ySBAoGBANkYWT6X5+SY7                                 mqcPjDDFLabR+AebfN+hJfkepSomr1X4GejTZMS\nqQ5G6S698EhYz6ZRw0+SkPat42droB4veIuA5cILe+if4azaGpaR7MDEE0T1v5Ix\nJzGcFtKRuwSxQvNfY93                                 lfy10RA9dkId/BRt37RP4smYtXIbW0bdre6B7AoGBAPjq\nbS2YGFuQM4cGyEz/b4E7JIfs007seCvcK1WQrWyjUgOa7q0DRflQHvJA07k0s4cl\niGHOswoxCg4C6                                 0ujS15OkTpc53uteMz0B910mRfDKuRnUkC6hTEmS46zO5o2+9Oa\n9cktydohy23waWXmy6zqPZZv4Axm9LxJS6ikTK4xAoGBAKdoIEGlUBu2Vnt9enON\nq2ZY/ab                                 0sCLJGCQs+t2x2olRv2kLw6E7DYRF6EC0FRsk6RM/D5ZH1mNymd5BXxqH\nzrP8tK/avTUYPSVWlpQveNr5GEbgHlb0cl3OGMdNu2KV8qPLli4hb920P1t98hqa\nN                                 20EIJx69c7XAfe0pcmEJ7QLAoGAJvrXJBe4YMZhO1j1jxFFTfCMFPkiUi631u6A\nnsKsVeHxmvztOYzUrWk9n2RFg7BcGOLoy6BJ62Oolm8gl9S3ncoh9gjMe1K8I                                 yRo\nAucafl0i32fKurY622qK1Ir+33SS1R1kNiAEhzNZnxrR9pJA/RAlmuRkKq0I0F+O\nCJfKJlECgYBS6LXjPECJMKeqp8H+kEnAGLrJ1h7Til0HP+mq3vWv2fk                                 7cxMZQ5Ox\nlNHrYTv39Z33H4XkqGbU1tvFj4Zj8kzS/XY50wXz7XZRaPppXAimfbeWL1JzmqrG\nYJN3xujW7REJ67RakQcgYtbLxcC04wSqMUqKCPDUfudy4+ecp                                 TcweQ==\n-----END RSA PRIVATE KEY-----","private_key_type":"rsa","serial_number":"64:f0:7d:1e:73:e0:df:53:8c:3f:e4:f5:01:3f:cc                                 :aa:4e:8b:18:b7"},"wrap_info":null,"warnings":null,"auth":null}