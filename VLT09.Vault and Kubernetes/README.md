# VLT 09: Vault and Kubernetes

*Описание:*
В текущем задании мы:

настроили Kubernetes AuthMethod
установили Vault Injector
настроили cert-manager для выпуска сертификатов из Vault PKI
настроили external secret operator для автоматического создания Secret
Вы можете посмотреть запись по ссылке. Также для Вашего удобства прикладываем презентацию к вебинару.

## Задание:
Вам будет предоставлен доступ к VPS, на которой уже запущен Kubernetes. Вам необходимо выполнить все задания в этом кластере. Все команды для работы с Kuberentes указаны в тексте задания.

Перед началом работы, обязательно выполните команды:
```
minikube update-context
minikube tunnel -c &
```

Helm репозиторий HashiCorp недоступен в РФ, поэтому в каталоге /home/usr/vault_helm лежит чарт vault-0.22.1.tgz. Его нужно будет установить в ходе задания.

Для работы с Kubernetes предлагаем использовать kubectl или k9s. По желанию Вы можете загрузить свои утилиты для работы с Kubernetes на виртуальную машину.

### Задание

####  1. Запустите minikube командой minikube start. Разверните инстанс Vault server и Vault Injector в кластере kubernetes. Проведите инициализацию кластера с тремя ключами, любые два из которых распечатывают Vault. Сохраните root token в файл /home/user/root_token. Используйте namespace vault и следующую конфигурацию Helm-chart:
```
global:
  enabled: true
  tlsDisable: true

injector:
  enabled: "true"
  replicas: 1
  port: 8080

server:
  standalone:
    enabled: true

ui:
  enabled: true
  serviceType: "NodePort"
  externalPort: 8200
  targetPort: 8200

ingress:
  enabled: false
```

Сохраните вышеуказанный код в файл values.yaml в каталоге /home/user/vault_helm/values.yaml и выполните команду

`kubectl create ns vault 2>/dev/null && helm install -f /home/user/vault_helm/values.yaml -n vault vault /home/user/vault_helm/vault-0.22.1.tgz`

URL сервиса vault-ui получите с помощью команды minikube service -n vault vault-ui --url, не забудьте указать его в VAULT_ADDR.

#### 2. Настройте AuthMethod Kubernetes. Укажите API-адрес Вашего Kubernetes кластера и сертификат CA.
Получите IP kubernetes с помощью команды kubectl describe svc kubernetes | grep IP:.

В качестве kubernetes_host укажите https://[полученный_ip].

В качестве token_reviewer_jwt используйте kubectl create token vault -n vault.

Создайте роль autocheck-role, которая будет разрешать авторизацию в Vault с ServiceAccount autocheck из namespace default c политикой db-readonly-policy. TTL выпускаемого токена установите в 10 минут.

#### 3. Включите движок kv-v2 (path=kv-v2). Создайте секрет db с полями login=user и password=pass.

#### 4. Создайте политику db-read-only-policy, которая разрешает чтение ранее созданного секрета db.

#### 5. Выполните деплой следующего манифеста (например, сохраните в /home/user/autocheck.yaml и выполните команду kubectl apply -f /home/user/autocheck.yaml):
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: autocheck
---
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: autocheck
  annotations:
    kubernetes.io/service-account.name: "autocheck"
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
data:
  nginx.conf: |
    user root;
    worker_processes  1;
    error_log  /var/log/nginx/error.log;
    events {
      worker_connections  1024;
    }
    http {
      server {
          listen       80;
          server_name  _;
          location / {
              root /vault/secrets;
          }
      }
    }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-with-vault-secrets
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/agent-inject-secret-config.yml: 'kv-v2/data/db'
        vault.hashicorp.com/template-static-secret-render-interval: "5s"
        vault.hashicorp.com/agent-inject-template-config.yml: |
          {{- with secret "kv-v2/data/db" -}}
          db_login: {{ .Data.data.login }}
          db_password: {{ .Data.data.password }}
          {{- end }}
        vault.hashicorp.com/role: 'autocheck-role'
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /etc/nginx
          readOnly: true
          name: nginx-conf
      volumes:
      - name: nginx-conf
        configMap:
          name: nginx-conf
          items:
            - key: nginx.conf
              path: nginx.conf
      serviceAccountName: autocheck
      automountServiceAccountToken: true
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-autocheck
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
Дождитесь, пока POD перейдет в состояние Running. Затем проверьте, что файл config.yml можно прочитать командой curl $(minikube service nginx-autocheck --url)/config.yml.

**Дополнительно**: Можно изменить значения полей в секрете `db` и выполнить команду повторно, убедившись, что значения меняются.

#### 6. Включите движок секретов PKI по стандартному пути k8s-pki/

#### 7. Импортируйте следующий CA сертификат в k8s-pki/
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
```
#### 8. Создайте PKI-роль k8s-certs со следующими параметрами:
max_ttl=24h allowed_domains=rebrain.local allow_bare_domains=true

#### 9. Создайте политику k8s-certs со следующими правилами:

path "k8s-pki*"                        { capabilities = ["read", "list"] }
path "k8s-pki/sign/k8s-certs"    { capabilities = ["create", "update"] }
path "k8s-pki/issue/k8s-certs"   { capabilities = ["create"] }
и периодический токен c периодом 24h (командой vault token create -period=24h -policy=k8s-certs), запомните или запишите его, он понадобится далее.

#### 10. Задеплойте cert-manager в кластер командой:
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.10.0/cert-manager.yaml

#### 11. Примените следующие манифесты для конфигурации Vault Issuer:
```
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: cert-manager-vault-token
  namespace: default
data:
  token: **ранее созданный токен в формате base64**
---
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: vault-issuer
  namespace: default
spec:
  vault:
    path: k8s-pki/sign/k8s-certs
    server: http://vault.vault:8200
    auth:
      tokenSecretRef:
          name: cert-manager-vault-token
          key: token
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: rebrain-local
  namespace: default
spec:
  secretName: rebrain-local-tls
  duration: 24h
  renewBefore: 23h
  commonName: rebrain.local
  dnsNames:
    - rebrain.local
  issuerRef:
    name: vault-issuer
    kind: Issuer
```    
Сохраните вышеуказанный код в файл, например, autocheck.yaml, и выполните команду kubectl apply -f autocheck.yaml