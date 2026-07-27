# Домашнее задание: Стенд для работы с NFSv3

Реализация лабораторной работы по настройке сетевой файловой системы NFSv3 с автоматизацией конфигурации через Bash-скрипты.

---

### Текст задания
1. Запустить 2 виртуальные машины (сервер NFS и клиента).
2. На сервере NFS подготовить и экспортировать директорию.
3. В экспортированной директории создать поддиректорию с именем `upload` с правами на запись в неё.
4. Обеспечить автоматическое монтирование экспортированной директории на клиенте при старте виртуальной машины (`systemd.automount`).
5. Организовать монтирование и работу NFS на клиенте с использованием протокола NFSv3.

---

### Структура NFS.md
*  документация и описание решения.
* `nfss_script.sh` — Bash-скрипт для автоматической настройки NFS-сервера (`nfss`).
* `nfsc_script.sh` — Bash-скрипт для автоматической настройки NFS-клиента (`nfsc`).

---

### Сетевая конфигурация стенда
* **NFS-сервер (nfss):** IP-адрес `10.0.100.118`
* **NFS-клиент (nfsc):** IP-адрес `10.0.100.213`

---

### Инструкция по развертыванию (Автоматизация)

#### 1. Настройка сервера (`nfss`)
Перенесите скрипт `nfss_script.sh` на серверную машину, сделайте его исполняемым и запустите от имени суперпользователя:
```bash
chmod +x nfss_script.sh
sudo ./nfss_script.sh
```

#### 2. Настройка клиента (`nfsc`)
Перенесите скрипт `nfsc_script.sh` на клиентскую машину, сделайте его исполняемым и запустите от имени суперпользователя:
```bash
chmod +x nfsc_script.sh
sudo ./nfsc_script.sh
```

---

### Особенности проектирования и реализации решения

1. **Версионирование протокола (NFSv3):** 
   В соответствии с требованиями задания, в конфигурационной строке файла `/etc/fstab` клиента жестко зафиксирована версия протокола с помощью параметра `vers=3`. Это принудительно отключает использование NFSv4.

2. **Автомонтирование через `systemd.automount`:**
   Вместо классического статического монтирования при загрузке системы, в `/etc/fstab` объявлены опции `noauto,x-systemd.automount`. Это заставляет systemd динамически генерировать юнит автомонтирования.
   Сетевой ресурс монтируется в каталог `/mnt` «на лету» — ровно в момент первого обращения любого процесса или пользователя к этой директории. Это предотвращает зависание ОС клиента при загрузке, если сервер NFS временно недоступен.

4. **Разграничение прав и безопасность:**
   * На сервере в `/etc/exports` доступ к общей папке ограничен строго IP-адресом клиента (`10.0.100.213/32`).
   * Включена опция `root_squash`, которая со стороны сервера преобразует запросы от локального `root` клиента в анонимного пользователя `nobody`.
   * Корень экспортируемой директории `/srv/share` защищен от записи (права `0755`), в то время как целевой подкаталог `/srv/share/upload` полностью открыт для записи клиенту (права `0777`), что полностью удовлетворяет условию задачи.

---

### Чек-лист проверки работоспособности

После запуска скриптов и перезагрузки обеих виртуальных машин выполняются следующие шаги для подтверждения корректности настройки.

#### 1. Проверка экспорта на сервере (`nfss`)
Команда `exportfs -s` позволяет убедиться, что директория экспортирована с правильными опциями безопасности для конкретного IP-адреса клиента:
```bash
sudo exportfs -s
```
**Ожидаемый вывод:**
```text
/srv/share  10.0.100.213/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```

#### 2. Проверка RPC-соединения со стороны клиента (`nfsc`)
Утилита `showmount` опрашивает сервер и показывает список активных точек монтирования:
```bash
showmount -a 10.0.100.118
```
**Ожидаемый вывод:**
```text
All mounts on 10.0.100.118:
10.0.100.213:/srv/share
```

#### 3. Проверка триггера автомонтирования и версии протокола
При переходе в каталог `/mnt` система `systemd.automount` автоматически монтирует удаленный ресурс. Проверяем это командой `mount`:
```bash
cd /mnt/upload
mount | grep mnt
```
**Ожидаемый вывод:**
```text
systemd-1 on /mnt type autofs (rw,relatime,fd=73,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=44232)
10.0.100.118:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=1048576,wsize=1048576,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=10.0.100.118,mountvers=3,mountport=49071,mountproto=udp,local_lock=none,addr=10.0.100.118)
```
*(Обратите внимание на флаг **vers=3**, подтверждающий строгое использование NFSv3).*

#### 4. Тест прав и сквозной записи
Проверяем защиту корня от записи и доступность папки `upload`:
```bash
# Попытка записи в корень (ожидается отказ в доступе из-за root_squash и прав 0755)
touch /mnt/test.txt
```
**Ожидаемый вывод:**
```text
touch: cannot touch '/mnt/test.txt': Permission denied
```

```bash
# Успешная запись в целевой каталог upload
touch /mnt/upload/final_check_file.txt
ls -l /mnt/upload/final_check_file.txt
```
**Ожидаемый вывод:**
```text
-rw-r--r-- 1 nobody nogroup 0 Jul 27 11:15 /mnt/upload/final_check_file.txt
```
*(Файл успешно создан, а владелец `nobody:nogroup` доказывает правильную работу механизма маскировки прав `root_squash`).*

### nfss_script.sh
```bash
#!/bin/bash
set -e

echo "=== Настройка NFS-сервера (nfss) ==="

# 1. Установка пакета
sudo apt-get update && sudo apt-get install -y nfs-kernel-server

# 2. Создание директорий и выставление прав
sudo mkdir -p /srv/share/upload
sudo chown -R nobody:nogroup /srv/share
sudo chmod 0755 /srv/share
sudo chmod 0777 /srv/share/upload
sudo tee /etc/exports << 'EOF'
/srv/share 10.0.100.213/32(rw,sync,root_squash)
EOF
sudo exportfs -r
sudo systemctl enable nfs-kernel-server
sudo systemctl restart nfs-kernel-server

echo "=== Проверка экспорта на сервере ==="
sudo exportfs -s
```
**Ожидаемый вывод:**
```text
/srv/share  10.0.100.213/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```
### nfsc_script.sh
```bash
#!/bin/bash
set -e

echo "=== Настройка NFS-клиента (nfsc) ==="

# 1. Установка пакета
sudo apt-get update && sudo apt-get install -y nfs-common

# 2. Очистка старой записи в fstab (если была) и добавление актуальной строки
sudo sed -i '\/srv\/share/d' /etc/fstab
echo "10.0.100.118:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0" | sudo tee -a /etc/fstab

# 3. Перезапуск конфигурации systemd для генерации automount-юнитов
sudo systemctl daemon-reload
sudo systemctl restart remote-fs.target

echo "=== Активация точки монтирования ==="
# Принудительно заходим в директорию, чтобы триггернуть systemd.automount
ls -l /mnt
mount | grep mnt
```
**Ожидаемый вывод:**
```text
systemd-1 on /mnt type autofs (rw,relatime,fd=73,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=44232)
10.0.100.118:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=1048576,wsize=1048576,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=10.0.100.118,mountvers=3,mountport=49071,mountproto=udp,local_lock=none,addr=10.0.100.118)
```
