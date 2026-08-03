# Домашнее задание Работа с загрузчиком

🎯Что нужно сделать?

    Включить отображение меню Grub.
    Попасть в систему без пароля несколькими способами.
    Установить систему с LVM, после чего переименовать VG.
```bash
nano /etc/default/grub
```
Комментируем строку, скрывающую меню и ставим задержку для выбора пункта меню в 10 секунд.
```console
#GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=10
```
```bash
update-grub
reboot
```
<img width="788" height="561" alt="изображение" src="https://github.com/user-attachments/assets/2e1b6d6d-b5f1-4e26-b225-92a8efc0aaf1" />

## Попасть в систему без пароля способ 1 
при выборе ядра для загрузки нажать e. Попадаем в окно, где мы можем изменить параметры загрузки:

<img width="805" height="596" alt="изображение" src="https://github.com/user-attachments/assets/a3909726-00e0-41af-8e5f-1b1df4c9547b" />

В конце строки, начинающейся с linux, добавляем init=/bin/bash и нажимаем
f10
>[!NOTE]
>Рутовая файловая система при этом монтируется в режиме Read-Only перемонтируем в режим Read-Write

```bash
mount -o remount,rw /
```
 проверяем что можем писать 
```bash
nano 123
```
ctrl+o ctrl+x

<img width="1068" height="171" alt="изображение" src="https://github.com/user-attachments/assets/b9031de1-f1fb-4464-8adb-af75ab918f3b" />

## Попасть в систему без пароля способ 2
В меню загрузчика на первом уровне выбрать второй пункт (Advanced options…), далее загрузить пункт меню с указанием recovery mode в названии. 
Получим меню режима восстановления.

<img width="788" height="561" alt="изображение" src="https://github.com/user-attachments/assets/2e1b6d6d-b5f1-4e26-b225-92a8efc0aaf1" />
<img width="786" height="535" alt="изображение" src="https://github.com/user-attachments/assets/21a7ac00-db60-4c15-858f-41ff97467cb7" />

>[!NOTE]
>В этом меню сначала включаем поддержку сети (network) для того, чтобы файловая система перемонтировалась в режим read/write.
>Далее выбираем пункт root и попадаем в консоль с пользователем root. Если вы ранее устанавливали пароль для пользователя root (по умолчанию его нет), то необходимо его ввести. 

Enable networking

<img width="718" height="404" alt="изображение" src="https://github.com/user-attachments/assets/a4e76ca8-1dd4-4ec8-8806-ef695e2ee94c" />

## 3. Установка системы с LVM, с последующим переименованием VG
>[!NOTE]
>Мы установили систему Ubuntu со стандартной разбивкой диска с использованием  LVM.
>Первым делом посмотрим текущее состояние системы (список Volume Group)
>Далее переименовываем

```bash
vgs
vgrename ubuntu-vg ubuntu-otus
```
<img width="664" height="119" alt="изображение" src="https://github.com/user-attachments/assets/80695733-c1e2-45ec-b645-ab39a6a9bd53" />

>[!NOTE]
>Далее правим /boot/grub/grub.cfg. Везде заменяем старое название VG на новое (в файле дефис меняется на два дефиса ubuntu--vg ubuntu--otus).
>После чего можем перезагружаться и, если все сделано правильно, успешно грузимся с новым именем Volume Group и проверяем:

<img width="672" height="655" alt="изображение" src="https://github.com/user-attachments/assets/e0bb8541-6c79-4fde-b44b-f04da7cd2e61" />
