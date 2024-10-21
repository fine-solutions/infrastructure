# Настройка модуля `image_filter`

## Настройка автоматического изменения размеров картинок

- Создать папку для кеширования картинок с помощью `Nginx`

  ```bash
  sudo mkdir -p /var/nginx/cache
  sudo chown -R nginx:nginx /var/nginx/cache/
  sudo chcon -v --type=var_t /var/nginx/cache/
  ```

- Добавить каталог для кеширования картинок в файл `nginx.conf`

  ```bash
  proxy_cache_path /var/nginx/cache levels=1:2 keys_zone=image_cache:10m inactive=24h max_size=5G;
  ```

- Добавить конфигурацию обратного прокси для кеширования картинок в основной раздел `server` конфигурации для сайта

  ```bash
  location ~ (?<file>.+)-(?<width>350|650|1250|1850|2250)w\.(?<format>png|jpg|webp)$ {                                                
    proxy_pass              http://localhost:3000;
    proxy_cache             image_cache;
    proxy_cache_lock        on;
    proxy_cache_valid       200 24h;
    proxy_cache_valid       404 415 1m;
  }
  ```

- Добавить директиву `server` для организации кеширования на `localhost:3000`

  ```bash
  server {
    listen  3000;
    root    /web/sites/fine-solutions.org/www;
    
    server_name _;
  
    location ~ (?<file>.+)-(?<width>\d+)w\.(?<format>png|jpg|webp)$ {
      add_header                      X-debug-message '$file - $width - $format' always;
      image_filter_jpeg_quality       80;
      image_filter_webp_quality       70;
      image_filter                    resize $width -;
    
      alias   /web/sites/fine-solutions.org/www$file.$format;                                                                     
      error_page      415 = /empty;
    }
    
    location / {
      try_files       $uri    $uri/ =404;
    }
    
    location = /empty {
      empty_gif;
    }
  }
  ```

- Перезагрузить Nginx

  ```bash
  sudo systemctl reload nginx
  ```
