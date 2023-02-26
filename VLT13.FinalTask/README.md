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
```

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

```
vault auth enable -path prod userpass && vault auth enable -path stage userpass && vault auth enable -path pdev userpass
```

### 16.Создайте политику secret-admin-policy, которая будет удовлетворять следующим условиям:

- по пути "auth/*" будут доступны следующие разрешения: все, кроме patch и deny
- по пути "sys/auth/*" будут доступны: "create", "update", "delete", "sudo"
- по пути "sys/auth" будет доступно только чтение: "read"
- по пути "sys/policies/acl" только листинг ACL: "list"
- по путям "sys/policies/acl/", "secret/", "prod/*", "stage/*", "dev/*", "sys/mounts*": все, кроме patch и deny
- по пути "sys/health" следующие разрешения: "read", "sudo"

```

vault policy write -tls-skip-verify secret-admin-policy - << EOF

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
vault write auth/corp-auth/users/admin policies=secret-admin-policy password=nimda
vault write auth/corp-auth/users/developer policies=developer password=ved
vault write auth/corp-auth/users/junior policies=junior password=roinuj

```

## PKI

### 20. Активируйте PKI по пути rebrain-pki, max=lease=ttl=8760h

### 21. Скачайте наш сертификат по ссылке bundle.pem

### 22. Запишите скачанный сертификат по пути rebrain-pki/config/ca

### 23. Создайте роль rebrain-pki/roles/local-certs для создания сертификата со следующими параметрами:
- max_ttl 24 часа
- localhost запрещен
- разрешенный домен myapp.st_login.rebrain.me
- разрешить bare domain
- запретить поддомены
- запретить wildcard сертификаты
- запретить ip sans

### 24. Создайте политику cert-issue-policy, которая будет удовлетворять следующим условиям:
- по пути "rebrain-pki*" - "read", "list"
- по пути "rebrain-pki/sign/local-certs" - "create", "update"
- по пути "rebrain-pki/issue/local-certs" - "create"

### 25. Активируйте аутентификацию kubernetes. В качестве хоста используйте https://$KUBERNETES_PORT_443_TCP_ADDR:443

### 26. Создайте роль auth/kubernetes/role/issuer с политикой cert-issue-policy, к которой могут обращаться только сервисные аккаунты с именем issuer из пространства имен default, ttl=20m.

### 27. Скачайте и установите cert-manager
```
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.9.1/cert-manager.crds.yaml

helm install cert-manager \
    --namespace cert-manager \
    --version v1.9.1 \
   jetstack/cert-manager
```   
### 28. Создайте в kubernetes сервисный аккаунт issuer
Добавьте секрет в kubernetes и настройте certmanager
issuer-secret.yaml
```
apiVersion: v1
kind: Secret
metadata:
  name: issuer-token-lmzpj
  annotations:
    kubernetes.io/service-account.name: issuer
type: kubernetes.io/service-account-token
```
```
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
``` 
### 32. Разверните helm chart
```
helm install loki grafana/loki-stack -n monitoring -f ~/loki-stack-values.yml
```
### 33.Настройте vault для логгирования в stdout
Можете выполнить port-forwarding для настройки графаны снаружи
```
kubectl port-forward -n monitoring svc/prometheus-grafana --address=0.0.0.0 3000:80
```
### 34. Сконфигурируйте dashboard grafana для произвольных метрик vault.

Отправьте на проверку куратору скриншоты Garafana:
1. настроек Data Source Loki и Prometheus
2. Дашборд мониторинга Prometheus.
3. Дашборд логов Loki.
Мы не ограничиваем в количестве панелей в дашбордах Loki и Prometheus, хотя бы 2-3 панели в каждом дашборде должно быть.

Перед отправкой задания на проверку убедитесь, что все инстансы vault открыты и запущены
