# Настройка протокола HTTPS

## Настройка файервола

```bash
sudo firewall-cmd --add-service=https --permanent
```

## Настройка Let’s Encrypt

Установка snap и certbot:

```bash
sudo dnf -y install snapd
sudo systemctl enable --now snapd.socket
sudo systemctl start snapd
sudo ln -s /var/lib/snapd/snap /snap
sudo snap install core
sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

## Генерация сертификатов

Генерация обычного сертификата с автоматическим обновлением (работает почти всегда):

```bash
/usr/bin/certbot certonly --webroot -w /var/www/html --email root@example.com -d example.com -d www.example.com
```

Генерация сертификата Wildcard с ручным обновлением:

```bash
/usr/bin/certbot certonly --manual --preferred-challenges=dns --email root@example.com --agree-tos -d *.example.com
```

## Настройка самоподписанных сертфикатов (опционально):

```bash
openssl req -new -x509 -days 30 -nodes -newkey rsa:2048 -keyout server.key -out server.crt -subj "/C=RU/ST=Moskow/L=Moscow/CN=server_ip_address"
sudo mkdir -p /etc/pki/nginx/private
sudo cp server.crt /etc/pki/nginx/ip
sudo cp server.key /etc/pki/nginx/ip/private
```
