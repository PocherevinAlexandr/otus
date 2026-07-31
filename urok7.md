# Лабораторная работа: Сборка кастомного DEB-пакета Nginx с модулем Brotli и создание локального APT-репозитория в Ubuntu 24.04

Данный репозиторий содержит пошаговое руководство по сборке веб-сервера Nginx с поддержкой модуля сжатия **Brotli** от Google и развертыванию локального репозитория для дистрибуции пакетов в окружении **Ubuntu 24.04 LTS**. Работа полностью переведена на нативные инструменты Debian/Ubuntu (`.deb`, `dpkg-dev`, `apt`).

---

## Часть 1. Сборка пакетов и создание репозитория (Сервер)

### Шаг 1. Подготовка окружения
Установите компиляторы, библиотеки и инструменты для сборки нативных пакетов:
```bash
sudo apt update
sudo apt install -y dpkg-dev build-essential devscripts quilt \
    cmake gcc git nano nginx libpcre3-dev zlib1g-dev libssl-dev wget
```

### Шаг 2. Загрузка исходного кода Nginx
1. Включите загрузку исходных кодов (`deb-src`) в конфигурации репозиториев Ubuntu (формат DEB822):
```bash
sudo sed -i 's/Types: deb/Types: deb deb-src/g' /etc/apt/sources.list.d/ubuntu.sources
sudo apt update
```

2. Создайте рабочую директорию, скачайте официальные исходники Nginx и установите все зависимости для его компиляции:
```bash
mkdir ~/deb && cd ~/deb
apt source nginx
sudo apt build-dep -y nginx
```

### Шаг 3. Скачивание и компиляция модуля ngx_brotli
Склонируйте официальный репозиторий модуля Brotli от Google и соберите статическую библиотеку:
```bash
cd /root
git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli.git
cd ngx_brotli/deps/brotli && mkdir out && cd out

cmake -DCMAKE_BUILD_TYPE=Release \
      -DBUILD_SHARED_LIBS=OFF \
      -DCMAKE_C_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" \
      -DCMAKE_CXX_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" \
      -DCMAKE_INSTALL_PREFIX=../installed ..

cmake --build . --config Release -j 2 --target brotlienc
cd /root
```

### Шаг 4. Автоматическая интеграция Brotli и сборка DEB-пакетов
1. Перейдите в каталог с исходным кодом Nginx:
```bash
cd ~/deb/nginx-*/
```

2. Автоматически внедрите флаг подключения модуля `ngx_brotli` во все конфигурации сборки (`core`, `full`, `light`) с помощью утилиты `sed`:
```bash
sed -i 's/--with-ld-opt=/--add-module=\/root\/ngx_brotli --with-ld-opt=/g' debian/rules
```

3. Обновите журнал изменений пакета и запустите компиляцию:
```bash
dch -i "Added ngx_brotli module support"
dpkg-buildpackage -b -uc -us
```
*После успешного завершения готовые исправленные `.deb` пакеты сгенерируются в каталоге `~/deb/`.*

### Шаг 5. Создание локального APT-репозитория
1. Создайте каталог для публикации файлов вашего репозитория и скопируйте туда все созданные пакеты:
```bash
sudo mkdir -p /var/www/html/repo/ubuntu
sudo cp ~/deb/*.deb /var/www/html/repo/ubuntu/
```

2. Сгенерируйте файл индексов манифеста пакетов (аналог утилиты `createrepo`):
```bash
cd /var/www/html/repo
sudo bash -c 'dpkg-scanpackages ubuntu /dev/null | gzip -9c > ubuntu/Packages.gz'
```

### Шаг 6. Настройка веб-сервера Nginx для раздачи репозитория
1. Откройте конфигурационный файл стандартного сайта Nginx:
```bash
sudo nano /etc/nginx/sites-available/default
```

2. Добавьте директиву листинга файлов (`autoindex on;`) внутри блока `server { ... }`:
```nginx
location /repo {
    autoindex on;
    try_files \(uri\)uri/ =404;
}
```

3. Проверьте конфигурацию и перезапустите веб-сервер Nginx для применения изменений:
```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

## Часть 2. Подключение репозитория и проверка работы (Клиент)

*Примечание: Если тестирование производится локально на той же машине, адресом сервера будет `http://localhost/repo`.*

### Шаг 1. Подключение репозитория и настройка приоритетов
1. Добавьте созданный репозиторий в систему как доверенный (без использования GPG-подписи):
```bash
sudo bash -c 'cat >> /etc/apt/sources.list.d/localrepo.list << EOF
deb [trusted=yes] http://localhost/repo ubuntu/
EOF'
```

2. Зафиксируйте наивысший приоритет для вашего репозитория (Apt Pinning), чтобы система принудительно скачивала ваш кастомный Nginx, а не заменяла его стандартным из интернета:
```bash
sudo bash -c 'cat > /etc/apt/preferences.d/localrepo << EOF
Package: *
Pin: origin localhost
Pin-Priority: 1000
EOF'
```

### Шаг 2. Обновление кэша и установка кастомного Nginx с Brotli
Обновите списки пакетов и установите веб-сервер Nginx вместе со всеми сопутствующими модулями напрямую из вашего нового локального репозитория:
```bash
sudo apt update
sudo apt install -y nginx nginx-common nginx-core libnginx-mod-http-geoip libnginx-mod-http-image-filter libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream libnginx-mod-stream-geoip
```

### Шаг 3. Проверка наличия модуля Brotli
Перезапустите веб-сервер и проверьте аргументы конфигурации:
```bash
sudo systemctl restart nginx
nginx -V
```
В выводе аргументов конфигурации в самом начале обязательно должен присутствовать флаг:
`--add-module=/root/ngx_brotli`

### Шаг 4. Проверка добавления стороннего пакета в репозиторий
1. Перейдите на Сервер в папку репозитория, скачайте туда любой сторонний `.deb` пакет (например, клиент мониторинга Percona PMM) и переиндексируйте репозиторий:
```bash
cd /var/www/html/repo/ubuntu
sudo wget https://downloads.percona.com/downloads/pmm3/3.9.0/binary/debian/noble/x86_64/pmm-client_3.9.0-1.noble_amd64.deb

cd /var/www/html/repo
sudo bash -c 'dpkg-scanpackages ubuntu /dev/null | gzip -9c > ubuntu/Packages.gz'
```

2. На Клиенте обновите кэш APT и установите этот пакет из вашего локального источника:
```bash
sudo apt update
sudo apt install -y pmm-client
```
