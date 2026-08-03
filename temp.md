# Решение практических задач по администрированию systemd

Репозиторий содержит готовые конфигурации, скрипты и unit-файлы для настройки системных служб в Linux.

---

## Содержание
1. [Мониторинг лог-файла (Сервис и Таймер)](#1-мониторинг-лог-файла-сервис-и-таймер)
2. [Создание unit-файла для spawn-fcgi](#2-создание-unit-файла-для-spawn-fcgi)
3. [Шаблонизация Nginx для нескольких инстансов](#3-шаблонизация-nginx-для-нескольких-инстансов)

---

## 1. Мониторинг лог-файла (Сервис и Таймер)

Задача: написать службу, которая раз в 30 секунд проверяет лог-файл на наличие ключевого слова. Параметры задаются в файле окружения.

### Шаг 1. Конфигурация окружения
Создайте файл `/etc/default/log_monitor`:
```bash
LOG_FILE="/var/log/syslog"
KEYWORD="ERROR"
```

### Шаг 2. Скрипт проверки
Создайте исполняемый скрипт `/usr/local/bin/log_monitor.sh`:
```bash
#!/bin/bash

# Загрузка переменных окружения
if [ -f /etc/default/log_monitor ]; then
    . /etc/default/log_monitor
else
    echo "Ошибка: Конфигурационный файл не найден!"
    exit 1
fi

# Проверка существования лога
if [ ! -f "\$LOG_FILE" ]; then
    echo "Ошибка: Файл лога \$LOG_FILE не существует!"
    exit 1
fi

# Поиск ключевого слова и отправка уведомления в системный лог
if grep -q "KEYWORD" "LOG_FILE"; then
    logger -t log_monitor "Внимание! Ключевое слово 'KEYWORD' обнаружено в LOG_FILE"
fi
```
> ⚠️ **Важно:** Не забудьте выдать права на выполнение: `sudo chmod +x /usr/local/bin/log_monitor.sh`

### Шаг 3. Создание Unit-файлов systemd

<details>
<summary>📦 Сервис: <code>/etc/systemd/system/log_monitor.service</code> (Нажмите, чтобы раскрыть)</summary>

```ini
[Unit]
Description=Log Monitor Service
After=network.target

[Service]
Type=oneshot
EnvironmentFile=/etc/default/log_monitor
ExecStart=/usr/local/bin/log_monitor.sh

[Install]
WantedBy=multi-user.target
```
</details>

<details>
<summary>⏱️ Таймер: <code>/etc/systemd/system/log_monitor.timer</code> (Нажмите, чтобы раскрыть)</summary>

```ini
[Unit]
Description=Run Log Monitor Every 30 Seconds

[Timer]
OnBootSec=10
OnUnitActiveSec=30

[Install]
WantedBy=timers.target
```
</details>

### Шаг 4. Запуск и проверка
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now log_monitor.timer
```
Проверить статус таймера можно командой: `systemctl status log_monitor.timer`

---

## 2. Создание unit-файла для spawn-fcgi

Задача: переписать старый init-скрипт под современный `systemd`-сервис.

### Шаг 1. Файл параметров окружения
Создайте `/etc/default/spawn-fcgi`:
```ini
FCGIPROGRAM="/usr/bin/php-cgi"
FCGIADDRESS="127.0.0.1"
FCGIPORT="9000"
FCGI_USER="www-data"
FCGI_GROUP="www-data"
OPTIONS="-u \$FCGI_USER -g \(FCGI_GROUP -a\)FCGIADDRESS -p FCGIPORT -f FCGIPROGRAM"
```

### Шаг 2. Unit-файл сервиса
Создайте `/etc/systemd/system/spawn-fcgi.service`:
```ini
[Unit]
Description=Spawn-fcgi Service
After=network.target

[Service]
Type=forking
EnvironmentFile=/etc/default/spawn-fcgi
ExecStart=/usr/bin/spawn-fcgi \$OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target
```

### Шаг 3. Запуск
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now spawn-fcgi.service
```

---

## 3. Шаблонизация Nginx для нескольких инстансов

Задача: доработать unit-файл Nginx для одновременного запуска нескольких независимых серверов с разными конфигурационными файлами.

### Шаг 1. Создание шаблонного Unit-файла
Создайте файл `/etc/systemd/system/nginx@.service` (символ `@` указывает на то, что это шаблон):
```ini
[Unit]
Description=The NGINX HTTP and reverse proxy server (Instance %I)
After=syslog.target network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx-%i.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-%i.conf -g 'pid /run/nginx-%i.pid;'
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-%i.conf -g 'pid /run/nginx-%i.pid;'
ExecReload=/usr/sbin/nginx -c /etc/nginx/nginx-%i.conf -g 'pid /run/nginx-%i.pid;' -s reload
ExecStop=/bin/kill -s QUIT \$MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

### Шаг 2. Использование и развертывание инстансов

1. Создайте индивидуальные файлы конфигураций для каждого инстанса. Имя файла должно строго соответствовать шаблону `nginx-{имя}.conf`.
   * `/etc/nginx/nginx-first.conf`
   * `/etc/nginx/nginx-second.conf`
   
   > 💡 *Убедитесь, что внутри конфигураций заданы разные порты (директива `listen`) и разные пути к лог-файлам во избежание конфликтов портов и блокировок.*

2. Примените изменения в systemd:
   ```bash
   sudo systemctl daemon-reload
   ```

3. Запустите и добавьте в автозагрузку конкретные инстансы (вместо `%i` systemd автоматически подставит значения `first` и `second`):
   ```bash
   sudo systemctl enable --now nginx@first
   sudo systemctl enable --now nginx@second
   ```
