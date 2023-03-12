# VLT: Финальное задание

## Описание:
Финальное задание для построения базовой высокодоступной инфраструктуры Vault в кластере Kubernetes.

Будет использован кастомный Helm Chart (из-за некоторых ограничений стандартного), лежащий в папке /home/user/vault-custom.

Необходимо использовать prometheus-operator для сбора метрик (указать в конфигурации vault).

## Задание:
### 1. Запустите minikube. Создайте необходимые namespaces
```
sudo sysctl -w fs.inotify.max_user_instances=8192
minikube start --driver=docker --nodes 3
kubectl create ns vault
kubectl create ns vault-a
kubectl create ns monitoring
kubectl create ns cert-manager
```
### 2. Добавьте все необходимые репозитории для Helm
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add jetstack https://charts.jetstack.io
helm repo update
```
### 3. Разверните отказоустойчивый кластер с Raft backend. При правильной настройке Helm Chart не запустится без инизиализации прометея, поэтому для начала раскатаем его
```
helm install -n monitoring prometheus prometheus-community/kube-prometheus-stack
```
### 4. Скачайте custom для vault
```
wget https://storage.yandexcloud.net/files.rebrainme.com/workshops/hashicorp-vault/vault-custom.tgz
tar -xvzf vault-custom.tgz
mv vault vault-custom
```
### 5. Измените StorageType для работы raft
```
wget https://storage.yandexcloud.net/files.rebrainme.com/workshops/hashicorp-vault/kubevirt-hostpath-provisioner.yaml

minikube addons disable storage-provisioner
kubectl delete storageclass standard
kubectl apply -f kubevirt-hostpath-provisioner.yaml
```
### 6. Ниже представлен Helm чарт для развертывания 3 инстансов Vault на трех нодах в кластере Kubernetes. Вам необходимо по пути server.ha.config дописать конфиг Vault, удовлетворяющий следующим условиям:
- UI активирован
- listener tcp:
  - tls отключен
  - прослушивает порт 8200 на всех интерфейсах (IPv4 и IPv6)
  - назначить адрес кластера на порту 8201
- в telemetry включить доступ к метрикам без авторизации
- Настроить raft
- по пути /vault/data
- Автопилот
- cleanup_dead_servers = "true"
- last_contact_threshold = "200ms"
- last_contact_failure_threshold = "10m"
- max_trailing_logs = 250000
- min_quorum = 5
- server_stabilization_time = "10s"
- Телеметрия
  - Prometheus хранит данные 24 часа
- Отключить hostname
- Зарегистрировать сервис kubernetes

vault-helm-config.yaml
```
cat > vault-helm-config.yaml << EOF

global:
  enabled: true
  tlsDisable: true
  serverTelemetry:
    prometheusOperator: true

injector:
  enabled: true
  image:
    repository: "hashicorp/vault-k8s"
    tag: "latest"

  resources:
    requests:
      memory: 256Mi
      cpu: 250m
    limits:
      memory: 256Mi
      cpu: 250m

  metrics:
    enabled: true

server:
  resources:
    requests:
      memory: 256Mi
      cpu: 500m
    limits:
      memory: 256Mi
      cpu: 500m

  readinessProbe:
    enabled: true
    path: "/v1/sys/health?standbyok=true&sealedcode=204&uninitcode=204"
  livenessProbe:
    enabled: false
    path: "/v1/sys/health?standbyok=true"
    initialDelaySeconds: 60

  auditStorage:
    enabled: true
    size: 1Gi
    storageClass: standard

  dataStorage:
    enabled: true
    storageClass: standard

  standalone:
    enabled: false

  ha:
    enabled: true
    replicas: 3
    raft:
      enabled: true
      setNodeId: true

      config: |
        ui = true
        listener "tcp" {
          address = "[::]:8200"
          cluster_address = "[::]:8201"
          telemetry {
            unauthenticated_metrics_access = "true"
          }
          tls_disable = 1
        }
        storage "raft" {
          path = "/vault/data"
          autopilot {
            cleanup_dead_servers = "true"
            last_contact_threshold = "200ms"
            last_contact_failure_threshold = "10m"
            max_trailing_logs = 250000
            min_quorum = 5
            server_stabilization_time = "10s"
          }
        }
        telemetry {
          prometheus_retention_time = "24h"
          disable_hostname = true
        }
        service_registration "kubernetes" {}

ui:
  enabled: true
  serviceType: "NodePort"
  externalPort: 8200
  targetPort: 8200

ingress:
  enabled: false

EOF
```

### 7. Разверните кластер
```
helm install -n vault vault ./vault-custom -f vault-helm-config.yaml
```
### 8. Инициализируйте vault на поде vault-0 с 3 ключами, любые два из которых распечатывают Vault. Сохраните root-token по пути /home/user/root_token

Подключите raft для vault-1 и vault-2. Url для vault-0 в кластере http://vault-0.vault-internal:8200

```
a. открываем консоль k9s
b. выбираем все неймспейсы(клавиша 0)
c. выбираем под "vault-0"
d. нажимаем "s" - в результате он в него заходит внутрь консоли
e. внутри пода выполняем команду: export VAULT_SKIP_VERIFY=true && vault operator init -key-shares=1 -key-threshold=1
f. export VAULT_TOKEN=<ваш токен>
g. vault operator unseal <ваш ключ unseal>
h. проверяем результат: vault status
i. выходим из пода и повторяем входим в под "vault-1"(пункты a-d)
j. выполняем команду: export VAULT_SKIP_VERIFY=true && export VAULT_TOKEN=<ваш токен> && vault operator raft join "http://vault-0.vault-internal:8200"
k. vault operator unseal <ваш ключ unseal>
l. проверяем результат командой: vault operator raft list-peers
m. повторяем пункты i-l для "vault-2"
```

```
пример вывода команды по результатам
/ $ vault operator raft list-peers
Node       Address                        State       Voter
----       -------                        -----       -----
vault-0    vault-0.vault-internal:8201    leader      true
vault-1    vault-1.vault-internal:8201    follower    true
vault-2    vault-2.vault-internal:8201    follower    true

```
```
minikube service -n vault vault-ui --url 

export VAULT_ADDR=http://192.168.49.2:31687

export VAULT_SKIP_VERIFY=true && export VAULT_TOKEN=hvs.6PE71MtGlJigeXTDNrHdc0fj

echo $VAULT_TOKEN >> /home/user/root_token

cat /home/user/root_token

```
## Настройка autounseal
https://developer.hashicorp.com/vault/tutorials/auto-unseal/autounseal-transit
https://dev.to/luafanti/vault-auto-unseal-using-transit-secret-engine-on-kubernetes-13k8
https://www.bogotobogo.com/DevOps/Docker/Docker_Kubernetes_Vault_Consul_minikube_Auto_Unseal_Vault_Transit.php


### 10. Активируйте transit autounseal на vault-0

```
kubectl port-forward vault-0 -n vault 8200:8200
```
далее в отдельном окне:
```
export VAULT_ADDR=http://127.0.0.1:8200

vault login

vault secrets enable transit
vault write -f transit/keys/autounseal
```

### 11. Создайте политику autounseal, токен для которой позволит выполнять autounseal

```
cat > autounseal.hcl << EOF
path "transit/encrypt/autounseal" {
   capabilities = [ "update" ]
}

path "transit/decrypt/autounseal" {
   capabilities = [ "update" ]
}
EOF

vault policy write autounseal autounseal.hcl

```
### 12. Сгенерируйте orphan токен для политики autounseal с периодом 24 часа
```
vault token create -orphan -policy="autounseal" -period=24h
```
### 13. Ниже представлен helm сhart для инстанса vault, используемого для autounseal. Напишите конфиг vault для autounseal. Файл vault-auto-unseal-helm-values.yml


 Пример:
```
global:
  enabled: true
injector:
  enabled: "false"
server:
  standalone:
    enabled: true
    config: |
      <vault autounseal config>
```
Решение:


token  - токен надо брать выше из пункта 12.

```
cat > vault-auto-unseal-helm-values.yml << EOF

global:
  enabled: true
  tlsDisable: true
  serverTelemetry:
    prometheusOperator: true
injector:
  enabled: "false"
server:
  standalone:
    enabled: true
    config: |
      disable_mlock = true
      ui=true

      storage "file" {
        path = "/vault/data"
      }

      listener "tcp" {
        address = "0.0.0.0:8200"
        tls_disable = "true"
      }
      seal "transit" {
          address            = "http://vault.vault:8200"
          token              = "hvs.CAESID8OWjtrwnVdlZ2Qfd7KP2i9e0VBExcPG41cbYHMMauCGh4KHGh2cy5MeEU2b1hMcTNIUUVKVG1xODVZaGs0ZHU"
          key_name           = "autounseal"
          mount_path         = "transit/"
          tls_skip_verify    = "true"
      }

EOF
```

### 14. Установите чарт
```

helm install -n vault-a vault ./vault-custom -f vault-auto-unseal-helm-values.yml && kubectl -n vault-a exec -it vault-0 -- vault operator init | cat > .vault-recovery
```

проверка:


1. Перезапускаем кластер minikube `minikube stop && minikube start` (эмуляция сбоя)

2. в результате основной волт запечатан на vault\vaul-0 и требует ручного unseal

3. vault-a\vaul-0 соответственно сразу не стартует автоматом, так как "распечатывающий" vault\vaul-0 пока запечатан.

`kubectl logs -n vault-a -f vault-0 -c vault`

```
2023-03-11T18:32:16.158Z [INFO]  proxy environment: http_proxy="" https_proxy="" no_proxy=""
Error parsing Seal configuration: Error making API request.

URL: PUT http://vault.vault:8200/v1/transit/encrypt/autounseal
Code: 503. Errors:

* Vault is sealed
```

4. делаю unseal vault\vaul-0 `vault operator unseal`
Можно посмотреть логи
```
kubectl logs -n vault -f vault-0 -c vault
```

5. vault-a\vaul-0 распечатывается сам через некоторое время
```
2023-03-11T18:35:30.233Z [INFO]  core: vault is unsealed
2023-03-11T18:35:30.233Z [INFO]  core: unsealed with stored key
```


## User-pass авторизация
### 15. Инициализируйте секреты по путям prod, stage, dev и разрешите userpass

```
vault auth enable userpass && vault auth enable -path prod userpass && vault auth enable -path stage userpass && vault auth enable -path dev userpass
```

### 16.Создайте политику secret-admin-policy, которая будет удовлетворять следующим условиям:

- по пути "auth/*" будут доступны следующие разрешения: все, кроме patch и deny
- по пути "sys/auth/*" будут доступны: "create", "update", "delete", "sudo"
- по пути "sys/auth" будет доступно только чтение: "read"
- по пути "sys/policies/acl" только листинг ACL: "list"
- по путям "sys/policies/acl/", "secret/", "prod/*", "stage/*", "dev/*", "sys/mounts*": все, кроме patch и deny
- по пути "sys/health" следующие разрешения: "read", "sudo"

```
vault policy write -tls-skip-verify admin - << EOF

path "auth/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "sys/auth/*" {
  capabilities = ["create", "update", "delete", "sudo"]
}

path "sys/auth" {
  capabilities = ["read"]
}

path "sys/policies/acl" {
  capabilities = ["list"]
}

path "sys/policies/acl/" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "secret/" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "prod/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "stage/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "dev/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "sys/mounts*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "sys/health" {
  capabilities = ["read", "sudo"]
}

EOF
```

### 17. Создайте политику developer, удовлетворяющую следующим условиям:

- по пути "prod/*" - "read", "create", "update"
- по пути "stage/*" - "read", "create", "update"
- по пути "dev/*" - "read", "create", "update"

```
vault policy write -tls-skip-verify developer - << EOF

path "prod/*" {
  capabilities = ["create", "read", "update"]
}

path "stage/*" {
  capabilities = ["create", "read", "update"]
}

path "dev/*" {
  capabilities = ["create", "read", "update"]
}

EOF
```

### 18. Создайте политику junior, удовлетворяющую следующим условиям:

по пути "stage/*" - "read", "create", "update"

```
vault policy write -tls-skip-verify junior - << EOF

path "stage/*" {
  capabilities = ["create", "read", "update"]
}

EOF
```

### 19. Создайте пользователей:

- admin c паролем nimda и ранее созданной политикой admin
- developer c паролем ved и ранее созданной политикой developer
- junior с паролем roinuj и ранее созданой политикой junior.

```
vault write auth/userpass/users/admin policies=admin password=nimda
vault write auth/userpass/users/developer policies=developer password=ved
vault write auth/userpass/users/junior policies=junior password=roinuj
```


## PKI

### 20. Активируйте PKI по пути rebrain-pki, max=lease=ttl=8760h

```
vault secrets enable -path=rebrain-pki -max-lease-ttl=8760h pki
```

### 21. Скачайте наш сертификат по ссылке bundle.pem

```
wget https://storage.yandexcloud.net/files.rebrainme.com/workshops/hashicorp-vault/bundle.pem
```

### 22. Запишите скачанный сертификат по пути rebrain-pki/config/ca

```
vault write rebrain-pki/config/urls issuing_certificates="http://127.0.0.1:8200/v1/pki/ca" crl_distribution_point="http://127.0.0.1:8200/v1/pki/crl"

vault write rebrain-pki/config/ca pem_bundle=@bundle.pem
```

### 23. Создайте роль rebrain-pki/roles/local-certs для создания сертификата со следующими параметрами:
- max_ttl 24 часа
- localhost запрещен
- разрешенный домен myapp.st_login.rebrain.me
- разрешить bare domain
- запретить поддомены
- запретить wildcard сертификаты
- запретить ip sans

```
vault write rebrain-pki/roles/local-certs \
allowed_domains="myapp.st_login.rebrain.me" \
allow_subdomains=false \
allow_wildcard_certificates=false \
allow_localhost=false \
allow_glob_domains=true \
enforce_hostnames=false \
allow_ip_sans=false \
allow_client=true \
max_ttl="24h" \
allow_server=false \
allow_client=true \
allow_any_name=true \
allow_bare_domains=true

```

### 24. Создайте политику cert-issue-policy, которая будет удовлетворять следующим условиям:
- по пути "rebrain-pki*" - "read", "list"
- по пути "rebrain-pki/sign/local-certs" - "create", "update"
- по пути "rebrain-pki/issue/local-certs" - "create"

```
vault policy write -tls-skip-verify cert-issue-policy - << EOF

path "rebrain-pki*" {
  capabilities = ["read", "list"]
}

path "rebrain-pki/sign/local-certs" {
  capabilities = ["create", "update"]
}

path "rebrain-pki/issue/local-certs" {
  capabilities = ["create"]
}

EOF
```

### 25. Активируйте аутентификацию kubernetes. В качестве хоста используйте https://$KUBERNETES_PORT_443_TCP_ADDR:443

https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-minikube-raft


```
kubectl describe svc kubernetes | grep IP

export K8S_HOST=https://10.96.0.1

vault auth enable kubernetes


vault write auth/kubernetes/config \
kubernetes_host="$K8S_HOST" 

```
проверяем результат настройки
```
vault read auth/kubernetes/config
```

### 26. Создайте роль auth/kubernetes/role/issuer с политикой cert-issue-policy, к которой могут обращаться только сервисные аккаунты с именем issuer из пространства имен default, ttl=20m.
https://docs.armory.io/continuous-deployment/armory-admin/secrets/vault-k8s-configuration/
```
vault write auth/kubernetes/role/issuer \
        bound_service_account_names=issuer \
        bound_service_account_namespaces=default \
        policies=cert-issue-policy \
        ttl=20m
```

### 27. Скачайте и установите cert-manager

https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-cert-manager

```
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.9.1/cert-manager.crds.yaml

helm install cert-manager \
    --namespace cert-manager \
    --version v1.9.1 \
   jetstack/cert-manager
```   
### 28. Создайте в kubernetes сервисный аккаунт issuer
```
kubectl create serviceaccount issuer
```

### 29. Добавьте секрет в kubernetes и настройте certmanager


issuer-secret.yaml
```
cat > issuer-secret.yaml << EOF
apiVersion: v1
kind: Secret
metadata:
  name: issuer-token-lmzpj
  annotations:
    kubernetes.io/service-account.name: issuer
type: kubernetes.io/service-account-token

EOF
```
```
kubectl apply -f issuer-secret.yaml
```
```
export ISSUER_SECRET_REF=$(kubectl get secrets --output=json | jq -r '.items[].metadata | select(.name|startswith("issuer-token-")).name')

```
`kubectl get svc -n vault | grep vault-ui` и там внутренний ip сервиса vault-ui -> далее заменяем в манифесте ниже %vault-ui-ip%:

```

cat > vault-issuer.yaml << EOF
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: vault-issuer
  namespace: default
spec:
  vault:
    server: http://10.105.143.251:8200
    path: rebrain-pki/sign/local-certs
    auth:
      kubernetes:
        mountPath: /v1/auth/kubernetes
        role: issuer
        secretRef:
          name: $ISSUER_SECRET_REF
          key: token
EOF

kubectl apply --filename vault-issuer.yaml

cat > myapp-cert.yaml <<EOF
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: myapp
  namespace: default
spec:
  secretName: myapp-tls
  issuerRef:
    name: vault-issuer
  commonName: myapp.st_login.rebrain.me
  dnsNames:
  - myapp.st_login.rebrain.me
EOF

kubectl apply --filename myapp-cert.yaml
```

!!!Verifying the issuer Deployment
https://cert-manager.io/docs/configuration/vault/

Проверить можно следующим образом:

```
kubectl get issuers vault-issuer -n default -o wide
kubectl get certificates

Пример вывода:
user@rebrain-host:~$ kubectl get issuers vault-issuer -n default -o wide
NAME           READY   STATUS           AGE
vault-issuer   True    Vault verified   7s
user@rebrain-host:~$ kubectl get certificates
NAME    READY   SECRET      AGE
myapp   True    myapp-tls   25s

```

## Мониторинг
### 30. Измените тип сервисов prometheus и grafana в пространстве имен monitoring с ClusterIP на NodePort
```
kubectl edit svc -n monitoring prometheus-kube-prometheus-prometheus
kubectl edit svc -n monitoring prometheus-grafana
```
Данные для подключения к графане UserName: admin Password: prom-operator

### 31. Создайте файл с настройками для Helm чарта loki-stack-values.yml
```
cat > loki-stack-values.yml <<EOF

loki:
 enabled: true
 persistence:
  enabled: false
  storageClassName: standard

promtail:
 enabled: true
 pipelineStages:
  - cri: {}
  - json:
    expressions:
     is_even: is_even
     level: level
     version: version

grafana:
 enabled: false

EOF
``` 
### 32. Разверните helm chart
```
helm install loki grafana/loki-stack -n monitoring -f ~/loki-stack-values.yml
```
### 33.Настройте vault для логгирования в stdout

https://developer.hashicorp.com/vault/docs/audit/file
```
vault audit enable file file_path=stdout
```

Можете выполнить port-forwarding для настройки графаны снаружи
```
kubectl port-forward -n monitoring svc/prometheus-grafana --address=0.0.0.0 3000:80
```
```
kubectl edit svc -n monitoring loki

 и поправить тоже на NodePort как в пункте 30.
 Далее в настройках графаны добавляем источник http://loki.monitoring:3100/

```

### 34. Сконфигурируйте dashboard grafana для произвольных метрик vault.

Отправьте на проверку куратору скриншоты Garafana:
1. настроек Data Source Loki и Prometheus
2. Дашборд мониторинга Prometheus.
3. Дашборд логов Loki.
Мы не ограничиваем в количестве панелей в дашбордах Loki и Prometheus, хотя бы 2-3 панели в каждом дашборде должно быть.

Перед отправкой задания на проверку убедитесь, что все инстансы vault открыты и запущены


Полезные ссылки:

1. Vault Installation to Minikube via Helm with Integrated Storage
https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-minikube-raft

2. Configure Vault as a Certificate Manager in Kubernetes with Helm
https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-cert-manager

3. Vault and Cert Manager
https://cert-manager.io/docs/configuration/vault/

4. https://github.com/cert-manager/cert-manager/issues/2969

5. https://github.com/cert-manager/cert-manager/issues/4144

6. user@rebrain-host:~$ kubectl get issuers vault-issuer -n default -o wide
NAME           READY   STATUS                                                        AGE
vault-issuer   False   Vault Kubernetes auth requires both role and secretRef.name   10m

https://www.ibm.com/docs/en/cloud-paks/cp-management/2.1.x?topic=manager-using-vault-issue-certificates

7. https://grafana.com/blog/2021/11/02/introducing-new-integrations-to-make-it-easier-to-monitor-vault-with-grafana/
7.1 https://habr.com/ru/flows/admin/ - 

8. https://developer.hashicorp.com/vault/docs/audit/file


9. 
https://grafana.com/grafana/dashboards/12019-loki-dashboard-quick-search/


---Grana Vault dashboard-JSON-Model-----
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": {
          "type": "prometheus",
          "uid": "prometheus"
        },
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      }
    ]
  },
  "description": " Hashicorp Vault Metrics",
  "editable": true,
  "fiscalYearStartMonth": 0,
  "gnetId": 12904,
  "graphTooltip": 1,
  "id": 30,
  "links": [],
  "liveNow": false,
  "panels": [
    {
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "fieldConfig": {
        "defaults": {
          "mappings": [
            {
              "options": {
                "1": {
                  "text": "SEALED"
                },
                "2": {
                  "text": "UNSEALED"
                }
              },
              "type": "value"
            }
          ],
          "noValue": "N/A",
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "red",
                "value": null
              },
              {
                "color": "yellow",
                "value": 1
              },
              {
                "color": "green",
                "value": 2
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 6,
        "x": 0,
        "y": 0
      },
      "id": 47,
      "options": {
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "last"
          ],
          "fields": "",
          "values": false
        },
        "textMode": "auto"
      },
      "pluginVersion": "9.3.8",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "max(1 + vault_core_unsealed{})",
          "format": "time_series",
          "interval": "",
          "legendFormat": "{{ instance }}",
          "refId": "A"
        }
      ],
      "title": "Sealed Status",
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "fieldConfig": {
        "defaults": {
          "mappings": [],
          "noValue": "N/A",
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 3,
        "x": 6,
        "y": 0
      },
      "id": 53,
      "options": {
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "textMode": "auto"
      },
      "pluginVersion": "9.3.8",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg(vault_token_count)",
          "interval": "",
          "legendFormat": "",
          "refId": "A"
        }
      ],
      "title": "Available Tokens",
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "fieldConfig": {
        "defaults": {
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 5,
        "x": 9,
        "y": 0
      },
      "id": 78,
      "options": {
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "textMode": "auto"
      },
      "pluginVersion": "9.3.8",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg(vault_identity_num_entities)",
          "interval": "",
          "legendFormat": "",
          "refId": "A"
        }
      ],
      "title": "Number of Identity Entities",
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "fieldConfig": {
        "defaults": {
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "#EAB839",
                "value": 100
              },
              {
                "color": "#EF843C",
                "value": 200
              },
              {
                "color": "red",
                "value": 400
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 5,
        "x": 14,
        "y": 0
      },
      "id": 95,
      "options": {
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "textMode": "auto"
      },
      "pluginVersion": "9.3.8",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg(vault_expire_num_leases)",
          "interval": "",
          "legendFormat": "",
          "refId": "A"
        }
      ],
      "title": "Number of Leases",
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "fieldConfig": {
        "defaults": {
          "mappings": [],
          "noValue": "0",
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "orange",
                "value": 1
              },
              {
                "color": "red",
                "value": 3
              }
            ]
          },
          "unit": "none"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 3,
        "x": 19,
        "y": 0
      },
      "id": 8,
      "links": [],
      "maxDataPoints": 100,
      "options": {
        "colorMode": "value",
        "fieldOptions": {
          "calcs": [
            "lastNotNull"
          ]
        },
        "graphMode": "none",
        "justifyMode": "auto",
        "orientation": "horizontal",
        "reduceOptions": {
          "calcs": [
            "last"
          ],
          "fields": "",
          "values": false
        },
        "textMode": "auto"
      },
      "pluginVersion": "9.3.8",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg(vault_token_create_count - vault_token_store_count)",
          "format": "time_series",
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "",
          "refId": "A"
        }
      ],
      "title": "Pending Tokens",
      "type": "stat"
    },
    {
      "aliasColors": {},
      "bars": true,
      "dashLength": 10,
      "dashes": false,
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "decimals": 0,
      "fieldConfig": {
        "defaults": {
          "links": []
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 8,
        "w": 21,
        "x": 0,
        "y": 4
      },
      "hiddenSeries": false,
      "id": 104,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": true,
        "max": true,
        "min": false,
        "rightSide": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": false,
      "linewidth": 1,
      "nullPointMode": "connected",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "9.3.8",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg without(instance) (vault_token_count_by_policy)",
          "interval": "",
          "legendFormat": "{{ policy }}",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "Tokens by Policy",
      "tooltip": {
        "shared": false,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "series",
        "show": true,
        "values": [
          "current"
        ]
      },
      "yaxes": [
        {
          "$$hashKey": "object:575",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:576",
          "format": "short",
          "logBase": 1,
          "show": true
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {},
      "bars": true,
      "dashLength": 10,
      "dashes": false,
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "decimals": 0,
      "fieldConfig": {
        "defaults": {
          "links": []
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 12
      },
      "hiddenSeries": false,
      "id": 102,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": true,
        "max": true,
        "min": false,
        "rightSide": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": false,
      "linewidth": 1,
      "nullPointMode": "connected",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "9.3.8",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg without(instance) (vault_token_count_by_ttl)",
          "format": "time_series",
          "interval": "",
          "legendFormat": "{{ creation_ttl }}",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "Tokens by TTL",
      "tooltip": {
        "shared": false,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "series",
        "show": true,
        "values": [
          "current"
        ]
      },
      "yaxes": [
        {
          "$$hashKey": "object:390",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:391",
          "format": "short",
          "logBase": 1,
          "show": true
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {},
      "bars": true,
      "dashLength": 10,
      "dashes": false,
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "decimals": 0,
      "fieldConfig": {
        "defaults": {
          "links": []
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 6,
        "w": 12,
        "x": 12,
        "y": 12
      },
      "hiddenSeries": false,
      "id": 65,
      "legend": {
        "alignAsTable": false,
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "rightSide": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": false,
      "linewidth": 1,
      "maxDataPoints": 100,
      "nullPointMode": "null as zero",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "9.3.8",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": true,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg by(auth_method, creation_ttl) (vault_token_creation)",
          "format": "time_series",
          "instant": false,
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "{{ auth_method }} - {{ creation_ttl }}",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "Tokens Creation by Method & TTL",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:1956",
          "format": "short",
          "label": "",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:1957",
          "format": "short",
          "logBase": 1,
          "show": false
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {
        "Create": "rgb(84, 183, 90)",
        "Store": "#0a437c"
      },
      "bars": true,
      "dashLength": 10,
      "dashes": false,
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "decimals": 0,
      "fieldConfig": {
        "defaults": {
          "links": []
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 6,
        "w": 12,
        "x": 12,
        "y": 18
      },
      "hiddenSeries": false,
      "id": 6,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": false,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "9.3.8",
      "pointradius": 5,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": true,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg without(instance) (vault_token_create_count)",
          "format": "time_series",
          "instant": false,
          "interval": "",
          "intervalFactor": 10,
          "legendFormat": "Create",
          "refId": "A"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg without(instance) (vault_token_store_count)",
          "format": "time_series",
          "hide": false,
          "instant": false,
          "interval": "",
          "intervalFactor": 10,
          "legendFormat": "Store",
          "refId": "B"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "Token Creation/Storage",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:877",
          "decimals": 0,
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:878",
          "format": "short",
          "logBase": 1,
          "show": false
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {},
      "bars": true,
      "dashLength": 10,
      "dashes": false,
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "decimals": 0,
      "fieldConfig": {
        "defaults": {
          "links": []
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 20
      },
      "hiddenSeries": false,
      "id": 100,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": true,
        "max": true,
        "min": false,
        "rightSide": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": false,
      "linewidth": 1,
      "nullPointMode": "connected",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "9.3.8",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg without(instance) (vault_token_count_by_auth)",
          "interval": "",
          "legendFormat": "{{ auth_method }}",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "Tokens by Auth Method",
      "tooltip": {
        "shared": false,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "series",
        "show": true,
        "values": [
          "current"
        ]
      },
      "yaxes": [
        {
          "$$hashKey": "object:136",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:137",
          "format": "short",
          "logBase": 1,
          "show": true
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {
        "GET": "#1f78c1"
      },
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "fieldConfig": {
        "defaults": {
          "links": []
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 6,
        "w": 12,
        "x": 12,
        "y": 24
      },
      "hiddenSeries": false,
      "id": 12,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "9.3.8",
      "pointradius": 5,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg(irate(vault_policy_get_policy_count[1m]))",
          "format": "time_series",
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "GET",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "Policy Get",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:2132",
          "decimals": 0,
          "format": "short",
          "logBase": 1,
          "min": "0",
          "show": true
        },
        {
          "$$hashKey": "object:2133",
          "format": "short",
          "logBase": 1,
          "show": true
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {
        "Lookup": "#0a50a1"
      },
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "decimals": 3,
      "fieldConfig": {
        "defaults": {
          "links": []
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 6,
        "w": 24,
        "x": 0,
        "y": 30
      },
      "hiddenSeries": false,
      "id": 14,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "hideEmpty": false,
        "hideZero": false,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "9.3.8",
      "pointradius": 5,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg(irate(vault_token_lookup_count[1m]))",
          "hide": false,
          "interval": "",
          "legendFormat": "Lookups",
          "refId": "B"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "Token Lookups",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:330",
          "decimals": 0,
          "format": "short",
          "label": "Lookups per second",
          "logBase": 1,
          "min": "0",
          "show": true
        },
        {
          "$$hashKey": "object:331",
          "format": "short",
          "logBase": 1,
          "show": false
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "collapsed": true,
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 36
      },
      "id": 45,
      "panels": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "description": "",
          "fieldConfig": {
            "defaults": {
              "mappings": [],
              "noValue": "N/A",
              "thresholds": {
                "mode": "absolute",
                "steps": [
                  {
                    "color": "green"
                  },
                  {
                    "color": "red",
                    "value": 0.2
                  }
                ]
              }
            },
            "overrides": []
          },
          "gridPos": {
            "h": 5,
            "w": 3,
            "x": 0,
            "y": 13
          },
          "id": 41,
          "options": {
            "colorMode": "value",
            "graphMode": "area",
            "justifyMode": "auto",
            "orientation": "auto",
            "reduceOptions": {
              "calcs": [
                "last"
              ],
              "fields": "",
              "values": false
            },
            "textMode": "auto"
          },
          "pluginVersion": "9.3.8",
          "targets": [
            {
              "datasource": {
                "type": "prometheus",
                "uid": "prometheus"
              },
              "expr": "avg(vault_runtime_heap_objects{} / vault_runtime_malloc_count{})",
              "interval": "",
              "intervalFactor": 10,
              "legendFormat": "",
              "refId": "A"
            }
          ],
          "title": "Heap Objects Used",
          "type": "stat"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "fieldConfig": {
            "defaults": {
              "mappings": [],
              "noValue": "N/A",
              "thresholds": {
                "mode": "absolute",
                "steps": [
                  {
                    "color": "green"
                  },
                  {
                    "color": "#EAB839",
                    "value": 70
                  },
                  {
                    "color": "#EF843C",
                    "value": 100
                  },
                  {
                    "color": "#E24D42",
                    "value": 150
                  }
                ]
              }
            },
            "overrides": []
          },
          "gridPos": {
            "h": 5,
            "w": 3,
            "x": 3,
            "y": 13
          },
          "id": 76,
          "options": {
            "colorMode": "value",
            "graphMode": "area",
            "justifyMode": "auto",
            "orientation": "auto",
            "reduceOptions": {
              "calcs": [
                "last"
              ],
              "fields": "",
              "values": false
            },
            "textMode": "auto"
          },
          "pluginVersion": "9.3.8",
          "targets": [
            {
              "datasource": {
                "type": "prometheus",
                "uid": "prometheus"
              },
              "expr": "avg(vault_runtime_num_goroutines{})",
              "interval": "",
              "legendFormat": "",
              "refId": "A"
            }
          ],
          "title": "Number of Goroutines",
          "type": "stat"
        },
        {
          "aliasColors": {},
          "bars": false,
          "dashLength": 10,
          "dashes": false,
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "decimals": 3,
          "fieldConfig": {
            "defaults": {
              "links": []
            },
            "overrides": []
          },
          "fill": 1,
          "fillGradient": 0,
          "gridPos": {
            "h": 5,
            "w": 18,
            "x": 6,
            "y": 13
          },
          "hiddenSeries": false,
          "id": 43,
          "legend": {
            "alignAsTable": true,
            "avg": true,
            "current": true,
            "hideEmpty": false,
            "hideZero": false,
            "max": true,
            "min": true,
            "rightSide": false,
            "show": true,
            "total": false,
            "values": true
          },
          "lines": true,
          "linewidth": 1,
          "nullPointMode": "null",
          "options": {
            "alertThreshold": true
          },
          "percentage": false,
          "pluginVersion": "9.3.8",
          "pointradius": 2,
          "points": false,
          "renderer": "flot",
          "seriesOverrides": [],
          "spaceLength": 10,
          "stack": false,
          "steppedLine": false,
          "targets": [
            {
              "datasource": {
                "type": "prometheus",
                "uid": "prometheus"
              },
              "expr": "avg(vault_runtime_alloc_bytes{})",
              "interval": "",
              "intervalFactor": 5,
              "legendFormat": "$node",
              "refId": "A"
            }
          ],
          "thresholds": [],
          "timeRegions": [],
          "title": "Allocated MB",
          "tooltip": {
            "shared": true,
            "sort": 0,
            "value_type": "individual"
          },
          "type": "graph",
          "xaxis": {
            "mode": "time",
            "show": true,
            "values": []
          },
          "yaxes": [
            {
              "$$hashKey": "object:373",
              "format": "decbytes",
              "logBase": 1,
              "show": true
            },
            {
              "$$hashKey": "object:374",
              "format": "short",
              "logBase": 1,
              "show": true
            }
          ],
          "yaxis": {
            "align": false
          }
        }
      ],
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "refId": "A"
        }
      ],
      "title": "CPU/Mem Info: $node",
      "type": "row"
    },
    {
      "collapsed": false,
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 37
      },
      "id": 16,
      "panels": [],
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "refId": "A"
        }
      ],
      "title": "Token",
      "type": "row"
    },
    {
      "collapsed": false,
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 38
      },
      "id": 20,
      "panels": [],
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "refId": "A"
        }
      ],
      "title": "Audit",
      "type": "row"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "description": "",
      "fieldConfig": {
        "defaults": {
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 1
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 5,
        "w": 3,
        "x": 0,
        "y": 39
      },
      "id": 97,
      "maxDataPoints": 100,
      "options": {
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "max"
          ],
          "fields": "",
          "values": false
        },
        "textMode": "auto"
      },
      "pluginVersion": "9.3.8",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "max(idelta(vault_audit_log_request_failure[1m]))",
          "format": "time_series",
          "interval": "",
          "legendFormat": "Request",
          "refId": "A"
        }
      ],
      "title": "Log Request Failures",
      "type": "stat"
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "decimals": 3,
      "fieldConfig": {
        "defaults": {
          "links": []
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 10,
        "w": 11,
        "x": 3,
        "y": 39
      },
      "hiddenSeries": false,
      "id": 4,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "hideEmpty": false,
        "hideZero": false,
        "max": true,
        "min": true,
        "rightSide": false,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "9.3.8",
      "pointradius": 5,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg(irate(vault_audit_log_request_count[1m]))",
          "format": "time_series",
          "instant": false,
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "Request ",
          "refId": "A"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg(irate(vault_audit_log_response_count[1m]))",
          "format": "time_series",
          "hide": false,
          "instant": false,
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "Response",
          "refId": "B"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg(irate(vault_core_handle_request_count[1m]))",
          "format": "time_series",
          "hide": false,
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "Handled",
          "refId": "C"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "Log Requests",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:109",
          "decimals": 0,
          "format": "short",
          "label": "Requests per second",
          "logBase": 1,
          "min": "0",
          "show": true
        },
        {
          "$$hashKey": "object:110",
          "format": "short",
          "logBase": 1,
          "show": false
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "fieldConfig": {
        "defaults": {
          "links": []
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 10,
        "w": 10,
        "x": 14,
        "y": 39
      },
      "hiddenSeries": false,
      "id": 61,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null as zero",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "9.3.8",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg(irate(vault_consul_get_count[1m]))",
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "GET",
          "refId": "A"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg(irate(vault_consul_put_count[1m]))",
          "interval": "",
          "legendFormat": "PUT",
          "refId": "B"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg(irate(vault_consul_delete_count[1m]))",
          "interval": "",
          "legendFormat": "DELETE",
          "refId": "C"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "irate(vault_consul_list_count{instance=\"$node:$port\"}[1m])",
          "interval": "",
          "legendFormat": "LIST",
          "refId": "D"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "Consul Requests",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:949",
          "format": "short",
          "label": "Requests per second",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:950",
          "format": "short",
          "logBase": 1,
          "show": false
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "description": "",
      "fieldConfig": {
        "defaults": {
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 1
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 5,
        "w": 3,
        "x": 0,
        "y": 44
      },
      "id": 98,
      "maxDataPoints": 100,
      "options": {
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "max"
          ],
          "fields": "",
          "values": false
        },
        "textMode": "auto"
      },
      "pluginVersion": "9.3.8",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "max(idelta(vault_audit_log_response_failure[1m]))",
          "format": "time_series",
          "interval": "",
          "legendFormat": "Request",
          "refId": "A"
        }
      ],
      "title": "Log Response Failures",
      "type": "stat"
    },
    {
      "collapsed": false,
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 49
      },
      "id": 18,
      "panels": [],
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "refId": "A"
        }
      ],
      "title": "Policy",
      "type": "row"
    },
    {
      "aliasColors": {
        "set": "#629e51"
      },
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "fieldConfig": {
        "defaults": {
          "links": []
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 6,
        "w": 12,
        "x": 0,
        "y": 50
      },
      "hiddenSeries": false,
      "id": 10,
      "legend": {
        "alignAsTable": false,
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "9.3.8",
      "pointradius": 5,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "avg(irate(vault_policy_set_policy_count[1m]))",
          "format": "time_series",
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "SET",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "Policy Set",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:1834",
          "format": "short",
          "logBase": 1,
          "min": "0",
          "show": true
        },
        {
          "$$hashKey": "object:1835",
          "format": "short",
          "logBase": 1,
          "show": true
        }
      ],
      "yaxis": {
        "align": false
      }
    }
  ],
  "refresh": false,
  "schemaVersion": 37,
  "style": "dark",
  "tags": [
    "vault"
  ],
  "templating": {
    "list": [
      {
        "current": {
          "isNone": true,
          "selected": false,
          "text": "None",
          "value": ""
        },
        "datasource": {
          "type": "prometheus",
          "uid": "prometheus"
        },
        "definition": "label_values(up{job=\"vault\"}, instance)",
        "hide": 2,
        "includeAll": false,
        "label": "Host:",
        "multi": false,
        "name": "node",
        "options": [],
        "query": {
          "query": "label_values(up{job=\"vault\"}, instance)",
          "refId": "Prometheus-node-Variable-Query"
        },
        "refresh": 1,
        "regex": "/([^:]+):.*/",
        "skipUrlSync": false,
        "sort": 1,
        "tagValuesQuery": "",
        "tagsQuery": "",
        "type": "query",
        "useTags": false
      },
      {
        "current": {
          "isNone": true,
          "selected": false,
          "text": "None",
          "value": ""
        },
        "datasource": {
          "type": "prometheus",
          "uid": "prometheus"
        },
        "definition": "label_values(up{job=\"vault\",instance=~\"$node:(.*)\"}, instance)",
        "hide": 2,
        "includeAll": false,
        "multi": false,
        "name": "port",
        "options": [],
        "query": {
          "query": "label_values(up{job=\"vault\",instance=~\"$node:(.*)\"}, instance)",
          "refId": "Prometheus-port-Variable-Query"
        },
        "refresh": 1,
        "regex": "/[^:]+:(.*)/",
        "skipUrlSync": false,
        "sort": 0,
        "tagValuesQuery": "",
        "tagsQuery": "",
        "type": "query",
        "useTags": false
      },
      {
        "allValue": "",
        "current": {
          "selected": false,
          "text": "All",
          "value": "$__all"
        },
        "datasource": {
          "type": "prometheus",
          "uid": "prometheus"
        },
        "definition": "label_values(vault_secret_kv_count, mount_point)",
        "hide": 0,
        "includeAll": true,
        "label": "Mount Point:",
        "multi": true,
        "name": "mountpoint",
        "options": [],
        "query": {
          "query": "label_values(vault_secret_kv_count, mount_point)",
          "refId": "Prometheus-mountpoint-Variable-Query"
        },
        "refresh": 2,
        "regex": "/(.*)//",
        "skipUrlSync": false,
        "sort": 1,
        "tagValuesQuery": "",
        "tagsQuery": "",
        "type": "query",
        "useTags": false
      }
    ]
  },
  "time": {
    "from": "now-30m",
    "to": "now"
  },
  "timepicker": {
    "refresh_intervals": [
      "10s",
      "30s",
      "1m",
      "5m",
      "15m",
      "30m",
      "1h",
      "2h",
      "1d"
    ],
    "time_options": [
      "5m",
      "15m",
      "1h",
      "6h",
      "12h",
      "24h",
      "2d",
      "7d",
      "30d"
    ]
  },
  "timezone": "",
  "title": "Hashicorp Vault",
  "uid": "vaults",
  "version": 3,
  "weekStart": ""
}