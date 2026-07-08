
## 📝 Лабораторная работа: Работа с mdadm (RAID)

В рамках данной задачи был разработан скрипт автоматизации и проведено тестирование отказоустойчивости программного RAID-6 массива.

### 🛠️ Скрипт автоматического развертывания RAID и разделов

Скрипт полностью очищает суперблоки, собирает массив из 5 дисков, размечает GPT на 5 равных секторов, создает файловые системы ext4 и монтирует их в `/raid/part*`.

```bash
#!/bin/bash
set -e

echo "=== 1. Очистка блокировок и зануление суперблоков ==="
sudo mdadm --stop /dev/md0 2>/dev/null || true
sudo mdadm --stop /dev/md/* 2>/dev/null || true
# Цикл поочередно обработает каждый диск по отдельности
for disk in /dev/xvdb /dev/xvdc /dev/xvde /dev/xvdf; do
    sudo vgchange -an "$disk" 2>/dev/null || true
    sudo wipefs -a -f "$disk"
    sudo mdadm --zero-superblock --force "$disk" 2>/dev/null || true
done

echo "=== 2. Создание RAID-6 массива ==="
# Создаем массив /dev/md0 уровня RAID-6 (-l 6) из 4 дисков (-n 4)
# Флаг --yes автоматически подтверждает создание в неинтерактивном режиме
echo y | sudo mdadm --create --verbose /dev/md0 -l 6 -n 4 /dev/xvdb /dev/xvdc /dev/xvde /dev/xvdf --run

echo "=== 3. Проверка статуса сборки массива ==="
cat /proc/mdstat
sudo mdadm -D /dev/md0

echo "=== 4. Разметка GPT и создание 5 разделов ==="
# Создаем таблицу разделов GPT на RAID-массиве
sudo parted -s /dev/md0 mklabel gpt

# Нарезаем 5 равных партиций по 20% каждая
sudo parted /dev/md0 mkpart primary ext4 0% 20%
sudo parted /dev/md0 mkpart primary ext4 20% 40%
sudo parted /dev/md0 mkpart primary ext4 40% 60%
sudo parted /dev/md0 mkpart primary ext4 60% 80%
sudo parted /dev/md0 mkpart primary ext4 80% 100%

echo "=== 5. Создание файловой системы ext4 ==="
# Форматируем созданные разделы (/dev/md0p1 ... /dev/md0p5) в ext4
for i in $(seq 1 5); do 
   echo y | sudo mkfs.ext4 /dev/md0p${i}
done

echo "=== 6. Создание точек монтирования и монтирование ==="
# Создаем целевые каталоги в системе
sudo mkdir -p /raid/part{1,2,3,4,5}

# Последовательно монтируем каждый раздел
for i in $(seq 1 5); do 
    sudo mkdir -p "/raid/part${i}"
    sudo mount "/dev/md0p${i}" "/raid/part${i}"
done

echo "=== Задание успешно выполнено! ==="
```

### 📄 Отчет по командам (Починка RAID и создание разделов)

#### 1. Определение дисков в системе
Поиск свежеподключенных блочных устройств выполнялся командами:
```bash
sudo lshw -short | grep disk
lsblk
```
Обнаружено 5 накопителей: `/dev/xvdb`, `/dev/xvdc`, `/dev/xvdd`, `/dev/xvde`, `/dev/xvdf`.

#### 2. Разметка и интеграция в систему
Разметка собранного массива `/dev/md0` выполнялась с помощью утилиты `parted`:
* **Создание таблицы GPT:** `parted -s /dev/md0 mklabel gpt`
* **Форматирование разделов в ext4 (в цикле):** `for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done`
* **Монтирование разделов:** `for i in $(seq 1 5); do sudo mount /dev/md0p$i /raid/part$i; done`

#### 3. Инструкция по обслуживанию и починке RAID-массива
Для проверки отказоустойчивости архитектуры RAID-6 проведена симуляция аварии:

1. **Искусственное повреждение диска (faulty):**
   ```bash
   mdadm /dev/md0 --fail /dev/xvde
   ```
   *Вывод `cat /proc/mdstat` фиксирует маркер `(F)` возле диска sde. Массив переходит в деградированное состояние `[5/4]`.*

2. **Извлечение накопителя из дисковой группы:**
   ```bash
   mdadm /dev/md0 --remove /dev/xvde
   ```

3. **Горячая замена устройства (добавление диска обратно):**
   ```bash
   mdadm /dev/md0 --add /dev/xvde
   ```

4. **Контроль процесса регенерации (Rebuilding):**
   ```bash
   cat /proc/mdstat
   ```
   *Команда отображает статус `recovery = X.X%`. После завершения синхронизации массив возвращается в штатное состояние.*
