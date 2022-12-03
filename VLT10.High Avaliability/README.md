**VLT 10: Отказоустойчивость**

Описание:
В текущем задании мы изучаем построение отказоустойчивого кластера Vault с использованием Raft Storage Backend.

Вы можете посмотреть запись по ссылке. Также для Вашего удобства прикладываем презентацию к вебинару.

Задание:

1. Создайте каталоги
```
/vault
/vault/cert
/vault/config
/vault/storage/node1
/vault/storage/node2
/vault/storage/node3
```
2. В каталог /vault/cert поместите сертификаты:
ca.pem:

```
-----BEGIN CERTIFICATE-----
MIID9DCCAtygAwIBAgIUdKUqKicUHh0Nrduu7tCZ5fUPjCkwDQYJKoZIhvcNAQEL
BQAwgZExCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQH
Ew1TYW4gRnJhbmNpc2NvMRYwFAYDVQQKEw1FeGFtcGxlLCBJbmMuMR0wGwYDVQQL
ExRSZWJyYWluIFZhdWx0IENvdXJzZTEeMBwGA1UEAxMVUmVicmFpbiBWYXVsdCBS
b290IENBMB4XDTIyMTAxNjEwMTAwMFoXDTI3MTAxNTEwMTAwMFowgZExCzAJBgNV
BAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1TYW4gRnJhbmNp
c2NvMRYwFAYDVQQKEw1FeGFtcGxlLCBJbmMuMR0wGwYDVQQLExRSZWJyYWluIFZh
dWx0IENvdXJzZTEeMBwGA1UEAxMVUmVicmFpbiBWYXVsdCBSb290IENBMIIBIjAN
BgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA8tpGdWz64XnYwB8BzahsT0xx4osi
tmamYqemSzGmPlgRH4Vd3JXFE1yitxkrYvSvWPsGo6R9bbp458PZ8hcfeDW/Z1BP
TkKW5ZQqKXx7Asnd/l4WPpzmmGIpZ8NncNub2Xa8IV1fvbWlQse3y7AI1eUmUMHS
Grnk+6DwTz4gdAWcs4nkiAz350NP8FgoVpxgnSa3Io92ZL0eYBM2wn9ge6qAQlzh
hgExW/mn8Vfk9UEv8JAM63/NpsnWlHrwjXDsvY6vEXZRF1D4i6i1rsAQEuaOzrqJ
bXcadEvBr54WP76w8KVTkdfAvYO6o64e/mwVsWDoueCv84f40YcjWIXziwIDAQAB
o0IwQDAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQU
rZlesDnSZqijPA2txCq0WSQSDCIwDQYJKoZIhvcNAQELBQADggEBAI2gprWzi29y
t/R0TFfRcofdgmQZbihgtUGPanfgvJeyHb8DTmW07tUgsxyQnhPOaMYtclhbiiO0
2+CaE0l+UV/XLA549qfZ6hpNwASDUdECDidhmQiieiJUvK4tpc5BCmn4QFfD5GKj
LycKixXwmZAY7j4OjI2YNhMto6Px3RKQ6pJGkuaTxUTyqnIjF3WT4GYHu2/6KZx+
W9lv7apEcssDq3wielaCtd42XpjnXP//+DlA/c7Czs9rleVyay4i3M23/9ko7QT2
UIn64Ne5/tX1QtpDSqBIZQqaO3bGH8f1y29BjMIyV7AnhPlu9KGCR4GS2FZeo7T6
PmbOK+nqyHY=
-----END CERTIFICATE-----
```
node-cert.pem:
```
-----BEGIN CERTIFICATE-----
MIIEZjCCA06gAwIBAgIUazj7Q3SRbxZM2GNu4BSMp20iUbEwDQYJKoZIhvcNAQEL
BQAwgZExCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQH
Ew1TYW4gRnJhbmNpc2NvMRYwFAYDVQQKEw1FeGFtcGxlLCBJbmMuMR0wGwYDVQQL
ExRSZWJyYWluIFZhdWx0IENvdXJzZTEeMBwGA1UEAxMVUmVicmFpbiBWYXVsdCBS
b290IENBMB4XDTIyMTExMzEyMjgwMFoXDTQyMTExMzEyMjgwMFowgYExCzAJBgNV
BAYTAlJVMRYwFAYDVQQIEw1Nb3Njb3cgcmVnaW9uMQ8wDQYDVQQHEwZNb3Njb3cx
FzAVBgNVBAoTDkV4YW1wbGUgLCBJbmMuMRowGAYDVQQLExFSZWJyYWluIENvdXJz
ZSBDQTEUMBIGA1UEAxMLRVRDRCBTZXJ2ZXIwggEiMA0GCSqGSIb3DQEBAQUAA4IB
DwAwggEKAoIBAQCsPskbBXsS/xOlojuub1YTrrz8GALAy8IftUobPp2WUQ1IdrW3
JeEI+J2J5Js5DWxd4pDqgdqNuPlk8QnZcnZuS50ySsDJeUoOsYFYcbt8ZpWS9WTB
zUBiGthPeSHVa/XwDuJx9Uc6xx1FkVT2/2pQPIkKNwgaW0ninM8RgeLIPdHpGjX4
vHXZ3TWLsxS0RNGov4Nxwc76WUA41Tys/Mz9WpaUSizjzlI00JJnqmTkttIUL2ae
KsVi9izUuWvRWxk+Q0sYyteFxUUdpdRV6jFqD1bMi/QPL/k+pZuAjk9KQ80EzevL
pLyf0QEjBPWK50Vu7iNedbtMz4QTjoCi9qKLAgMBAAGjgcMwgcAwDgYDVR0PAQH/
BAQDAgWgMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjAMBgNVHRMBAf8E
AjAAMB0GA1UdDgQWBBQgRVMXdj2qgFtdkx8mSZokipPRBzAfBgNVHSMEGDAWgBSt
mV6wOdJmqKM8Da3EKrRZJBIMIjBBBgNVHREEOjA4gglsb2NhbGhvc3SCC3ZhdWx0
LW5vZGUxggt2YXVsdC1ub2RlMoILdmF1bHQtbm9kZTOHBH8AAAEwDQYJKoZIhvcN
AQELBQADggEBAL35RypDe7/U2eWyjzAJI/Fwi2hIWfD5H0S+EGmtl8u5dJ89Vnbd
IPtMd8/MS4vglY5+dOIszkUrO4vFdjfOWtQA8ifhNgVKWvSqsRilI2g38JOX4Ok7
ZgR7dgi2VkWUVbhdldakMemoWmK2HTR0jpxCDrY5Ltch8xQI/wgjgvUsWgD/1/Ho
wd4WfjV1E0WVrKSph3QP2t58LjP2VLbh4J96EnQo8gdIh1sT+MvNflyfsyugYDfX
Yr8U044iSrQ3JMxbBDB/wTEEzybIXlSVfgiuBvL7xG48iBQOYm5HyidCMSAWCRTS
/Rqs9//go/56El3mPPEhSqMXrwAsUjDnlZE=
-----END CERTIFICATE-----
```
node-key.pem:
```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEArD7JGwV7Ev8TpaI7rm9WE668/BgCwMvCH7VKGz6dllENSHa1
tyXhCPidieSbOQ1sXeKQ6oHajbj5ZPEJ2XJ2bkudMkrAyXlKDrGBWHG7fGaVkvVk
wc1AYhrYT3kh1Wv18A7icfVHOscdRZFU9v9qUDyJCjcIGltJ4pzPEYHiyD3R6Ro1
+Lx12d01i7MUtETRqL+DccHO+llAONU8rPzM/VqWlEos485SNNCSZ6pk5LbSFC9m
nirFYvYs1Llr0VsZPkNLGMrXhcVFHaXUVeoxag9WzIv0Dy/5PqWbgI5PSkPNBM3r
y6S8n9EBIwT1iudFbu4jXnW7TM+EE46AovaiiwIDAQABAoIBAQCi9wxi6n6VbIz0
K1h4I5K3MJ5RjY4dRys1wNqKiGWk8K62nsoyrD4LtN2ot4g9JHwhH9moZo+XgylC
3eNJvshadmQWTy+z73OoDz2npoOSoaRm1JIt4rpFl8yM9LiUKn8YT5zj4QMxk24Y
gfZ3cxTtMTkfVw3tke2H4IDxuYgNlxJ3tKLNdYfiquVWnsk1Wrf6NOWrVK+IaOdu
wyVJR1UT706+N63KFQVfuM2Ts05SUEcB4FBjihbMqM2TWZVJ+DQW3TobTcB1wjXK
oHygMxkgJjaOb5v8Llq1aXJtDKGTBElbbrf9V+38Emep7YQ+2HV+B8Tu9t4/g7iK
sc3Y9nwBAoGBANWojDXV4Wy84CHXkxxaaurK4Ct+Nt0vpUveaiq6pl1qiU3NZX0l
T+A9QGr6uTuvZOStlKlWRXHrGE6u1194/X4lj+6yGN1SRilpZK4Z+pBs8d9HBh47
1OGEJJlW3HA889C07x/Ub/WSrUJz6awTcmvQtpu7S6umLbJj9PQvOCkBAoGBAM5h
PZsO/tmd20Nw3FXnvG808y+wHuciqnFaYgDgimnx8IsiX7Hok+7OOfvsmICrugvH
VTZMyDneHBoP451Nploj8ykrv0fhGDEHeFXzNQrPBpl/XtWeuxrAbHtHRJmvJax/
mNO6KzXMOkev60/B1cGFdNgkhgAzMhb8yYu/QV+LAoGBAMA+H6pw/5wvdhv9NEjW
sk0AriN0NTlfnYNeZHh96SM0sMZogWDRKcXCVyvq3LBvaIC6DoEvNt0Bg6WIfBFT
dAMFGTTU2rqJRMgOJKDijylUXW1hIoghnbIsjCHMnhv/PAIWSvKA2xxDFdItKZvD
A7ku2p/VLokLxSI1/jmYIxgBAoGAUDTeemqjhPOiiV1NZF2BkD6l3Hy4JeAFGbSk
re6WHIKYl5ouUrgu9fpT8qKKykbzMSyw4z+H+WVmyoIuVa4d3p5mHDQSTN8gRb2/
eLfif2biC7nCo4bi9IygHHEgKhI1tAqK3I5XyLqsU7v82axdZK53MKFRKra62tA3
jAYyY+0CgYAb0FWL6I9egOLrVfvQ/JzW8eaB2t9VyzZI577tPpN4/rod+g4r7zKJ
eQrPuLC0mLb76plezBzVLBkyVDXH5w5TEhTEq4pWmkVd6cr+oYNdTSSKIM85dxxk
Hdmp7MA1TnlktiuxVvRblZkUWkaLHRvK8AIyoGluHaeHTmTlVzb9tA==
-----END RSA PRIVATE KEY-----
```
3. Создайте конфигурацию для каждой ноды Vault HA raft storage.
- Для адресов нод в кластере используйте следующие dns-имена: vault-node1, vault-node2, vault-node3.
- storage должен находиться в каталоге /vault/storage
- Используйте ранее созданные сертификаты для настройки TLS для нод в кластере
- Файл конфигурации разместите по адресу /vault/config/vault-node[номер-ноды].hcl
Пример конфигурации для первой ноды:
```
ui = true
disable_mlock = true

listener "tcp" {
  address       = "0.0.0.0:8200"
  tls_cert_file = "/vault/cert/node-cert.pem"
  tls_key_file  = "/vault/cert/node-key.pem"
}

cluster_addr = "https://vault-node1:8201"
api_addr    = "https://vault-node1:8200"
cluster_name = "vault"

storage "raft" {
  path         = "/vault/storage"
  node_id      = "vault-node1"

  retry_join {
    leader_api_addr = "https://vault-node1:8200"
    leader_ca_cert_file = "/vault/cert/ca.pem"
    leader_client_cert_file = "/vault/cert/node-cert.pem"
    leader_client_key_file = "/vault/cert/node-key.pem"
  }

  retry_join {
    leader_api_addr = "https://vault-node2:8200"
    leader_ca_cert_file = "/vault/cert/ca.pem"
    leader_client_cert_file = "/vault/cert/node-cert.pem"
    leader_client_key_file = "/vault/cert/node-key.pem"
  }

  retry_join {
    leader_api_addr = "https://vault-node3:8200"
    leader_ca_cert_file = "/vault/cert/ca.pem"
    leader_client_cert_file = "/vault/cert/node-cert.pem"
    leader_client_key_file = "/vault/cert/node-key.pem"
  }
}
```
4. Создайте docker-compose файл по пути /home/user/docker-compose.yml.
```
version: '3'
services:
  vault-node1:
    image: hashicorp/vault:1.12.1
    restart: always
    command: sh -c "chown -R 0:0 /vault && vault server -config=/vault/config/config.hcl"
    privileged: true
    cap_add:
      - IPC_LOCK
    volumes:
      - /vault/storage/node1:/vault/storage
      - /vault/cert:/vault/cert
      - /vault/config/vault-node1.hcl:/vault/config/config.hcl
    ports:
      - "8300:8200/tcp"
    container_name: vault-node1

  vault-node2:
    image: hashicorp/vault:1.12.1
    restart: always
    command: sh -c "chown -R 0:0 /vault && vault server -config=/vault/config/config.hcl"
    privileged: true
    cap_add:
      - IPC_LOCK
    volumes:
      - /vault/storage/node2:/vault/storage
      - /vault/cert:/vault/cert
      - /vault/config/vault-node2.hcl:/vault/config/config.hcl
    ports:
      - "8400:8200/tcp"
    container_name: vault-node2

  vault-node3:
    image: hashicorp/vault:1.12.1
    restart: always
    command: sh -c "chown -R 0:0 /vault && vault server -config=/vault/config/config.hcl"
    privileged: true
    cap_add:
      - IPC_LOCK
    volumes:
      - /vault/storage/node3:/vault/storage
      - /vault/cert:/vault/cert
      - /vault/config/vault-node3.hcl:/vault/config/config.hcl
    ports:
      - "8500:8200/tcp"
    container_name: vault-node3
```
5. Запустите кластер командой docker-compose -f /home/user/docker-compose.yml.

6. Произведите инициализацию первой ноды с любым количеством ключей и unseal кластера. Cохраните рут токен в файл /home/user/root_token. Для нод используется следующий мапинг портом на localhost:
```
https://127.0.0.1:8300 - первая нода
https://127.0.0.1:8400 - вторая нода
https://127.0.0.1:8500 - третья нода
```
7. Выполните unseal второй и третьей ноды.

8. С помощью команды vault operator raft list-peers убедитесь, что кластер собран и выбран лидер.