# Домашнее задание: Знакомство с Docker и Docker Compose

Цель: освоить базовые принципы работы с Docker, научиться создавать, настраивать и управлять контейнерами;

Описание/Пошаговая инструкция выполнения домашнего задания:

🎯 Задание

    Установите Docker на хост машину https://docs.docker.com/engine/install/ubuntu/
    Установите Docker Compose - как плагин, или как отдельное приложение
    Создайте свой кастомный образ nginx на базе alpine. После запуска nginx должен отдавать кастомную страницу (достаточно изменить дефолтную страницу nginx)
    Определите разницу между контейнером и образом
    Вывод опишите в домашнем задании.
    Ответьте на вопрос: Можно ли в контейнере собрать ядро?


🗂Формат сдачи

Собранный образ необходимо запушить в docker hub и дать ссылку на ваш репозиторий.



## 1. Теоретические вопросы

### В чем разница между образом и контейнером?
* **Образ (Image)** — это неизменяемый шаблон (чертеж), который содержит приложение и все его зависимости. 
* **Контейнер (Container)** — это запущенный и изолированный процесс, созданный на основе данного образа, имеющий свой слой для записи данных.

### Можно ли в контейнере собрать ядро?
**Да, можно.** Сборка ядра — это обычный вычислительный процесс компиляции исходного кода (с использованием `gcc`, `make` и т.д.). Контейнер предоставляет все необходимые изолированные утилиты для успешной компиляции исходного кода ядра в бинарный файл.

## 2. Практическая часть

Кастомный образ Nginx успешно собран на базе легковесного дистрибутива `Alpine Linux` и отправлен в удаленный реестр.

* **Ссылка на собранный образ в Docker Hub:** [https://docker.com](https://hub.docker.com/repository/docker/sashapoc/my-custom-nginx)

### Команда для запуска моего образа:
`docker run -d -p 8080:80 sashapoc/my-custom-nginx:alpine`


### Шаг 1. Установка Docker и Docker Compose на хост-машину
```bash
# Скачивание и запуск официального установщика
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Добавление пользователя в группу docker для работы без sudo
sudo usermod -aG docker $USER
```
*После выполнения команд был произведен перезапуск пользовательской сессии (выход и повторный вход в систему) для успешного обновления прав доступа к сокету Docker.*

### Шаг 2. Проверка версий и работоспособности
Убеждаемся, что компоненты Docker и плагин Compose успешно функционируют:
```bash
docker --version
docker run hello-world
```

<img width="838" height="86" alt="изображение" src="https://github.com/user-attachments/assets/bcfcf89b-1307-465f-8924-6ede642a16fe" />

```bash
docker compose version
```

<img width="508" height="41" alt="изображение" src="https://github.com/user-attachments/assets/9b30f430-1561-482e-bb2b-6da8b88db73a" />

<img width="786" height="555" alt="изображение" src="https://github.com/user-attachments/assets/760dad28-d297-4b38-8716-26517ee5ef76" />


### Шаг 3. Создание проекта и автоматическая генерация конфигурационных файлов
```bash
# Создание директории проекта
mkdir nginx-project && cd nginx-project

# Генерация кастомной HTML-страницы
echo '<!DOCTYPE html>
<html>
<head>
    <title>Custom Nginx Alpine</title>
</head>
<body>
    <h1>Ура! Этот кастомный образ Nginx собран на базе Alpine!</h1>
</body>
</html>' > index.html

# Генерация Dockerfile
echo 'FROM nginx:alpine
RUN apk update && apk upgrade
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80' > Dockerfile
```

### Шаг 4. Локальная сборка и запуск контейнера
Из-за особенностей сетевого окружения (подмена SSL-сертификатов на шлюзе) строгая TLS-проверка и движок BuildKit были временно отключены на этапе сборки:

```bash
# Сборка кастомного образа с обходом TLS-проверки
NODE_TLS_REJECT_UNAUTHORIZED=0 DOCKER_BUILDKIT=0 docker build -t sashapoc/my-custom-nginx:alpine .

# Запуск контейнера в фоновом режиме с пробросом порта 8080
docker run -d -p 8080:80 --name test-nginx sashapoc/my-custom-nginx:alpine
```

Для верификации работы веб-сервера была выполнена локальная проверка доступности порта с помощью `curl`:
```bash
curl http://localhost:8080
```
*Контейнер успешно перехватил запрос и отдал кастомную страницу с текстом о сборке на Alpine Linux.*

<img width="675" height="192" alt="изображение" src="https://github.com/user-attachments/assets/096325c6-aed1-48ae-b80d-19013ee304e4" />

### Шаг 5. Пуш образа в удаленный реестр Docker Hub
```bash
# Авторизация в реестре с использованием Personal Access Token вместо пароля
docker login

# Отправка собранного образа в Docker Hub
docker push sashapoc/my-custom-nginx:alpine
```
