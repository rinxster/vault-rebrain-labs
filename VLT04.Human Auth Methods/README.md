# VLT 04: Human Auth Methods
***
## Описание
Прежде чем приступить к выполнению практического задания, рекомендуем Вам освежить в памяти темы вебинара. На уроке мы с Вами разобрали следующие вопросы:

Продвинутые AuthMethod-ы: JWT и LDAP
Интеграция Vault и Gitlab CI\CD
Интеграция Vault и FreeIPA LDAP
Response Wrapping и AppRole
Если Вы приступили к выполнению практического задания по прошествии нескольких дней с момента вебинара, Вам будет полезно посмотреть запись и повторить пройденный материал.

Также для Вашего удобства прикладываем презентацию к вебинару (обратите внимание, что информация в ней понадобится для выполнения текущего и следующего практического задания).
***
Подготовка:
В текущем задании Вам предоставляется две машины. На одной установлен vault, на второй необходимо будет установить freeipa. Для начала настроим freeipa на предоставленном сервере. Каждой машине выдается доменное имя, которое действует внутри сети облачного провайдера. То есть выданные машины «видят» друг друга по доменному имени, но из внешней сети домен разрешить не удастся. Добавьте домен машины с freeipa в Ваш hosts-файл, чтобы обращаться к ней по домену вне сети облачного провайдера. Пример:
```
1.1.1.1 abc.rbrvault.com
```

### 1. Выполните команду. Замените <DOMAIN_NAME>, <DOMAIN_NAME_UPPER_CASE> (домен в верхнем регистре), <SERVER_IP>. Запуск контейнера может занять некоторое время.
```
docker run -ti -h <DOMAIN_NAME> --read-only \
    -v /var/lib/ipa-data11:/data:Z \
    -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
    -e PASSWORD=Secret123 \
    -e IPA_SERVER_IP=<SERVER_IP> \
    -p 443:443 -p 389:389 -p 636:636 -p 88:88 -p 464:464 -p 123:123 \
    --sysctl net.ipv6.conf.all.disable_ipv6=0 \
    freeipa-fed36 ipa-server-install -r <DOMAIN_NAME_UPPER_CASE> --no-ntp
```
### 2. На вопросы при установке отвечайте следующим образом:
```
Do you want to configure integrated DNS (BIND)? [no]: 
Server host name [<ВАШ>.rbrvault.com]: 
Please confirm the domain name [rbrvault.com]: 
Do you wish to continue? [no]: yes
NetBIOS domain name [RBRVAULT]: 
Continue to configure the system with these values? [no]: yes
```
### 3. После установки зайдите в браузере по адресу https://<домен_freeipa> (необходимо добавление домена в hosts-файл). И авторизуйтесь
```
username: admin
password: Secret123
```
### 4. Добавьте двух пользователей с именами devjunior и devsenior и паролем password во вкладке Identity/Users.
### 5. Создайте две группы с названиями juniordevs и seniordevs во вкладке Identity/Groups.
### 6. Добавьте пользователя devjunior в группу juniordevs.
### 7. Добавьте пользователя devsenior в группу seniordevs.
### 8. Создайте пользователя vault с паролем password и добавьте его в группу admins.
***
# Задание:
Следующие пункты выполняйте на машине с Vault.

### 1. Проведите инициализацию кластера с 3 ключами, любые два из которых распечатывают Vault. Сохраните root token в файл /home/user/root_token.
```

export VAULT_ADDR=https://127.0.0.1:8200 && sudo systemctl restart vault && vault operator init -key-shares=3 -key-threshold=2 >> /home/user/vault_keys

touch root_token

echo "hvs.V4hJd4mtiSXLWrQboyQsZQ3F" >> root_token
```

### 2.Активируйте авторизацию по LDAP.
```
vault auth enable ldap 
```

### 3. Запишите в Vault данные для FreeIPA. Замените необходимые данные.

```
vault write auth/ldap/config \
    url="ldaps://<DOMAIN_NAME>" \
    userdn="cn=users,cn=accounts,dc=<ВАШ_ДОМЕН_3_УРОВНЯ>,dc=rbrvault,dc=com" \
    binddn="uid=vault,cn=users,cn=accounts,dc=<ВАШ_ДОМЕН_3_УРОВНЯ>,dc=rbrvault,dc=com" \
    bindpass='password' \
    groupdn="cn=groups,cn=accounts,dc=<ВАШ_ДОМЕН_3_УРОВНЯ>,dc=rbrvault,dc=com" \
    groupattr="cn" \
    userattr="uid" \
    insecure_tls=true \
    starttls=true \
```

```
vault write auth/ldap/config \
    url="ldaps://nrrrd.rbrvault.com" \
    userdn="cn=users,cn=accounts,dc=nrrrd,dc=rbrvault,dc=com" \
    binddn="uid=vault,cn=users,cn=accounts,dc=nrrrd,dc=rbrvault,dc=com" \
    bindpass='password' \
    groupdn="cn=groups,cn=accounts,dc=nrrrd,dc=rbrvault,dc=com" \
    groupattr="cn" \
    userattr="uid" \
    insecure_tls="true" \
    starttls="true
```
### 4.Настройте Vault так, чтобы пользователи FreeIPA из группы juniordevs имели политику dev-junior. Политику создавать не обязательно.
```
vault write auth/ldap/groups/juniordevs policies=dev-junior
```

### 5. Настройте Vault так, чтобы пользователи FreeIPA из группы seniordevs имели политику dev-senior. Политику создавать не обязательно.
```
vault write auth/ldap/groups/seniordevs policies=dev-senior
```