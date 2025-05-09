# Типовая настройка

- Добавить пользователя с правами администратора (на примере `user`):

  ```bash
  # RedHat based
  useradd user             # Создание пользователя
  grep user /etc/passwd    # Проверка наличия пользователя
  passwd user              # Создание пароля для пользователя
  su - user                # Проверка авторизации
  ww                       # Проверка правильности авторизации
  pwd                      # Проверка корректности домашней директории
  exit                     # Выход и переход к пользователю root
  grep wheel /etc/sudoers  # Проверка настроек для админов
  usermod -aG wheel user   # Добавление пользователя в группу админов
  grep wheel /etc/group    # Проверка состава группы админов

  # Debian based
  useradd -m -s /bin/bash user # Создание пользователя
  passwd user                  # Создание пароля для пользователя
  usermod -a -G sudo user      # Добавление пользователя в группу админов
  ```

- Добавить публичный ключ пользователя на сервер (на примере `user`):

  ```bash
  # Создание ключа на новых системах
  ssh-keygen -t ed25519

  # Создание ключа на старых системах
  ssh-keygen -t rsa -b 4096

  # Копирование ключа на сервер
  ssh-copy-id user@server_ip_address
  ```
- Настроить доступ через SSH только по ключу (на примере `user`):

  ```bash
  # Редактирование конфигурации sshd
  sudo vi /etc/ssh/sshd_config

  # Установить значение PasswordAuthentication no
  # Установить значение PermitRootLogin no

  # Перезагрузка службы
  sudo systemctl restart sshd

  # В отдельном окне терминала зайти под пользователем user для проверки
  ssh user@server_ip_address

  # Выйти командой exit и проверить отсутствие доступа для root
  ssh root@server_ip_address

  # Если что-то пошло не так, можно проверить файлы и права для них на сервере
  cat /home/user/.ssh/authorized_keys
  chmod 700 /home/user/.ssh
  chmod 600 /home/user/.ssh/authorized_keys
  restorecon -r -vv /home/user/.ssh/authorized_keys
  ```

- Добавить пользователя с правами на развёртывание сайтов и сгенерировать пару ключей для авторизации:

  ```bash
  # Создание пользователя
  useradd deploy

  # Проверка наличия пользователя
  grep deploy /etc/passwd
  
  # Зайти под пользователем
  su deploy

  # Сгенерировать пару ключей
  ssh-keygen -t rsa -b 4096
  
  # Выйти
  exit
  
  # Зайти под администратором
  sudo su

  # Перейти в домашний каталог
  cd /home/deploy/.ssh

  # Добавить в ключи авторизации
  cp id_rsa.pub authorized_keys

  # Выставить права
  chmod 600 /home/deploy/.ssh/authorized_keys
  
  # Выйти
  exit
  ```

- Обновить пакеты и установить самое необходимое:

  ```bash
  # RedHat based
  sudo dnf -y update
  sudo dnf install -y dnf-utils zip unzip curl wget git rsync mc emacs tmux nano vim

  # Debian based
  sudo apt update
  sudo apt install -y dnf-utils zip unzip curl wget git rsync mc emacs tmux nano vim
  ```
