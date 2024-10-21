# Настройка службы для автоматической конвертации картинок

- Установить сервис для отслеживания изменений ы папках

  ```bash
  sudo dnf-config-manager --enable ol9_developer_EPEL
  sudo dnf install inotify-tools
  ```

- Установить необходимые библиотеки для обрботки изображений

  ```bash
  sudo dnf -y install libjpeg libjpeg-turbo-utils libwebp libwebp-tools libavif libavif-tools ImageMagick ImageMagick-deve
  ```

- Настроить скрипт для автоматическиой конвертации картинок в формате `PNG` в другие форматы

  ```bash
  sudo touch /path/to/script/image-watcher.sh
  sudo chown root:root /path/to/script/image-watcher.sh
  sudo chmod 755 /path/to/script/image-watcher.sh
  sudo chcon -v --type=bin_t /path/to/script/image-watcher.sh
  sudo vi /path/to/script/image-watcher.sh
  ```

  ```bash
  #!/bin/bash
  WATCHING_DIR='/path/to/image/folder'
  WATCHING_FORMAT='%e %w%f'
  WEBP_QUALITY=70
  JPEG_QUALITY=60
  inotifywait --quiet --monitor --recursive --format "$WATCHING_FORMAT" \
  --event close_write --event moved_from --event moved_to --event delete \
  $WATCHING_DIR \
  | grep -i -E '\.png$' --line-buffered \
  | while read OPERATION PATH; do
    TIMESTAMP=$(/usr/bin/date +"%Y-%m-%d %H:%M:%S")
    JPEG_PATH=$(/usr/bin/sed -e 's/\.[^.]*$/.jpg/' <<< "$PATH");
    WEBP_PATH=$(/usr/bin/sed -e 's/\.[^.]*$/.webp/' <<< "$PATH");
    AVIF_PATH=$(/usr/bin/sed -e 's/\.[^.]*$/.avif/' <<< "$PATH");
    if [ $OPERATION = "MOVED_FROM" ] || [ $OPERATION = "DELETE" ]; then
      echo "File $PATH has been deleted or moved somewhere at $TIMESTAMP..."
      if [ -f "$JPEG_PATH" ]; then
        /usr/bin/rm -f "$JPEG_PATH"
        echo "  - $JPEG_PATH has been deleted"
      fi
      if [ -f "$WEBP_PATH" ]; then
        /usr/bin/rm -f "$WEBP_PATH"
        echo "  - $WEBP_PATH has been deleted"
      fi
      if [ -f "$AVIF_PATH" ]; then
        /usr/bin/rm -f "$AVIF_PATH"
        echo "  - $AVIF_PATH has been deleted"
      fi
    elif [ $OPERATION = "MOVED_TO" ] || [ $OPERATION = "CLOSE_WRITE,CLOSE" ]; then
      echo "File $PATH has been added or modified at $TIMESTAMP..."
      /usr/bin/convert -quality "$JPEG_QUALITY" "$PATH" "$JPEG_PATH"
      if [ -f "$JPEG_PATH" ]; then
        echo "  - file $JPEG_PATH has been generated"
      fi
      /usr/local/bin/cwebp -quiet -q "$WEBP_QUALITY" "$PATH" -o "$WEBP_PATH"
      if [ -f "$WEBP_PATH" ]; then
        echo "  - file $WEBP_PATH has been generated"
      fi
      /usr/bin/avifenc --min 0 --max 63 --maxalpha 63 --advanced tune=ssim --speed 0 --jobs 1 --ignore-icc "$PATH" "$AVIF_PATH"
      if [ -f "$AVIF_PATH" ]; then
        echo "  - file $AVIF_PATH has been generated"
      fi
    fi;
  done;
  ```

- Добавить скрипт как службу

  ```bash
  sudo vi /lib/systemd/system/image-watcher.service
  ```

  ```bash
  [Unit]
  Description=Watcher of Image Convertion
  
  [Service]
  ExecStart=/path/to/script/image-watcher.sh
  
  [Install]
  WantedBy=multi-user.target
  ```

- Запустить скрипт как службу

  ```bash
  sudo systemctl enable image-watcher.service
  sudo systemctl start image-watcher.service
  sudo systemctl status image-watcher.service
  ```
