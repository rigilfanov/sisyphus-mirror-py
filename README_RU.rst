##################
Sisyphus Mirror Py
##################
`English <README.rst>`_ | Русский

Современная Python-реализация инструмента для зеркалирования репозитория `Сизиф <https://packages.altlinux.org>`_.

Проект вдохновлён оригинальной утилитой `sisyphus-mirror <https://git.altlinux.org/gears/s/sisyphus-mirror.git>`_.

Технические требования
======================
* ОС GNU/Linux  
* Python 3.12+  
* RSync 3.2+  
* 110+ ГБ свободного места  

Установка
=========
.. code-block:: bash

  # Создание системного пользователя и директорий:
  useradd -rMU -d /opt/sisyphus-mirror -s /usr/sbin/nologin sisyphus-mirror
  mkdir -p /etc/sisyphus-mirror /opt/sisyphus-mirror /srv/mirrors/altlinux
  chown sisyphus-mirror:sisyphus-mirror /opt/sisyphus-mirror
  chown sisyphus-mirror:sisyphus-mirror /srv/mirrors/altlinux

  # Виртуальное окружение Python и установка пакета:
  sudo -u sisyphus-mirror python3 -m venv /opt/sisyphus-mirror/venv
  sudo -u sisyphus-mirror /opt/sisyphus-mirror/venv/bin/pip install sisyphus-mirror

  # Создание конфигурационного файла TOML:
  echo '[sisyphus-mirror]
  dry_run = false
  verbose = false
  debug = false

  # Список веток репозитория.
  branch_list = ["p11"]

  # URL источника репозитория.
  source_url = "rsync://ftp.altlinux.org/ALTLinux"

  # Рабочая директория для снимков зеркала и временных файлов.
  working_dir = "/srv/mirrors/altlinux"

  # Целевые архитектуры процессора.
  arch_list = ["noarch", "x86_64", "x86_64-i586"]

  # Дополнительные шаблоны включения файлов.
  include_files = ["list/**", ".timestamp"]

  # Дополнительные шаблоны исключения файлов.
  exclude_files = ["*debuginfo*", "SRPMS"]

  # Максимальное количество снимков на ветку.
  snapshot_limit = 1

  # Ограничение пропускной способности сетевого ввода-вывода.
  rate_limit = "5m"

  # Таймаут установки соединения (seconds).
  conn_timeout = 60

  # Таймаут операций ввода-вывода (в секундах).
  io_timeout = 600' > /etc/sisyphus-mirror/sisyphus-mirror.toml

Редактирование конфигурации:

.. code-block:: bash

  nano /etc/sisyphus-mirror/sisyphus-mirror.toml

Тестовый запуск
===============
Пробный запуск без записи данных:

.. code-block:: bash

  sudo -u sisyphus-mirror \
    /opt/sisyphus-mirror/venv/bin/sisyphus-mirror \
    --dry-run \
    --verbose \
    --config /etc/sisyphus-mirror/sisyphus-mirror.toml

Итоговое значение `total size` должно быть меньше доступного дискового пространства.

Рабочий запуск
==============
Выполнение синхронизации:

.. code-block:: bash

  sudo -u sisyphus-mirror \
    /opt/sisyphus-mirror/venv/bin/sisyphus-mirror \
    --config /etc/sisyphus-mirror/sisyphus-mirror.toml

Интеграция с systemd
====================
.. code-block:: bash

  # Служба systemd:
  echo '[Unit]
  Description=sisyphus-mirror service
  Wants=sisyphus-mirror.timer

  [Service]
  Type=oneshot
  User=sisyphus-mirror
  Group=sisyphus-mirror
  WorkingDirectory=/opt/sisyphus-mirror
  ExecStart=/opt/sisyphus-mirror/venv/bin/sisyphus-mirror -c /etc/sisyphus-mirror/sisyphus-mirror.toml
  ProtectHome=true
  ProtectSystem=true
  SyslogIdentifier=sisyphus-mirror

  [Install]
  WantedBy=multi-user.target' > /etc/systemd/system/sisyphus-mirror.service

  # Таймер ежедневного запуска:
  echo '[Unit]
  Description=Periodic run of sisyphus-mirror

  [Timer]
  # Запуск в случайное время с 00:15 до 07:45 по локальному времени сервера
  OnCalendar=*-*-* 00:15:00
  RandomizedDelaySec=7h 30min
  Persistent=true
  Unit=sisyphus-mirror.service

  [Install]
  WantedBy=timers.target' > /etc/systemd/system/sisyphus-mirror.timer

  # Перезагрузка конфигурации systemd и активация таймера;
  systemctl daemon-reload
  systemctl enable --now sisyphus-mirror.timer

Раздача через rsyncd
====================
.. code-block:: bash

  echo 'port = 873
  uid = 65534
  gid = 65534
  use chroot = yes
  max connections = 30

  [altlinux]
  path = /srv/mirrors/altlinux
  read only = yes
  list = yes' > /etc/rsyncd.conf

  systemctl enable --now rsync.service
