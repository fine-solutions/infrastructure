# Настройка мониторинга с помощью Zabbix

## Установка PostgreSQL

Для работы Zabbix необходимо использовать корректную версию PostgreSQL в соответствие с [документацией](https://www.zabbix.com/documentation/current/en/manual/installation/requirements). Для установки необходимо выполнить следующие команды:

```bash
# Установить корректную версию репозитория
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo dnf clean all

# Выключить текущую версию БД
sudo dnf -qy module disable postgresql

# Установить PostgreSQL
sudo dnf install -y postgresql16-server

# Инициализировать службу БД
sudo /usr/pgsql-16/bin/postgresql-16-setup initdb
sudo systemctl enable postgresql-16
sudo systemctl start postgresql-16
```

## Установить Zabbix-сервер

```bash
# Установить корректную версию репозитория
sudo rpm -Uvh https://repo.zabbix.com/zabbix/6.4/rhel/8/x86_64/zabbix-release-6.4-1.el8.noarch.rpm
sudo dnf clean all

# Переключить версию PHP
sudo dnf module switch-to php:7.4

# Установить Zabbix-сервер
sudo dnf install -y zabbix-server-pgsql zabbix-web-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-selinux-policy

# Настроить БД
sudo -u postgres createuser --pwprompt zabbix
sudo -u postgres createdb -O zabbix zabbix
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
```

Редактировать `sudo vi /etc/zabbix/zabbix_server.conf`:

```zabbixconf
DBPassword=[Password]
```

Редактировать `sudo vi /etc/nginx/conf.d/zabbix.conf`:

```nginxconf
listen 80;
server_name zabbix.fine-solutions.org;
```

Запустить сервер и агент

```bash
sudo systemctl restart zabbix-server nginx php-fpm
sudo systemctl enable zabbix-server nginx php-fpm
```

## Установить Zabbix-агент:

```bash
sudo dnf install -y zabbix-agent
```

Для запуска агента, сгенерировать PSK-хеш:

```bash
sudo sh -c "openssl rand -hex 32 > /etc/zabbix/zabbix_agentd.psk"
```

Редактировать `sudo vi /etc/zabbix/zabbix_agentd.conf`:

```zabbixconf
Hostname=[Name]
Server=[IP]
TLSConnect=psk
TLSAccept=psk
TLSPSKIdentity=PSK 001 # 001 — msk / 002 — spb / 003 — fra
TLSPSKFile=/etc/zabbix/zabbix_agentd.psk
```

Настроить файервол для работы агента

```bash
sudo firewall-cmd --permanent --new-service=zabbix
sudo firewall-cmd --permanent --service=zabbix --add-port=10050/tcp
sudo firewall-cmd --permanent --service=zabbix --set-short="Zabbix Agent"
sudo firewall-cmd --permanent --add-service=zabbix
sudo firewall-cmd --reload
```

Запустить агент
```bash
sudo systemctl start zabbix-agent
sudo systemctl enable zabbix-agent
```
