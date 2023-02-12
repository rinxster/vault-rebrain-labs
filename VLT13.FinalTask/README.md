# VLT: Финальное задание

## Описание:
Финальное задание для построения базовой высокодоступной инфраструктуры Vault в кластере Kubernetes.

Будет использован кастомный Helm Chart (из-за некоторых ограничений стандартного), лежащий в папке /home/user/vault-custom.

Необходимо использовать prometheus-operator для сбора метрик (указать в конфигурации vault).

## Задание:
### 1. Запустите minikube. Создайте необходимые namespaces
```
sudo sysctl -w fs.inotify.max_user_instances=8192
minikube start --driver docker --nodes 3
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
* UI активирован
* listener tcp:
  - tls отключен
  - прослушивает порт 8200 на всех интерфейсах (IPv4 и IPv6)
  - назначить адрес кластера на порту 8201
* в telemetry включить доступ к метрикам без авторизации
Настроить raft
по пути /vault/data
Автопилот
cleanup_dead_servers = "true"
last_contact_threshold = "200ms"
last_contact_failure_threshold = "10m"
max_trailing_logs = 250000
min_quorum = 5
server_stabilization_time = "10s"
Телеметрия
Prometheus хранит данные 24 часа
Отключить hostname
Зарегестрировать сервис kubernetes
vault-helm-config.yaml
### 7. Разверните кластер
```
helm install -n vault vault ./vault-custom -f vault-helm-config.yaml
```
### 8. Инициализируйте vault на поде vault-0 с 3 ключами, любые два из которых распечатывают Vault. Сохраните root-token по пути /home/user/root_token

Подключите raft для vault-1 и vault-2. Url для vault-0 в кластере http://vault-0.vault-internal:8200

## Настройка autounseal

### 10. Активируйте transit autounseal на vault-0
### 11. Создайте политику autounseal, токен для которой позволит выполнять autounseal
### 12. Сгенерируйте orphan токен для политики autounseal с периодом 24 часа
### 13. Ниже представлен helm сhart для инстанса vault, используемого для autounseal. Напишите конфиг vault для autounseal. Файл vault-auto-unseal-helm-values.yml
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
### 14. Установите чарт
```
helm install -n vault-a vault ./vault-custom -f vault-auto-unseal-helm-values.yml\
kubectl -n vault-a exec -it vault-0 -- vault operator init | cat > .vault-recovery
```

## User-pass авторизация
### 15. Инициализируйте секреты по путям prod, stage, dev и разрешите userpass

### 16.Создайте политику secret-admin-policy, которая будет удовлетворять следующим условиям:

- по пути "auth/*" будут доступны следующие разрешения: все, кроме patch и deny
- по пути "sys/auth/*" будут доступны: "create", "update", "delete", "sudo"
- по пути "sys/auth" будет доступно только чтение: "read"
- по пути "sys/policies/acl" только листинг ACL: "list"
- по путям "sys/policies/acl/", "secret/", "prod/*", "stage/*", "dev/*", "sys/mounts*": все, кроме patch и deny
- по пути "sys/health" следующие разрешения: "read", "sudo"

### 17. Создайте политику developer, удовлетворяющую следующим условиям:

по пути "prod/*" - "read", "create", "update"
по пути "stage/*" - "read", "create", "update"
по пути "dev/*" - "read", "create", "update"
Создайте политику junior, удовлетворяющую следующим условиям:

по пути "stage/*" - "read", "create", "update"
Создайте пользователей:

admin c паролем nimda и ранее созданной политикой admin
developer c паролем ved и ранее созданной политикой developer
junior с паролем roinuj и ранее созданой политикой junior.
PKI
Активируйте PKI по пути rebrain-pki, max=lease=ttl=8760h
Скачайте наш сертификат по ссылке bundle.pem
Запишите скачанный сертификат по пути rebrain-pki/config/ca
Создайте роль rebrain-pki/roles/local-certs для создания сертификата со следующими параметрами:
max_ttl 24 часа
localhost запрещен
разрешенный домен myapp.st_login.rebrain.me
разрешить bare domain
запретить поддомены
запретить wildcard сертификаты
запретить ip sans
Создайте политику cert-issue-policy, которая будет удовлетворять следующим условиям:

по пути "rebrain-pki*" - "read", "list"
по пути "rebrain-pki/sign/local-certs" - "create", "update"
по пути "rebrain-pki/issue/local-certs" - "create"
Активируйте аутентификацию kubernetes. В качестве хоста используйте https://$KUBERNETES_PORT_443_TCP_ADDR:443

Создайте роль auth/kubernetes/role/issuer с политикой cert-issue-policy, к которой могут обращаться только сервисные аккаунты с именем issuer из пространства имен default, ttl=20m.

Скачайте и установите cert-manager

kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.9.1/cert-manager.crds.yaml

helm install cert-manager \
    --namespace cert-manager \
    --version v1.9.1 \
   jetstack/cert-manager
Создайте в kubernetes сервисный аккаунт issuer
Добавьте секрет в kubernetes и настройте certmanager
issuer-secret.yaml

apiVersion: v1
kind: Secret
metadata:
  name: issuer-token-lmzpj
  annotations:
    kubernetes.io/service-account.name: issuer
type: kubernetes.io/service-account-token
kubectl apply -f issuer-secret.yaml

export ISSUER_SECRET_REF=$(kubectl get secrets --output=json | jq -r '.items[].metadata | select(.name|startswith("issuer-token-")).name')

cat > vault-issuer.yaml << EOF
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: vault-issuer
  namespace: cert-manager
spec:
  vault:
    server: http://%vault-ui-ip%:8200
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
  namespace: cert-manager
spec:
  secretName: myapp-tls
  issuerRef:
    name: vault-issuer
  commonName: myapp.st_login.rebrain.me
  dnsNames:
  - myapp.st_login.rebrain.me
EOF

kubectl apply --filename myapp-cert.yaml
Мониторинг
Измените тип сервисов prometheus и grafana в пространстве имен monitoring с ClusterIP на NodePort
kubectl edit svc -n monitoring prometheus-kube-prometheus-prometheus
kubectl edit svc -n monitoring prometheus-grafana
Данные для подключения к графане UserName: admin Password: prom-operator

Создайте файл с настройками для Helm чарта loki-stack-values.yml
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
Разверните helm chart
helm install loki grafana/loki-stack -n monitoring -f ~/loki-stack-values.yml
Настройте vault для логгирования в stdout
Можете выполнить port-forwarding для настройки графаны снаружи

kubectl port-forward -n monitoring svc/prometheus-grafana --address=0.0.0.0 3000:80
Сконфигурируйте dashboard grafana для произвольных метрик vault.
Отправьте на проверку куратору скриншоты Garafana:

настроек Data Source Loki и Prometheus
Дашборд мониторинга Prometheus.
Дашборд логов Loki.
Мы не ограничиваем в количестве панелей в дашбордах Loki и Prometheus, хотя бы 2-3 панели в каждом дашборде должно быть.

Перед отправкой задания на проверку убедитесь, что все инстансы vault открыты и запущены















----
# VLT 12: Audit
Описание:
Привет! В текущем задании мы изучим:

что такое логирование;
логи и их уровни;
аудит логов и инструменты;
как настроить логирование и аудит логов Vault на базе PLG-стека.
Вы можете посмотреть запись по ссылке. Также для Вашего удобства прикладываем презентацию к вебинару.

## Задание:
### 1. Создайте директории для задания по пути /tmp/vault-monitoring/:

vault-config
promtail-config
grafana-config
loki-config

```
mkdir -p /tmp/vault-monitoring/vault-config \
/tmp/vault-monitoring/promtail-config \
/tmp/vault-monitoring/grafana-config \
/tmp/vault-monitoring/loki-config
```

`docker volume create vault && docker volume create logs && docker volume create grafana-data`

### 2. Задайте переменные VAULT_ADDR=http://127.0.0.1:8200 и VAULT_HOME=/tmp/vault-monitoring. Создайте файл server.hcl в папке /tmp/vault-monitoring/vault-config с описанием конфигурации сервера Vault
```
api_addr  = "http://0.0.0.0:8200"

listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = "true"
}

storage "file" {
  path = "/vault/data"
}

telemetry {
  disable_hostname = true
  prometheus_retention_time = "12h"
}
```

`export VAULT_ADDR=http://127.0.0.1:8200 && export VAULT_HOME=/tmp/vault-monitoring`


```
cat >  $VAULT_HOME/vault-config/server.hcl << EOF

api_addr  = "http://0.0.0.0:8200"

listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = "true"
}

storage "file" {
  path = "/vault/data"
}

telemetry {
  disable_hostname = true
  prometheus_retention_time = "12h"
}
EOF
```

### 3. Запустите Vault в docker контейнере. Инициализируйте его, рут токен сохраните по пути /home/user/root_token. Настройте Audit Device для записи в файл /vault/logs/vault_audit.log
```
docker run \
  --cap-add=IPC_LOCK \
  -d \
  --name vault \
  -p 8200:8200 \
  --volume $VAULT_HOME/vault-config:/vault/config \
  --volume vault:/vault \
  --volume logs:/vault/logs \
  vault server
```
`docker logs vault`


```
vault operator init -key-shares=1 -key-threshold=1 \
| head -n3 \
| cat > $VAULT_HOME/.vault-init

```

```
vault operator unseal \
$(grep 'Unseal Key 1' $VAULT_HOME/.vault-init | awk '{print $NF}')
```

```
vault login -no-print \
$(grep 'Initial Root Token' $VAULT_HOME/.vault-init | awk '{print $NF}')
```

`grep 'Initial Root Token' $VAULT_HOME/.vault-init | awk '{print $NF}' > /home/user/root_token`

`vault audit enable file file_path=/vault/logs/vault_audit.log mode="0644"`

`vault audit list`


### 4. Настройте контейнер Loki для приема лог-стримов и запустите его. Файл конфига разместите по пути $VAULT_HOME/loki-config/loki-config.yaml.
```
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096

ingester:
  wal:
    enabled: true
    dir: /tmp/wal
  lifecycler:
    address: 0.0.0.0
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
    final_sleep: 0s
  chunk_idle_period: 1h       
  max_chunk_age: 1h           
  chunk_target_size: 1048576  
  chunk_retain_period: 30s   

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

storage_config:
  boltdb_shipper:
    active_index_directory: /tmp/loki/boltdb-shipper-active
    cache_location: /tmp/loki/boltdb-shipper-cache
    cache_ttl: 24h         
    shared_store: filesystem
  filesystem:
    directory: /tmp/loki/chunks

compactor:
  working_directory: /tmp/loki/boltdb-shipper-compactor
  shared_store: filesystem

limits_config:
  reject_old_samples: true
  reject_old_samples_max_age: 168h

chunk_store_config:
  max_look_back_period: 0s

table_manager:
  retention_deletes_enabled: false
  retention_period: 0s

ruler:
  storage:
    type: local
    local:
      directory: /tmp/loki/rules
  rule_path: /tmp/loki/rules-temp
  alertmanager_url: http://0.0.0.0:9093
  ring:
    kvstore:
      store: inmemory
  enable_api: true
```
запускаем контейнер с loki
```
docker run \
    -d \
    --name loki \
    -p 3100:3100 \
    --volume $VAULT_HOME/loki-config:/tmp/loki-config \
    grafana/loki -config.file=/tmp/loki-config/loki-config.yaml
```
### 5. Настройте контейнер Promtail для забора лог-файлов и отправки его в Loki (не забудьте указать ip адрес контейнера loki для пуша логов). Файл конфига разместите по пути $VAULT_HOME/promtail-config/promtail-config.yaml
```
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://<loki_container_ip>:3100/loki/api/v1/push

scrape_configs:
- job_name: system
  static_configs:
  - targets:
      - 0.0.0.0
    labels:
      job: varlogs
      __path__: /opt/log/*log
```
```
cat > $VAULT_HOME/promtail-config/promtail-config.yaml << EOF

server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://<loki_container_ip>:3100/loki/api/v1/push

scrape_configs:
- job_name: system
  static_configs:
  - targets:
      - 0.0.0.0
    labels:
      job: varlogs
      __path__: /opt/log/*log

EOF

```

```
docker run \
  -d \
  --name promtail \
  --volume logs:/opt/log \
  --volume $VAULT_HOME/promtail-config:/tmp/promtail \
  grafana/promtail -config.file=/tmp/promtail/promtail-config.yaml
```
### 6. Добавьте Loki data source для Grafana и запустите Grafana
```
apiVersion: 1

datasources:
- name: Loki
  type: loki
  access: server
  orgId: 1
  url: http://<loki_container_ip_addr>:3100
  password:
  user:
  database:
  basicAuth:
  basicAuthUser:
  basicAuthPassword:
  withCredentials:
  isDefault:
  jsonData:
     graphiteVersion: "1.1"
     tlsAuth: false
     tlsAuthWithCACert: false
  secureJsonData:
    tlsCACert: ""
    tlsClientCert: ""
    tlsClientKey: ""
  version: 1
  editable: true
```

создаём файл конфигурации с конфигом выше и указываем в нем предварительно внешний адрес сервера
```
cat >  $VAULT_HOME/grafana-config/datasource.yml << EOF

apiVersion: 1

datasources:
- name: Loki
  type: loki
  access: server
  orgId: 1
  url: http://<loki_container_ip_addr>:3100
  password:
  user:
  database:
  basicAuth:
  basicAuthUser:
  basicAuthPassword:
  withCredentials:
  isDefault:
  jsonData:
     graphiteVersion: "1.1"
     tlsAuth: false
     tlsAuthWithCACert: false
  secureJsonData:
    tlsCACert: ""
    tlsClientCert: ""
    tlsClientKey: ""
  version: 1
  editable: true

EOF
```

запускаем Grafana

```
docker run \
    -d \
    --name grafana \
    -p 3000:3000 \
    --volume $VAULT_HOME/grafana-config/datasource.yml:/etc/grafana/provisioning/datasources/prometheus_datasource.yml \
    --volume grafana-data:/var/lib/grafana \
    grafana/grafana
```   


### 7. Создайте дашборд для мониторинга логов. Возможно вам будет необходимо поменять все Datasource UID. Для того, чтобы узнать актуальный datasource uid, зайдите по пути Configuration -> Data Soruces -> Loki. Кликнув на него, в адресной строке увидите его uid. Например: http://<ip>/datasources/edit/<datasource uid>. В дашборде замените все значения следующей структуры на актуальные.
```
"datasource": {
  "type": "loki",
  "uid": "P8E80F9AEF21F6940"
}
```
Текст дашборда
```
{
  "annotations": {
    "list": [
      {
        "$$hashKey": "object:75",
        "builtIn": 1,
        "datasource": {
          "type": "datasource",
          "uid": "grafana"
        },
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "target": {
          "limit": 100,
          "matchAny": false,
          "tags": [],
          "type": "dashboard"
        },
        "type": "dashboard"
      }
    ]
  },
  "description": "Loki logs panel with prometheus variables ",
  "editable": true,
  "fiscalYearStartMonth": 0,
  "gnetId": 12019,
  "graphTooltip": 0,
  "id": 2,
  "links": [],
  "liveNow": false,
  "panels": [
    {
      "aliasColors": {},
      "bars": true,
      "dashLength": 10,
      "dashes": false,
      "datasource": {
        "type": "loki",
        "uid": "P8E80F9AEF21F6940"
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
        "h": 3,
        "w": 24,
        "x": 0,
        "y": 0
      },
      "hiddenSeries": false,
      "id": 6,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": false,
        "total": false,
        "values": false
      },
      "lines": false,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "9.3.2",
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
            "type": "loki",
            "uid": "P8E80F9AEF21F6940"
          },
          "editorMode": "code",
          "expr": "sum(count_over_time({job=\"$Job\"} |~ \"$search\"[$__interval]))",
          "queryType": "range",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "Search count",
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
          "$$hashKey": "object:168",
          "format": "short",
          "logBase": 1,
          "show": false
        },
        {
          "$$hashKey": "object:169",
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
        "type": "loki",
        "uid": "P8E80F9AEF21F6940"
      },
      "gridPos": {
        "h": 25,
        "w": 24,
        "x": 0,
        "y": 3
      },
      "id": 2,
      "maxDataPoints": "",
      "options": {
        "dedupStrategy": "none",
        "enableLogDetails": true,
        "prettifyLogMessage": false,
        "showCommonLabels": false,
        "showLabels": false,
        "showTime": true,
        "sortOrder": "Descending",
        "wrapLogMessage": true
      },
      "targets": [
        {
          "datasource": {
            "type": "loki",
            "uid": "P8E80F9AEF21F6940"
          },
          "editorMode": "builder",
          "expr": "{job=\"$Job\"} |~ `(?i)$search`",
          "queryType": "range",
          "refId": "A"
        }
      ],
      "title": "Logs Panel",
      "type": "logs"
    },
    {
      "datasource": {
        "type": "datasource",
        "uid": "grafana"
      },
      "gridPos": {
        "h": 3,
        "w": 24,
        "x": 0,
        "y": 28
      },
      "id": 4,
      "options": {
        "code": {
          "language": "plaintext",
          "showLineNumbers": false,
          "showMiniMap": false
        },
        "content": "
 For Grafana Loki blog example 
\n\n\n",
        "mode": "html"
      },
      "pluginVersion": "9.3.2",
      "targets": [
        {
          "datasource": {
            "type": "datasource",
            "uid": "grafana"
          },
          "refId": "A"
        }
      ],
      "transparent": true,
      "type": "text"
    }
  ],
  "schemaVersion": 37,
  "style": "dark",
  "tags": [],
  "templating": {
    "list": [
      {
        "current": {
          "selected": false,
          "text": "varlogs",
          "value": "varlogs"
        },
        "datasource": {
          "type": "loki",
          "uid": "P8E80F9AEF21F6940"
        },
        "definition": "",
        "hide": 0,
        "includeAll": false,
        "multi": false,
        "name": "Job",
        "options": [],
        "query": {
          "label": "job",
          "refId": "LokiVariableQueryEditor-VariableQuery",
          "stream": "",
          "type": 1
        },
        "refresh": 1,
        "regex": "",
        "skipUrlSync": false,
        "sort": 0,
        "type": "query"
      },
      {
        "current": {
          "selected": false,
          "text": "error",
          "value": "error"
        },
        "hide": 0,
        "name": "search",
        "options": [
          {
            "selected": true,
            "text": "level=warn",
            "value": "level=warn"
          }
        ],
        "query": "error",
        "skipUrlSync": false,
        "type": "textbox"
      }
    ]
  },
  "time": {
    "from": "now-6h",
    "to": "now"
  },
  "timepicker": {
    "refresh_intervals": [
      "5s",
      "10s",
      "30s",
      "1m",
      "5m",
      "15m",
      "30m",
      "1h",
      "2h",
      "1d"
    ]
  },
  "timezone": "",
  "title": "Loki Dashboard quick search",
  "uid": "liz0yRCZz",
  "version": 2,
  "weekStart": ""
}
```