# BorgBackup
Настройка BorgBackup server client, а так же автоматизация.
## Что такое BorgBackup
BorgBackup это дедуплицирующая программа для резервного копирования. \
Опционально доступно сжатие и шифрование данных. \
Основная задача BorgBackup — предоставление эффективного и безопасного решения для резервного копирования. \
Благодаря дедупликации резервное копирование происходит очень быстро. \
Все данные можно зашифровать на стороне клиента, что делает Borg интересным для использования на арендованных хранилищах.

### Как это работает
Разворачиваем стенд: \
`vagrant up` \
Конфигурим сервер: \
`ansible-playbook configure-backup.yml` \
Конфигурим клиента: \
`ansible-playbook configure-client.yml`

Для конфигурирования используются роли `ansible`. \
В ролях задействованы шаблоны `Jinja` и скрипты `bash`
Для боле гибкой настройки `ssh` используются шаблоны `J2` - опционально.

После запуска плейбуков будет сконфигурировано: \
На сервере:
- Примантирован диск `2G` в папку `/var/Backuprepo`
- Добвавлена папка `/var/Backuprepo/backup` (если примантировать диск сразу в дирокторию, то репозиторий для BorgBackup не создается, т.к появляется служебная директория `lost+found`  BorgBackup требует чистой директории)
- Создан пользователь `borg` без доступа входа по паролю, только по ключу.
- Установлен BorgBackup

На клиенте:
- Установлен BorgBackup
- Сформирован `ssh` ключ. (который будет скопирован в домашнею директорю пользователя `borg` `/home/borg/.ssh/authorized_keys`

### Дальнейшее выполнение

#### Заходим на клиента:
`vagrant ssh backup-client` \
Создаем ключ на клиенте: \
`ssh-keygen` \
Копируем ключ `id-rsa.pub` на сервер в папку \
`/home/borg/.ssh/authorized_keys`

Инициализируем репозиторий borg на backup сервере с client сервера: \
`borg init --encryption=repokey borg@192.168.11.160:/var/Backuprepo/backup/` \
Запускаем создания бэкапа: \
`borg create --stats --list borg@192.168.11.160:/var/Backuprepo/backup/::"etc-{now:%Y-%m-%d_%H:%M:%S}" /etc` \
Проверяем создания бекапа: \
`borg list borg@192.168.11.160:/var/Backuprepo/backup/`




