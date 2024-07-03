# Установка Nginx из исходников

## Установить пакеты

```bash
sudo dnf -y update
sudo dnf -y install gcc gcc-c++ gcc-toolset-11-gcc gcc-toolset-11-gcc-c++ kernel-devel dkms socat bash-completion socat pcre pcre-devel gd gd-devel zlib zlib-devel openssl openssl-devel GeoIP-devel libxslt-devel libxml2-devel bzip2-devel libffi-devel zlib-devel make
yum groupinstall 'Development Tools'
source /opt/rh/gcc-toolset-11/enable
gcc -v
sudo dnf install cmake
```

## Удалить стоящий по умолчанию Nginx

```bash
dnf --showduplicates list nginx

# В первой строчке вывода предыдущей команды будет текущая версия, установленная в системе
dnf remove nginx-1.20.1
```

## Установить последнюю версии Nginx и его модулей

```bash
# Скачать и распаковать последнюю версию
wget http://nginx.org/download/nginx-1.25.4.tar.gz
tar zxvf nginx-1.26.1.tar.gz

# Скачать модуль Brotli для поддержки высокой степени сжатия статических файлов
git clone https://github.com/google/ngx_brotli.git
cd ngx_brotli && git submodule update --init && cd ..

# Скомпилировать модуль Brotli
cd ngx_brotli/deps/brotli
mkdir out && cd out
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_C_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_CXX_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_INSTALL_PREFIX=./installed ..
cmake --build . --config Release --target brotlienc

# Сконфигурировать сборку Nginx
cd ../../../../nginx-1.26.1
export CFLAGS="-m64 -march=native -mtune=native -Ofast -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections"
export LDFLAGS="-m64 -Wl,-s -Wl,-Bsymbolic -Wl,--gc-sections"

./configure --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-debug --with-compat --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --with-http_addition_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module=dynamic --with-http_sub_module --with-http_xslt_module=dynamic --with-stream=dynamic --with-stream_ssl_module --with-mail=dynamic --with-mail_ssl_module --with-http_mp4_module --add-module=../ngx_brotli
make && make install

vi /lib/systemd/system/nginx.service
```

Записать следующее содержимое в файл:

```bash
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/local/nginx/nginx -t
ExecStart=/usr/local/nginx/nginx
ExecReload=/usr/local/nginx/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

Перезапустить Nginx

```bash
sudo systemctl stop nginx
sudo systemctl daemon-reload
sudo systemctl start nginx
sudo systemctl enable nginx
```
