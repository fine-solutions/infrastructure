# Настройка логирования

## Использование `cron`

Для конкретного сайта состоит в написании скрипта для обновления логов, например для папки `/web/sites/example/logs`:

```bash
$FOLDERPATH=/web/sites/example/logs
gzip --best --name "$FOLDERPATH/access.log" 
gzip --best --name "$FOLDERPATH/error.log"
kill -USR1 `cat /var/run/nginx.pid`
mv "$FOLDERPATH/access.log.gz" "$FOLDERPATH/access.$(date +"%Y-%m-%d-%H-%M-%S").log.gz"
mv "$FOLDERPATH/error.log.gz "$FOLDERPATH/error.$(date +"%Y-%m-%d-%H-%M-%S").log.gz"
```

Добавление периода ротации (для пользователя `root`):

```bash
sudo crontab -e
```

```bash
0 0 * * 0 /web/sites/zabbix.fine-solutions.org/logs/rotate.sh
```
