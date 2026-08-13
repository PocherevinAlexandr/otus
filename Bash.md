# Урок 10 BASH
Домашнее задание

Пишем скрипт
Цель:

написать bash-скрипт, который ежечасно формирует и отправляет на email отчёт о работе веб-сервера;

Описание/Пошаговая инструкция выполнения домашнего задания:

🎯Что нужно сделать?

Написать скрипт для CRON, который раз в час формирует отчёт и отправляет его на заданную почту.


Отчёт должен содержать:

    IP-адреса с наибольшим числом запросов (с момента последнего запуска);
    Запрашиваемые URL с наибольшим числом запросов (с момента последнего запуска);
    Ошибки веб-сервера/приложения (с момента последнего запуска);
    HTTP-коды ответов с указанием их количества (с момента последнего запуска).


Скрипт должен предотвращать одновременный запуск нескольких копий, до его завершения.

В письме должен быть прописан обрабатываемый временной диапазон.
### скрипт

```bash
#!/bin/bash
export LC_TIME=C
export TZ="Europe/Samara"
LOG_FILE="access.log"
EMAIL="sas*****in@mail.ru"
SUBJECT="Ежечасный отчет о работе веб-сервера"
LOCK_FILE="web_report.pid"
TMP_LOG=$(mktemp /tmp/web_report_XXXXXX.log)
exec 200>"$LOCK_FILE"
if ! flock -n 200; then
    echo "Ошибка: Скрипт уже запущен в другом процессе!" >&2
    exit 1
fi
echo $$ >&200
cleanup() {
    rm -f "$TMP_LOG"
}
trap cleanup EXIT INT TERM
START_TIME="14/Aug/2019:04"
END_TIME=$(date +"%d/%b/%Y:%H")
sed -n "\|$START_TIME:|,"'$p' "$LOG_FILE" > "$TMP_LOG"
if [ ! -s "$TMP_LOG" ]; then
    REPORT_BODY="За период с $START_TIME:00 по $END_TIME:00 запросов к веб-серверу не зафиксировано."
    echo "$REPORT_BODY" | mail -s "$SUBJECT" -a "From: sas*****in@mail.ru" "$EMAIL"
    exit 0
fi
TOP_IPS=$(awk '{print $1}' "$TMP_LOG" | sort | uniq -c | sort -rn | head -n 10)
TOP_URLS=$(awk '{print $7}' "$TMP_LOG" | sort | uniq -c | sort -rn | head -n 10)
ERRORS=$(awk '$9 ~ /^[45]/ {print $0}' "$TMP_LOG" | head -n 20)
HTTP_CODES=$(awk '{print $9}' "$TMP_LOG" | sort | uniq -c | sort -rn)
{
    echo "========================================================="
    echo " ЕЖЕЧАСНЫЙ ОТЧЕТ О РАБОТЕ ВЕБ-СЕРВЕРА"
    echo " Обрабатываемый временной диапазон: с $START_TIME:00 по $END_TIME:00"
    echo "========================================================="
    echo ""
    echo "--- ТОП-10 IP-адресов по количеству запросов ---"
    echo " Кол-во  | IP-адрес"
    echo "$TOP_IPS"
    echo ""
    echo "--- ТОП-10 запрашиваемых URL ---"
    echo " Кол-во  | URL"
    echo "$TOP_URLS"
    echo ""
    echo "--- Сводка по HTTP-кодам ответов ---"
    echo " Кол-во  | Код"
    echo "$HTTP_CODES"
    echo ""
    echo "--- Лог ошибок (4xx и 5xx, первые 20 строк) ---"
    if [ -z "$ERRORS" ]; then
        echo " Ошибок не обнаружено."
    else
        echo "$ERRORS"
    fi
    echo ""
    echo "========================================================="
} > mail_body.txt
mail -s "$SUBJECT" -a "From: sas*****in@mail.ru" "$EMAIL" < mail_body.txt
rm -f mail_body.txt
```
>[!NOTE]
>START_TIME="14/Aug/2019:04" жёстко задан для срабатывания на конкретном лог файле, из-за того что он статичный и старый
>отчёт отправляет один раз с информацией, а в дальнейшем присылает отчёт что "За период с 14/Aug/2019:04:00 по 12/авг/2026:21:00 запросов к веб-серверу не зафиксировано."

>[!WARNING]
>для нормальной работы скрипта в реальном времени нужно задать START_TIME=$(date -d "1 hour ago" +"%d/%b/%Y:%H")
>sed -n "\|$START_TIME:|,\$p" "$LOG_FILE" > "$TMP_LOG"
> и в начале export LC_TIME=C  export TZ="Europe/Samara"

### Письмо отчёт
```text
=========================================================
ЕЖЕЧАСНЫЙ ОТЧЕТ О РАБОТЕ ВЕБ-СЕРВЕРА
Обрабатываемый временной диапазон: с 14/Aug/2019:04:00 по 10/авг/2026:12:00
=========================================================

--- ТОП-10 IP-адресов по количеству запросов ---
Кол-во | IP-адрес
45 93.158.167.130
39 109.236.252.130
37 212.57.117.19
33 188.43.241.106
31 87.250.233.68
24 62.75.198.172
22 148.251.223.21
20 185.6.8.9
17 217.118.66.161
16 95.165.18.146

--- ТОП-10 запрашиваемых URL ---
Кол-во | URL
157 /
120 /wp-login.php
57 /xmlrpc.php
26 /robots.txt
12 /favicon.ico
11 400
9 /wp-includes/js/wp-embed.min.js?ver=5.0.4
7 /wp-admin/admin-post.php?page=301bulkoptions
7 /1
6 /wp-content/uploads/2016/10/robo5.jpg

--- Сводка по HTTP-кодам ответов ---
Кол-во | Код
498 200
95 301
51 404
11 "-"
7 400
3 500
2 499
1 405
1 403
1 304

--- Лог ошибок (4xx и 5xx, первые 20 строк) ---
93.158.167.130 - - [14/Aug/2019:05:02:20 +0300] "GET / HTTP/1.1" 404 169 "-" "Mozilla/5.0 (compatible; YandexMetrika/2.0; +http://yandex.com/bots yabs01)"rt=0.000 uct="-" uht="-" urt="-"
87.250.233.68 - - [14/Aug/2019:05:04:20 +0300] "GET / HTTP/1.1" 404 169 "-" "Mozilla/5.0 (compatible; YandexMetrika/2.0; +http://yandex.com/bots yabs01)"rt=0.000 uct="-" uht="-" urt="-"
107.179.102.58 - - [14/Aug/2019:05:22:10 +0300] "GET /wp-content/plugins/uploadify/readme.txt HTTP/1.1" 404 200 "http://dbadmins.ru/wp-content/plugins/uploadify/readme.txt" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.152 Safari/537.36"rt=0.000 uct="-" uht="-" urt="-"
193.106.30.99 - - [14/Aug/2019:06:02:50 +0300] "GET /wp-includes/ID3/comay.php HTTP/1.1" 500 595 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.103 Safari/537.36"rt=0.000 uct="-" uht="-" urt="-"
87.250.244.2 - - [14/Aug/2019:06:07:07 +0300] "GET / HTTP/1.1" 404 169 "-" "Mozilla/5.0 (compatible; YandexMetrika/2.0; +http://yandex.com/bots yabs01)"rt=0.000 uct="-" uht="-" urt="-"
77.247.110.165 - - [14/Aug/2019:06:13:53 +0300] "HEAD /robots.txt HTTP/1.0" 404 0 "-" "-"rt=0.018 uct="-" uht="-" urt="-"
87.250.233.76 - - [14/Aug/2019:06:45:20 +0300] "GET / HTTP/1.1" 404 169 "-" "Mozilla/5.0 (compatible; YandexMetrika/2.0; +http://yandex.com/bots yabs01)"rt=0.000 uct="-" uht="-" urt="-"
71.6.199.23 - - [14/Aug/2019:07:07:19 +0300] "GET /robots.txt HTTP/1.1" 404 3652 "-" "-"rt=0.000 uct="-" uht="-" urt="-"
71.6.199.23 - - [14/Aug/2019:07:07:20 +0300] "GET /sitemap.xml HTTP/1.1" 404 3652 "-" "-"rt=0.000 uct="-" uht="-" urt="-"
71.6.199.23 - - [14/Aug/2019:07:07:20 +0300] "GET /.well-known/security.txt HTTP/1.1" 404 3652 "-" "-"rt=0.000 uct="-" uht="-" urt="-"
71.6.199.23 - - [14/Aug/2019:07:07:21 +0300] "GET /favicon.ico HTTP/1.1" 404 3652 "-" "python-requests/2.19.1"rt=0.000 uct="-" uht="-" urt="-"
141.8.141.136 - - [14/Aug/2019:07:09:43 +0300] "GET / HTTP/1.1" 404 169 "-" "Mozilla/5.0 (compatible; YandexMetrika/2.0; +http://yandex.com/bots yabs01)"rt=0.000 uct="-" uht="-" urt="-"
93.158.167.130 - - [14/Aug/2019:08:10:56 +0300] "GET / HTTP/1.1" 404 169 "-" "Mozilla/5.0 (compatible; YandexMetrika/2.0; +http://yandex.com/bots yabs01)"rt=0.000 uct="-" uht="-" urt="-"
87.250.233.68 - - [14/Aug/2019:08:21:48 +0300] "GET / HTTP/1.1" 404 169 "-" "Mozilla/5.0 (compatible; YandexMetrika/2.0; +http://yandex.com/bots yabs01)"rt=0.000 uct="-" uht="-" urt="-"
62.75.198.172 - - [14/Aug/2019:08:23:40 +0300] "POST /wp-cron.php?doing_wp_cron=1565760219.4257180690765380859375 HTTP/1.1" 499 0 "https://dbadmins.ru/wp-cron.php?doing_wp_cron=1565760219.4257180690765380859375" "WordPress/5.0.4; https://dbadmins.ru"rt=1.001 uct="-" uht="-" urt="-"
78.39.67.210 - - [14/Aug/2019:08:23:41 +0300] "GET /admin/config.php HTTP/1.1" 404 29500 "-" "curl/7.15.5 (x86_64-redhat-linux-gnu) libcurl/7.15.5 OpenSSL/0.9.8b zlib/1.2.3 libidn/0.6.5"rt=0.480 uct="0.000" uht="0.192" urt="0.243"
176.9.56.104 - - [14/Aug/2019:08:30:17 +0300] "GET /1 HTTP/1.1" 404 29513 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:64.0) Gecko/20100101 Firefox/64.0"rt=0.233 uct="0.000" uht="0.182" urt="0.233"
87.250.233.75 - - [14/Aug/2019:09:21:46 +0300] "GET / HTTP/1.1" 404 169 "-" "Mozilla/5.0 (compatible; YandexMetrika/2.0; +http://yandex.com/bots yabs01)"rt=0.000 uct="-" uht="-" urt="-"
162.243.13.195 - - [14/Aug/2019:09:31:47 +0300] "POST /wp-admin/admin-ajax.php?page=301bulkoptions HTTP/1.1" 400 11 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.143 Safari/537.36"rt=0.241 uct="0.000" uht="0.241" urt="0.241"
162.243.13.195 - - [14/Aug/2019:09:31:48 +0300] "GET /1 HTTP/1.1" 404 29500 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:64.0) Gecko/20100101 Firefox/64.0"rt=0.308 uct="0.000" uht="0.187" urt="0.237"

=========================================================
```
<img width="901" height="251" alt="изображение" src="https://github.com/user-attachments/assets/d617ffa6-425c-4e87-894d-92ab55b5d42c" />
