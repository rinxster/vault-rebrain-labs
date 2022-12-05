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


`export VAULT_SKIP_VERIFY=true && export VAULT_ADDR=https://127.0.0.1:8200 && sudo systemctl restart vault && vault operator init -key-shares=2 -key-threshold=2 >> /home/user/vault_keys`

`cat /home/user/vault_keys`

`vault operator unseal`

`export VAULT_TOKEN="hvs.e2k53DiMA03Gen8kqYme24cO" && echo $VAULT_TOKEN > /home/user/root_token`



2.Включите PKI SecretEngine по пути rebrain-pki/ с максимальным сроком жизни сертификата 1 год (8760 часов)

`vault secrets enable -path=rebrain-pki -max-lease-ttl=8760h pki`

3.Импортируйте ранее созданный bundle.pem в секрет rebrain-pki/config/ca.



// `vault write rebrain-pki/import/bundle pem_bundle=@/opt/certs/bundle.pem`

vault write rebrain-pki/config/urls issuing_certificates="http://127.0.0.1:8200/v1/pki/ca" crl_distribution_point="http://127.0.0.1:8200/v1/pki/crl"

`vault write rebrain-pki/config/ca pem_bundle=@/opt/certs/bundle.pem`

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

`echo hvs.CAESIOxJPrIFO167tlxHvJJJFa712hDqBm8kYOW8oAP75AiXGh4KHGh2cy5xaVplRVlCVWhzbUE4bnk5a2tlSVF2Sk0 > /home/user/cert_issuer_token`

7. Выпустите сертификат с common_name="rebrain-vault.local" alt_names="random.rebrain-vault.local". Сохраните его в файл /home/user/cert.pem

выпускаем сертификат командой curl:

```
 curl \
    --header "X-Vault-Token: hvs.CAESIOxJPrIFO167tlxHvJJJFa712hDqBm8kYOW8oAP75AiXGh4KHGh2cy5xaVplRVlCVWhzbUE4bnk5a2tlSVF2Sk0" \
    --request POST \
    --data  '{ "common_name": "rebrain-vault.local", "alt_names": "random.rebrain-vault.local" }' \
    https://127.0.0.1:8200/v1/rebrain-pki/issue/local-certs

```
сохраняем содержимое в файл:

`CERT > /home/user/cert.pem` 


8.Запустите dev-server Vault на 9200/TCP порту в бэкграунде любым удобным способом (screen, nohup и т.д.). Используйте команду vault server -dev -dev-root-token-id=1 -dev-listen-address=0.0.0.0:9200. Она запустит dev-сервер с root-токеном "1", который необходимо использовать для дальнейшей настройки.

в отдельном окне открываем доп сессию и в ней делаем команде
`vault server -dev -dev-root-token-id=1 -dev-listen-address=0.0.0.0:9200`

далее в основном окне переключаемся на ДЕВ консоль

`export VAULT_ADDR="http://127.0.0.1:9200" && export VAULT_TOKEN=1`

9. В dev-сервере смонтируйте SecretEngine Transit (не забудьте поменять env VAULT_ADDR и VAULT_TOKEN) как transit-autounseal и создайте ключ vault-autounseal.

монтируем транзит:
`vault secrets enable -path=transit-autounseal transit`

создаём ключ:
`vault write -f transit-autounseal/keys/vault-autounseal`

10. В dev-сервере создайте политику transit-autounseal со следующим содержимым:
```
path "transit-autounseal/encrypt/vault-autounseal" {
   capabilities = [ "update" ]
}
path "transit-autounseal/decrypt/vault-autounseal" {
   capabilities = [ "update" ]
}
```

```
echo 'path "transit-autounseal/encrypt/vault-autounseal" {
   capabilities = [ "update" ]
}
path "transit-autounseal/decrypt/vault-autounseal" {
   capabilities = [ "update" ]
}' | vault policy write transit-autounseal -
```

проверяем политику и убеждаемся, что она создалась корректно.

`vault policy read transit-autounseal`

11. Создайте периодический токен для этой политики с периодом 24 часа.

`vault token create -policy=transit-autounseal -period=24h`

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
открываем конфигурационный файл:

`sudo nano /etc/vault.d/vault.hcl`

и добавляем туда кусок, где токен - ваш токен сгенерированный в шаге 11 выше.

```
seal "transit" {
  address            = "http://127.0.0.1:9200"
  token              = "hvs.CAESIAs37fvdlVeZqu2mp7G2KUl03ytNL9uw3lplVwNvJPR5Gh4KHGh2cy5lQUFyMTJwRjlxUWxGMWJGYlRheXU3bUw"
  key_name           = "vault-autounseal"
  mount_path         = "transit-autounseal"
  tls_skip_verify    = "true"
}
```
переключаемся обратно на основной экземпляр Vault

`export VAULT_ADDR="https://127.0.0.1:8200" && export VAULT_TOKEN=hvs.e2k53DiMA03Gen8kqYme24cO`

перезапускаем волт, чтобы применилось измнение конфиг файла

`systemctl  restart vault`

13. Перезапустите vault и выполните команду vault operator unseal -migrate.

`vault status`

`vault operator unseal -migrate`

распечатываем волт вбиваем 2 ключа для распаковки основного экземлпяра и смотрим статус.

`vault status`

перезапускаем волт и снова смотрим статус. он должен быть автоматически распечатан.

`systemctl restart vault`

`vault status`

ВАЖНО! Перед отправкой задания на проверку перезагрузите основной инстанс vault-сервера (`systemctl restart vault`). Убедитесь, что unseal происходит автоматически!

