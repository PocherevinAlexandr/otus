# otus
Обновление ядра системы
Цель:

научиться обновлять ядро в ОС Linux;

Описание/Пошаговая инструкция выполнения домашнего задания:

🎯Задание

    Запустите ВМ c Ubuntu.
    Обновите ядро ОС на новейшую стабильную версию из mainline-репозитория.
    Оформите отчет в README-файле в GitHub-репозитории.

user@user-HVM-domU:~$ uname -r
6.17.0-35-generic
user@user-HVM-domU:~$ mkdir kernel && cd kernel
user@user-HVM-domU:~/kernel$ wget https://kernel.ubuntu.com/mainline/v7.1/amd64/linux-headers-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb
--2026-07-03 14:58:16--  https://kernel.ubuntu.com/mainline/v7.1/amd64/linux-headers-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb
Распознаётся kernel.ubuntu.com (kernel.ubuntu.com)… 185.125.189.76, 185.125.189.74, 185.125.189.75
Подключение к kernel.ubuntu.com (kernel.ubuntu.com)|185.125.189.76|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 4108018 (3,9M) [application/x-debian-package]
Сохранение в: ‘linux-headers-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb.1’

linux-headers-7.1.0-070100-generic_7.1.0-0701 100%[===============================================================================================>]   3,92M  7,83MB/s    за 0,5s

2026-07-03 14:58:17 (7,83 MB/s) - ‘linux-headers-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb.1’ сохранён [4108018/4108018]

user@user-HVM-domU:~/kernel$ wget https://kernel.ubuntu.com/mainline/v7.1/amd64/linux-headers-7.1.0-070100_7.1.0-070100.202606141628_all.deb
--2026-07-03 14:58:26--  https://kernel.ubuntu.com/mainline/v7.1/amd64/linux-headers-7.1.0-070100_7.1.0-070100.202606141628_all.deb
Распознаётся kernel.ubuntu.com (kernel.ubuntu.com)… 185.125.189.75, 185.125.189.76, 185.125.189.74
Подключение к kernel.ubuntu.com (kernel.ubuntu.com)|185.125.189.75|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 14855142 (14M) [application/x-debian-package]
Сохранение в: ‘linux-headers-7.1.0-070100_7.1.0-070100.202606141628_all.deb.1’

linux-headers-7.1.0-070100_7.1.0-070100.20260 100%[===============================================================================================>]  14,17M  21,1MB/s    за 0,7s

2026-07-03 14:58:27 (21,1 MB/s) - ‘linux-headers-7.1.0-070100_7.1.0-070100.202606141628_all.deb.1’ сохранён [14855142/14855142]

user@user-HVM-domU:~/kernel$ wget https://kernel.ubuntu.com/mainline/v7.1/amd64/linux-image-unsigned-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb
--2026-07-03 14:58:44--  https://kernel.ubuntu.com/mainline/v7.1/amd64/linux-image-unsigned-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb
Распознаётся kernel.ubuntu.com (kernel.ubuntu.com)… 185.125.189.76, 185.125.189.75, 185.125.189.74
Подключение к kernel.ubuntu.com (kernel.ubuntu.com)|185.125.189.76|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 17490112 (17M) [application/x-debian-package]
Сохранение в: ‘linux-image-unsigned-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb.1’

linux-image-unsigned-7.1.0-070100-generic_7.1 100%[===============================================================================================>]  16,68M  17,8MB/s    за 0,9s

2026-07-03 14:58:45 (17,8 MB/s) - ‘linux-image-unsigned-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb.1’ сохранён [17490112/17490112]

user@user-HVM-domU:~/kernel$ wget https://kernel.ubuntu.com/mainline/v7.1/amd64/linux-modules-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb
--2026-07-03 14:59:01--  https://kernel.ubuntu.com/mainline/v7.1/amd64/linux-modules-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb
Распознаётся kernel.ubuntu.com (kernel.ubuntu.com)… 185.125.189.76, 185.125.189.74, 185.125.189.75
Подключение к kernel.ubuntu.com (kernel.ubuntu.com)|185.125.189.76|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 170117312 (162M) [application/x-debian-package]
Сохранение в: ‘linux-modules-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb.1’

linux-modules-7.1.0-070100-generic_7.1.0-0701 100%[===============================================================================================>] 162,24M  49,2MB/s    за 3,7s

2026-07-03 14:59:05 (43,6 MB/s) - ‘linux-modules-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb.1’ сохранён [170117312/170117312]

user@user-HVM-domU:~/kernel$ sudo dpkg -i *.deb
[sudo] пароль для user:
Выбор ранее не выбранного пакета linux-headers-7.1.0-070100.
(Чтение базы данных … на данный момент установлено 173170 файлов и каталогов.)
Подготовка к распаковке linux-headers-7.1.0-070100_7.1.0-070100.202606141628_all.deb …
Распаковывается linux-headers-7.1.0-070100 (7.1.0-070100.202606141628) …
Выбор ранее не выбранного пакета linux-headers-7.1.0-070100-generic.
Подготовка к распаковке linux-headers-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb …
Распаковывается linux-headers-7.1.0-070100-generic (7.1.0-070100.202606141628) …
Выбор ранее не выбранного пакета linux-image-unsigned-7.1.0-070100-generic.
Подготовка к распаковке linux-image-unsigned-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb …
Распаковывается linux-image-unsigned-7.1.0-070100-generic (7.1.0-070100.202606141628) …
Выбор ранее не выбранного пакета linux-modules-7.1.0-070100-generic.
Подготовка к распаковке linux-modules-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb …
Распаковывается linux-modules-7.1.0-070100-generic (7.1.0-070100.202606141628) …
Настраивается пакет linux-headers-7.1.0-070100 (7.1.0-070100.202606141628) …
Настраивается пакет linux-headers-7.1.0-070100-generic (7.1.0-070100.202606141628) …
Настраивается пакет linux-modules-7.1.0-070100-generic (7.1.0-070100.202606141628) …
Настраивается пакет linux-image-unsigned-7.1.0-070100-generic (7.1.0-070100.202606141628) …
I: /boot/vmlinuz is now a symlink to vmlinuz-7.1.0-070100-generic
I: /boot/initrd.img is now a symlink to initrd.img-7.1.0-070100-generic
Обрабатываются триггеры для linux-image-unsigned-7.1.0-070100-generic (7.1.0-070100.202606141628) …
/etc/kernel/postinst.d/initramfs-tools:
update-initramfs: Generating /boot/initrd.img-7.1.0-070100-generic
/etc/kernel/postinst.d/zz-update-grub:
Sourcing file `/etc/default/grub'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-7.1.0-070100-generic
Found initrd image: /boot/initrd.img-7.1.0-070100-generic
Found linux image: /boot/vmlinuz-6.17.0-35-generic
Found initrd image: /boot/initrd.img-6.17.0-35-generic
Found memtest86+x64 image: /memtest86+x64.bin
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
user@user-HVM-domU:~/kernel$  ls -al /boot
итого 218092
drwxr-xr-x  4 root root     4096 июл  3 14:59 .
drwxr-xr-x 23 root root     4096 июл  3 13:42 ..
-rw-r--r--  1 root root   302871 мая 26 21:56 config-6.17.0-35-generic
-rw-r--r--  1 root root   307299 июн 14 20:28 config-7.1.0-070100-generic
drwxr-xr-x  5 root root     4096 июл  3 15:00 grub
lrwxrwxrwx  1 root root       31 июл  3 14:59 initrd.img -> initrd.img-7.1.0-070100-generic
-rw-r--r--  1 root root 82050257 июл  3 14:16 initrd.img-6.17.0-35-generic
-rw-r--r--  1 root root 83454096 июл  3 14:59 initrd.img-7.1.0-070100-generic
lrwxrwxrwx  1 root root       28 июл  3 13:48 initrd.img.old -> initrd.img-6.17.0-35-generic
drwx------  2 root root    16384 июл  3 13:43 lost+found
-rw-r--r--  1 root root   142796 апр  8  2024 memtest86+ia32.bin
-rw-r--r--  1 root root   143872 апр  8  2024 memtest86+ia32.efi
-rw-r--r--  1 root root   147744 апр  8  2024 memtest86+x64.bin
-rw-r--r--  1 root root   148992 апр  8  2024 memtest86+x64.efi
-rw-------  1 root root 10475899 мая 26 21:56 System.map-6.17.0-35-generic
-rw-------  1 root root 11899326 июн 14 20:28 System.map-7.1.0-070100-generic
lrwxrwxrwx  1 root root       28 июл  3 14:59 vmlinuz -> vmlinuz-7.1.0-070100-generic
-rw-------  1 root root 16746568 мая 26 21:56 vmlinuz-6.17.0-35-generic
-rw-------  1 root root 17457664 июн 14 20:28 vmlinuz-7.1.0-070100-generic
lrwxrwxrwx  1 root root       25 июл  3 13:48 vmlinuz.old -> vmlinuz-6.17.0-35-generic
user@user-HVM-domU:~/kernel$ sudo update-grub
Sourcing file `/etc/default/grub'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-7.1.0-070100-generic
Found initrd image: /boot/initrd.img-7.1.0-070100-generic
Found linux image: /boot/vmlinuz-6.17.0-35-generic
Found initrd image: /boot/initrd.img-6.17.0-35-generic
Found memtest86+x64 image: /memtest86+x64.bin
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
user@user-HVM-domU:~/kernel$ sudo grub-set-default 0
[sudo] пароль для user:
user@user-HVM-domU:~/kernel$ sudo reboot
Last login: Fri Jul  3 14:50:05 2026 from 10.0.100.25
user@user-HVM-domU:~$ uname -r
7.1.0-070100-generic

⭐️Задание со звездочкой

Собрать ядро самостоятельно из исходных кодов.
#Примечание остальные ядра особенно стабильные не доступны ошибка 404 по этому использовал 5.15.210
sudo apt update
sudo apt install build-essential libncurses-dev bison flex libssl-dev libelf-dev bc git ccache -y
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.15.210.tar.xz
tar -xf linux-5.15.210.tar.xz
cp /boot/config-$(uname -r) .config
cd linux-5.15.210
make menuconfig
save exit
make
# Ubuntu по умолчанию требует цифровую подпись модулей ядра. Без отключения этих строк сборка прервется с ошибкой
# make[1]: *** Нет правила для сборки цели «debian/canonical-certs.pem», требуемой для «certs/x509_certificate_list».  Останов.
# make: *** [Makefile:1924: certs] Ошибка 2
scripts/config --disable SYSTEM_TRUSTED_KEYS
scripts/config --disable SYSTEM_REVOCATION_KEYS
make
# BTF: .tmp_vmlinux.btf: pahole (pahole) is not available
# Failed to generate BTF for vmlinux
# Try to disable CONFIG_DEBUG_INFO_BTF
# make: *** [Makefile:1244: vmlinux] Ошибка 1
user@user-HVM-domU:~/linux-5.15.210$ make menuconfig
# Kernel hacking -> Compile-time checks and compiler options -> Generate BTF typeinfo = отключить
scripts/config --disable SYSTEM_TRUSTED_KEYS
scripts/config --disable SYSTEM_REVOCATION_KEYS
make localmodconfig
make -j2
sudo make modules_install
sudo make install
sudo update-grub
user@user-HVM-domU:~/linux-5.15.210$ sudo update-grub
        Sourcing file `/etc/default/grub'
        Generating grub configuration file ...
        Found linux image: /boot/vmlinuz-7.1.0-070100-generic
        Found initrd image: /boot/initrd.img-7.1.0-070100-generic
        Found linux image: /boot/vmlinuz-6.17.0-35-generic
        Found initrd image: /boot/initrd.img-6.17.0-35-generic
        Found linux image: /boot/vmlinuz-5.15.210
        Found initrd image: /boot/initrd.img-5.15.210
        Found memtest86+x64 image: /memtest86+x64.bin
        Warning: os-prober will not be executed to detect other bootable partitions.
        Systems on them will not be added to the GRUB boot configuration.
        Check GRUB_DISABLE_OS_PROBER documentation entry.
        Adding boot menu entry for UEFI Firmware Settings ...
        done
sudo awk -F\' '/menuentry / {print $2}' /boot/grub/grub.cfg
sudo grub-reboot "Advanced options for Ubuntu>Ubuntu, with Linux 5.15.210"
sudo reboot
uname -r
<img width="219" height="45" alt="изображение" src="https://github.com/user-attachments/assets/028dcb7b-92a4-4bcf-ba88-db07585c09cc" />
