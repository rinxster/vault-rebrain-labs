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