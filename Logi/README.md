# ДЗ: Настройка центрального сервера для сбора логов

Проект разворачивает две виртуальные машины (`web` и `log`). 
Ansible запускается локально на машине `log` (`ansible_local`), что позволяет разворачивать стенд с хостовых машин на Windows без установленного Ansible.

## Как запустить:
1. Выполните команду в корне проекта:
   ```bash
   vagrant up
   ```

## Как проверить работу:
1. Зайдите на `web` сервер и сгенерируйте событие для Nginx и Auditd:
   ```bash
   vagrant ssh web
   curl localhost
   sudo touch /etc/nginx/test_audit.conf
   exit
   ```
2. Зайдите на `log` сервер и проверьте, что логи прилетели:
   ```bash
   vagrant ssh log
   ls -la /var/log/remote/web-server/
   cat /var/log/remote/web-server/nginx_access.log
   cat /var/log/remote/web-server/audispd.log
   ```
