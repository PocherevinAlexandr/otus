# Домашнее задание Сценарии iptables
## Цель: Написать сценарии iptables.
## Описание/Пошаговая инструкция выполнения домашнего задания:
 Что нужно сделать?

   1. реализовать knocking port
      centralRouter может попасть на ssh inetrRouter через knock скрипт (пример в материалах).

   2. добавить inetRouter2, который виден(маршрутизируется (host-only тип сети для виртуалки)) с хоста или форвардится порт через локалхост.
   3. запустить nginx на centralServer.
   4. пробросить 80й порт на inetRouter2 8080.
   5. дефолт в инет оставить через inetRouter.
   *. Реализовать проход на 80й порт без маскарадинга

Формат сдачи ДЗ - vagrant + ansible

## Как проверить работу стенда?


### Проверка 1: Port Knocking (centralRouter → inetRouter)

По условию SSH на inetRouter должен быть закрыт для centralRouter, но открываться на 10 секунд после последовательного стука по портам 7001 и 7002.
1. Зайдите на centralRouter
```console
vagrant ssh centralRouter
```
3. Попробуйте подключиться по SSH напрямую (без стука) — соединение должно повиснуть по таймауту
```console
ssh -o ConnectTimeout=3 vagrant@192.168.255.1
```
(Ожидаемый результат: Connection timed out)
5. Выполните последовательный «стук» и сразу подключитесь (пароль по умолчанию для пользователя vagrant — vagrant)
```console
timeout 1 bash -c 'echo > /dev/tcp/192.168.255.1/7001'
timeout 1 bash -c 'echo > /dev/tcp/192.168.255.1/7002'
ssh vagrant@192.168.255.1
```
(Ожидаемый результат: запрос пароля или успешный вход в консоль inetRouter).
7. Находясь внутри inetRouter, можно посмотреть на сам список ядра, куда записался ваш IP
```console
cat /proc/net/xt_recent/K2
```

### Проверка 2: Проброс порта 8080 → 80 (хост → inetRouter2 → centralServer)

По условию веб-сервер Nginx на centralServer должен открываться при обращении на порт 8080 роутера inetRouter2.
1. Узнайте внешний DHCP-адрес, который VirtualBox выдал для inetRouter2
```console
vagrant ssh inetRouter2 -c "ip -4 -br addr"
```
(Найдите интерфейс eth2 или enp0s8, обычно адрес начинается на 192.168.56.X или 172.28.128.X).
3. Сделайте HTTP-запрос прямо с терминала вашего основного компьютера (хоста) на этот IP и порт 8080
```console
curl -I http://<IP_из_шага_1>:8080
```
(Ожидаемый результат: HTTP/1.1 200 OK от сервера nginx).

### Проверка 3: Отсутствие маскарадинга и работа PBR (centralServer)

Главное условие со звездочкой: маскарадинг запрещен, поэтому Nginx должен видеть реальный IP-адрес вашего хоста, а не адрес роутера.
1. Зайдите на centralServer и проверьте последние строки лога доступа Nginx
```console
vagrant ssh centralServer -c "sudo tail -n 5 /var/log/nginx/access.log"
```
(Ожидаемый результат: в начале строки должен стоять реальный IP вашего хоста в сети VirtualBox, например 192.168.56.1, а не 10.10.2.1 или 192.168.254.1).
2. Убедитесь, что в таблице nat на inetRouter2 нет правил MASQUERADE или SNAT
```console
vagrant ssh inetRouter2 -c "sudo iptables -t nat -S"
```
(Ожидаемый результат: присутствует только правило -A PREROUTING -p tcp -m tcp --dport 8080 -j DNAT --to-destination 10.10.2.2:80).
3. Проверьте счетчики маркировки трафика на centralRouter, чтобы убедиться, что асимметричная маршрутизация отрабатывает через таблицу 100
```console
vagrant ssh centralRouter -c "sudo iptables -t mangle -nvx -L PREROUTING"
```
(Ожидаемый результат: оба счетчика pkts и bytes для правил CONNMARK больше нуля).
