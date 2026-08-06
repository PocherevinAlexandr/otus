# Домашнее задание Systemd - создание unit-файла
1. Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).
2. Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020).
3. Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.

---

## 1. Мониторинг лог-файла (Сервис и Таймер)

Задача: написать службу, которая раз в 30 секунд проверяет лог-файл на наличие ключевого слова. Параметры задаются в файле окружения.

### Шаг 1. Конфигурация окружения
Создаём файл `sudo nano /etc/default/log_monitor`:
```bash
LOG_FILE="/var/log/syslog"
KEYWORD="CRON"
```

### Шаг 2. Скрипт проверки
Создаём исполняемый скрипт `sudo nano /usr/local/bin/log_monitor.sh`:
```bash
#!/bin/bash

# Загрузка переменных окружения
if [ -f /etc/default/log_monitor ]; then
    source /etc/default/log_monitor
else
    echo "Ошибка: Конфигурационный файл не найден!"
    exit 1
fi

# Проверка существования лога
if [ ! -f "$LOG_FILE" ]; then
    echo "Ошибка: Файл лога $LOG_FILE не существует!"
    exit 1
fi

# Поиск ключевого слова и отправка уведомления в системный лог
if grep -q "$KEYWORD" "$LOG_FILE"; then
    logger -t log_monitor "Внимание! Ключевое слово 'KEYWORD' обнаружено в LOG_FILE"
fi
```
> ⚠️ **Важно:** Не забыть выдать права на выполнение: `sudo chmod +x /usr/local/bin/log_monitor.sh`

<img width="950" height="261" alt="изображение" src="https://github.com/user-attachments/assets/69d77bd2-17ca-4a1c-9054-9683e2be5d19" />


### Шаг 3. Создание Unit-файлов systemd

<details>
<summary>📦 Сервис: <code>sudo nano /etc/systemd/system/log_monitor.service</code> (Нажмите, чтобы раскрыть)</summary>

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
<summary>⏱️ Таймер: <code>sudo nano /etc/systemd/system/log_monitor.timer</code> (Нажмите, чтобы раскрыть)</summary>

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

<img width="946" height="223" alt="изображение" src="https://github.com/user-attachments/assets/e8f08386-e070-46ba-b73d-fe489b95873a" />


---

## 2. Создание unit-файла для spawn-fcgi

Задача: Установить spawn-fcgi и создать unit-файл, переписать старый init-скрипт под современный `systemd`-сервис.
Сам Init скрипт, который будем переписывать, можно найти здесь: https://gist.github.com/cea2k/1318020 

### Шаг 1. Файл параметров окружения
Создаём `sudo nano /etc/default/spawn-fcgi`:
```ini
FCGIPROGRAM="/usr/bin/php-cgi"
FCGIADDRESS="127.0.0.1"
FCGIPORT="9000"
FCGI_USER="www-data"
FCGI_GROUP="www-data"
OPTIONS="-u $FCGI_USER -g $FCGI_GROUP -a $FCGIADDRESS -p $FCGIPORT -f $FCGIPROGRAM"
```

### Шаг 2. Unit-файл сервиса
Создаём `sudo nano /etc/systemd/system/spawn-fcgi.service`:
```ini
[Unit]
Description=Spawn-fcgi Service
After=network.target

[Service]
Type=forking
EnvironmentFile=/etc/default/spawn-fcgi
ExecStart=/usr/bin/spawn-fcgi $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target
```

### Шаг 3. Запуск
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now spawn-fcgi.service
```

>  ⚠️  Если возникает ошибка запуска то необходимо установить sudo apt update && sudo apt install -y spawn-fcgi php-cgi
---

Проверяем
<img width="1421" height="288" alt="изображение" src="https://github.com/user-attachments/assets/4b8d6ca0-edf4-4d3d-9926-13924043121d" />
<img width="866" height="116" alt="изображение" src="https://github.com/user-attachments/assets/746abb49-bb48-4f7c-86eb-443371c4494f" />


## 3. Шаблонизация Nginx для нескольких инстансов

Задача: доработать unit-файл Nginx для одновременного запуска нескольких независимых серверов с разными конфигурационными файлами.

### Шаг 1. Создание шаблонного Unit-файла
Создаём файл `sudo nano /etc/systemd/system/nginx@.service` (символ `@` указывает на то, что это шаблон):
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
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

### Шаг 2. Использование и развертывание инстансов

1. Создаём индивидуальные файлы конфигураций для каждого инстанса. Имя файла должно строго соответствовать шаблону `nginx-{имя}.conf`.
   
<details>
<summary>  <code>sudo nano /etc/nginx/nginx-first.conf</code>(Нажмите, чтобы раскрыть) </summary>
    
```ini
events {
    worker_connections 768;
}

http {
    # Свои изолированные логи
    access_log /var/log/nginx/first-access.log;
    error_log /var/log/nginx/first-error.log;

    server {
        # Свой порт
        listen 8081;
        server_name localhost;

        location / {
            return 200 "Hello from the FIRST Nginx instance!\n";
            add_header Content-Type text/plain;
        }
    }
}
```

</details>

<details>
<summary>  <code> sudo nano /etc/nginx/nginx-second.conf </code> (Нажмите, чтобы раскрыть) </summary>

```ini
     events {
    worker_connections 768;
}

http {
    # Свои изолированные логи
    access_log /var/log/nginx/second-access.log;
    error_log /var/log/nginx/second-error.log;

    server {
        # Свой порт
        listen 8082;
        server_name localhost;

        location / {
            return 200 "Hello from the SECOND Nginx instance!\n";
            add_header Content-Type text/plain;
        }
    }
}
 ```

</details>

   > 💡 *Убедимся, что внутри конфигураций заданы разные порты (директива `listen`) и разные пути к лог-файлам во избежание конфликтов портов и блокировок.*

2. Применим изменения в systemd:
   ```bash
   sudo systemctl daemon-reload
   ```

3. Запустим и добавим в автозагрузку конкретные инстансы (вместо `%i` systemd автоматически подставит значения `first` и `second`):
   ```bash
   sudo systemctl enable --now nginx@first
   sudo systemctl enable --now nginx@second
   ```
   Проверяем
```bash
ps afx | grep nginx
systemctl status nginx@second
sudo ss -tlnp | grep nginx
```
<img width="1350" height="543" alt="изображение" src="https://github.com/user-attachments/assets/a61c2ac5-8483-4cb0-b94e-2707626c646a" />
