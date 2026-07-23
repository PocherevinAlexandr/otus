# Домашнее задание: Практические навыки работы с ZFS

Цель:

научится самостоятельно устанавливать ZFS, настраивать пулы, изучить основные возможности ZFS;

Описание/Пошаговая инструкция выполнения домашнего задания:

🎯 Что нужно сделать?

    Определить алгоритм с наилучшим сжатием:
    определить, какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);
    создать 4 файловых системы, на каждой применить свой алгоритм сжатия;
    для сжатия использовать либо текстовый файл, либо группу файлов.

    Определить настройки пула.
    С помощью команды zfs import собрать pool ZFS.
    Командами zfs определить настройки:
    размер хранилища;
    тип pool;
    значение recordsize;
    какое сжатие используется;
    какая контрольная сумма используется.

    Работа со снапшотами:
    скопировать файл из удаленной директории;
    восстановить файл локально. zfs receive;
    найти зашифрованное сообщение в файле secret_message.
---

## Задание 1. Определение алгоритма с наилучшим сжатием

### Текст задания
* Определить, какие алгоритмы сжатия поддерживает ZFS (`gzip`, `zle`, `lzjb`, `lz4`).
* Создать 4 файловых системы (или пула), на каждой применить свой алгоритм сжатия.
* Для сжатия использовать текстовый лог-файл.

### Описание команд и их вывод

1. Создание 4 пулов в режиме `mirror` (зеркало) из дисков по 512 МБ:
```console
sudo zpool create otus1 mirror /dev/xvdb /dev/xvdc
sudo zpool create otus2 mirror /dev/xvde /dev/xvdf
sudo zpool create otus3 mirror /dev/xvdg /dev/xvdh
sudo zpool create otus4 mirror /dev/xvdi /dev/xvdj
```
2. Установка различных алгоритмов сжатия на корневые датасеты пулов:
```bash
sudo zfs set compression=lzjb otus1
sudo zfs set compression=lz4 otus2
sudo zfs set compression=gzip-9 otus3
sudo zfs set compression=zle otus4
```
3. Проверка установленных параметров сжатия:
```bash
zfs get all | grep compression
# Вывод:
# otus1  compression           lzjb                   local
# otus2  compression           lz4                    local
# otus3  compression           gzip-9                 local
# otus4  compression           zle                    local
```
4. Скачивание тестового текстового лога размером ~41 МБ во все пулы:
```bash
for i in {1..4}; do sudo wget -O /otus$i/book.txt https://gutenberg.org; done
```

5. Анализ коэффициента сжатия (`compressratio`):
```bash
zfs get compressratio otus1 otus2 otus3 otus4
# Вывод:
# otus1  compressratio         1.82x                  -
# otus2  compressratio         2.23x                  -
# otus3  compressratio         3.66x                  -
# otus4  compressratio         1.00x                  -
```

### Вывод по Заданию 1
Алгоритм **`gzip-9`** (на пуле `otus3`) показал наилучшую эффективность сжатия текстовых данных с коэффициентом **3.66x**. Однако для реального использования на серверах чаще выбирают lz4 из-за идеального баланса скорости и эффективности.

---

## Задание 2. Определение настроек пула

1. Скачивание и распаковка архива с файлами пула:
```bash
wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download' 
tar -xzvf archive.tar.gz
```

2. Поиск доступных для импорта пулов в каталоге `zpoolexport`:
```bash
sudo /sbin/zpool import -d zpoolexport/
# Вывод показал пул с именем "otus" и типом "mirror-0" из файлов filea и fileb
```

3. Импорт пула в систему:
```bash
sudo /sbin/zpool import -d zpoolexport/ otus
```

4. Определение параметров импортированного пула:
```bash
sudo /sbin/zfs get available,readonly,recordsize,compression,checksum otus
```

### Итоговые настройки пула `otus`:
* **Размер хранилища (Доступно):** `350M`
* **Тип пула (RAID):** `mirror` (зеркало из двух файлов-дисков)
* **Значение `recordsize`:** `128K`
* **Какое сжатие используется:** `zle`
* **Какая контрольная сумма используется:** `sha256`

---

## Задание 3. Работа со снапшотами и поиск секретного сообщения

### Текст задания
Скачать файл снапшота из удаленного источника, восстановить файловую систему локально через `zfs receive` и найти зашифрованное сообщение в файле `secret_message`.

### Описание команд и их вывод

1. Скачивание файла снапшота:
```bash
sudo wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download
```

2. Восстановление файловой системы из снапшота в пул `otus`:
```bash
sudo zfs receive otus/test@today < otus_task2.file
```

3. Поиск файла `secret_message` внутри восстановленной структуры:
```bash
find /otus/test -name "secret_message"
# Вывод: /otus/test/task1/file_mess/secret_message
```

4. Чтение содержимого секретного файла:
```bash
cat /otus/test/task1/file_mess/secret_message
# Вывод: https://otus.ru/lessons/linux-hl/
```

### Вывод по Заданию 3
Файловая система успешно развернута из снапшота. Внутри неё обнаружен скрытый файл
