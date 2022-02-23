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
Повышаем права до `root` \
Или создаем ключи используя `sudo` \
Создаем ключ на клиенте: \
`ssh-keygen` \
Ключ будет создан в каталоге `/root/.ssh/` \
Копируем ключ `id-rsa.pub` на сервер в папку \
`echo 'command="/usr/bin/borg serve" ssh-rsa public_key' >> ~borg/.ssh/authorized_keys` \
Где public_key - содержимое вашего открытого ключа (то, что выделено желтым на скриншоте выше). \ 
Данная запись указывает, что при логине владельца открытого ключа запускать для него серверный процесс Borg, \
интерактивный вход в консоль для него невозможен. 

Также можно дополнительно ограничить пользователя работой только с собственным репозиторием, немного изменим команду: \
`echo 'command="/usr/bin/borg serve --restrict-to-path ~borg/my_repo",restrict ssh-rsa public_key' >> ~borg/.ssh/authorized_keys`

Инициализируем репозиторий borg на backup сервере с client сервера: \
`borg init --encryption=repokey borg@192.168.11.160:/var/Backuprepo/backup/` \
Вводим пароль `123456` \
`Enter new passphrase:` \
`Enter same passphrase again:`

#### Так же можно указать инициализацию репозитория без пароля
`borg init --encryption=none borg@192.168.11.160:/var/Backuprepo/backup/`

Запускаем создания бэкапа: \
`borg create --stats --list borg@192.168.11.160:/var/Backuprepo/backup/::"etc-{now:%Y-%m-%d_%H:%M:%S}" /etc` \
Вводим пароль созданый при инициализации репозитория \
Проверяем создания бекапа: \
`borg list borg@192.168.11.160:/var/Backuprepo/backup/`
![]()

Смотрим список файлов: \
`borg list borg@192.168.11.160:/var/Backuprepo/backup/::etc-2022-02-20_12:20:30`

Достаем файл из бекапа: \
`borg extract borg@192.168.11.160:/var/backup/::etc-2022-02-20_12:20:30 etc/hostname`

![]()

Автоматизируем создание бэкапов с помощью systemd \
Создаем сервис и таймер в каталоге `/etc/systemd/system/` \
[file]()






