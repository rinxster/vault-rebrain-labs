# VLT 11: Monitoring

## Описание:
Привет! В текущем задании мы изучили:

что такое мониторинг;
какие метрики необходимо наблюдать;
какие инструменты помогают следить за метриками;
как настроить мониторинг Vault с Grafana и Prometheus.
Вы можете посмотреть запись по ссылке. Также для Вашего удобства прикладываем презентацию к вебинару.

## Задание:

### 1. Создайте директории для задания по пути /tmp/vault-monitoring/:

vault-config
prometheus-config
grafana-config

```
mkdir -p /tmp/vault-monitoring/vault-config \
/tmp/vault-monitoring/prometheus-config \
/tmp/vault-monitoring/grafana-config
```
`docker volume create vault-data && docker volume create grafana-data`

### 2. Задайте переменные VAULT_ADDR=http://127.0.0.1:8200 и VAULT_HOME=/tmp/vault-monitoring

`export VAULT_ADDR=http://127.0.0.1:8200 && export VAULT_HOME=/tmp/vault-monitoring`

### 3. Создайте файл server.hcl в папке /tmp/vault-monitoring/vault-config с описанием конфигурации сервера Vault
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

### 4. Запустите и инициализируйте сервер Vault в Docker контейнере c одним ключом, сохранив его по пути /home/user/root_token. Создайте политику под названием prometheus-metrics, позволяющую читать метрики. Создайте токен для этой политики и запишите его по пути /tmp/vault-monitoring/prometheus-config/prometheus-token.
```
docker run \
  --cap-add=IPC_LOCK \
  -d \
  --name vault \
  -p 8200:8200 \
  --volume $VAULT_HOME/vault-config:/vault/config \
  --volume vault-data:/vault \
  vault server
```

`docker logs `

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

```
vault policy write prometheus-metrics - << EOF

path  "/sys/metrics" {
  capabilities = ["read"]
}

path  "/sys/internal/counters/activity" {
  capabilities = ["read"]
}

EOF
```
Альтернативный вариант политики
```
 vault policy write prometheus-metrics - << EOF
path "/sys/metrics" {
  capabilities = ["read"]
}
EOF
```



`vault policy list`

```
vault token create \
-field=token \
-policy prometheus-metrics \
> $VAULT_HOME/prometheus-config/prometheus-token
```

`grep 'Initial Root Token' $VAULT_HOME/.vault-init | awk '{print $NF}' > /home/user/root_token`



### 5. Создайте файл конфигурации prometheus по пути $VAULT_HOME/prometheus-config/prometheus.yml. Не забудьте поменять ip Vault.
```
scrape_configs:
- job_name: vault
    metrics_path: /v1/sys/metrics
    params:
      format: ['prometheus']
    scheme: http
    authorization:
      credentials_file: /etc/prometheus/prometheus-token
    static_configs:
    - targets: ['<vault_docker_ip_addr>:8200']
```

```
cat > $VAULT_HOME/prometheus-config/prometheus.yml << EOF

scrape_configs:
- job_name: vault
    metrics_path: /v1/sys/metrics
    params:
      format: ['prometheus']
    scheme: http
    authorization:
      credentials_file: /etc/prometheus/prometheus-token
    static_configs:
    - targets: ['<vault_docker_ip_addr>:8200']

EOF

```

### 6. Запустите контейнер prometheus.
```
docker run \
    -d \
    --name prometheus \
    -p 9090:9090 \
    --volume $VAULT_HOME/prometheus-config/prometheus.yml:/etc/prometheus/prometheus.yml \
    --volume $VAULT_HOME/prometheus-config/prometheus-token:/etc/prometheus/prometheus-token \
    prom/prometheus
```
### 7. Создайте файл с конфигурацией data source для prometheus по пути $VAULT_HOME/grafana-config/datasource.yml. Не забудьте поменять ip prometheus.
```
# config file version
apiVersion: 1

datasources:
- name: vault
  type: prometheus
  access: server
  orgId: 1
  url: http://<prometheus_docker_ip_addr>:9090
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

```
cat > $VAULT_HOME/grafana-config/datasource.yml << EOF

# config file version
apiVersion: 1

datasources:
- name: vault
  type: prometheus
  access: server
  orgId: 1
  url: http://<prometheus_docker_ip_addr>:9090
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


### 8. Запустите контейнер Grafana.

```
docker run \
    -d \
    --name grafana \
    -p 3000:3000 \
    --volume $VAULT_HOME/grafana-config/datasource.yml:/etc/grafana/provisioning/datasources/prometheus_datasource.yml \
    --volume grafana-data:/var/lib/grafana \
    grafana/grafana
```    
### 9. Создайте дашборд из следующего JSON:
Текст дашборда
```
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": {
          "type": "grafana",
          "uid": "-- Grafana --"
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
  "editable": true,
  "fiscalYearStartMonth": 0,
  "graphTooltip": 0,
  "id": 1,
  "links": [],
  "liveNow": false,
  "panels": [
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PE6F0A1FBB43C8919"
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
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "short"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 9,
        "w": 12,
        "x": 0,
        "y": 0
      },
      "id": 2,
      "options": {
        "colorMode": "background",
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
      "pluginVersion": "9.3.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PE6F0A1FBB43C8919"
          },
          "editorMode": "builder",
          "expr": "vault_token_count",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Stored Tokens",
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PE6F0A1FBB43C8919"
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
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "short"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 12,
        "y": 0
      },
      "id": 4,
      "options": {
        "colorMode": "background",
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
      "pluginVersion": "9.3.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PE6F0A1FBB43C8919"
          },
          "editorMode": "builder",
          "expr": "vault_core_active",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Node Activity",
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PE6F0A1FBB43C8919"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 0,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 9
      },
      "id": 6,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PE6F0A1FBB43C8919"
          },
          "editorMode": "builder",
          "expr": "go_memstats_alloc_bytes",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Memory Allocated",
      "type": "timeseries"
    }
  ],
  "schemaVersion": 37,
  "style": "dark",
  "tags": [],
  "templating": {
    "list": []
  },
  "time": {
    "from": "now-6h",
    "to": "now"
  },
  "timepicker": {},
  "timezone": "",
  "title": "Vault Metrics",
  "uid": "P_5Kwy54z",
  "version": 1,
  "weekStart": ""
}
```
