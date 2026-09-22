# Домашнее задание: Знакомство с Docker и Docker Compose

## 1. Теоретические вопросы

### В чем разница между образом и контейнером?
* **Образ (Image)** — это неизменяемый шаблон (чертеж), который содержит приложение и все его зависимости. 
* **Контейнер (Container)** — это запущенный и изолированный процесс, созданный на основе данного образа, имеющий свой слой для записи данных.

### Можно ли в контейнере собрать ядро?
**Да, можно.** Сборка ядра — это обычный вычислительный процесс компиляции исходного кода (с использованием `gcc`, `make` и т.д.). Контейнер предоставляет все необходимые изолированные утилиты для успешной компиляции исходного кода ядра в бинарный файл.

## 2. Практическая часть

Кастомный образ Nginx успешно собран на базе легковесного дистрибутива `Alpine Linux` и отправлен в удаленный реестр.

* **Ссылка на собранный образ в Docker Hub:** [https://docker.com](https://docker.com)

### Команда для запуска моего образа:
`docker run -d -p 8080:80 ваш_логин/my-custom-nginx:alpine`


1. Быстрая установка Docker в Ubuntu
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

2. Проверка работы
docker --version
docker run hello-world

<img width="838" height="86" alt="изображение" src="https://github.com/user-attachments/assets/bcfcf89b-1307-465f-8924-6ede642a16fe" />

нужно выйти из пользователя чтобы применились права

<img width="786" height="555" alt="изображение" src="https://github.com/user-attachments/assets/760dad28-d297-4b38-8716-26517ee5ef76" />


3. Сборка кастомного Nginx

`mkdir nginx-project && cd nginx-project`
`nano index.html`
``` html
<!DOCTYPE html>
<html>
<head>
    <title>Custom Nginx Alpine</title>
</head>
<body>
    <h1>Ура! Этот кастомный образ Nginx собран на базе Alpine!</h1>
</body>
</html>
```
`nano Dockerfile`

#1. Базовый образ nginx на alpine

`FROM nginx:alpine`

#2. Обновление пакетов (адаптировано под apk для alpine вместо apt)

`RUN apk update && apk upgrade`

#3. Копируем кастомную страницу (используем COPY согласно методичке)

`COPY index.html /usr/share/nginx/html/index.html`

#4. Указываем порт, который слушает nginx по умолчанию

`EXPOSE 80`

`docker build -t sashapoc/my-custom-nginx:alpine .`

в моём случае так из за подмены ssl

`NODE_TLS_REJECT_UNAUTHORIZED=0 DOCKER_BUILDKIT=0 docker build -t sashapoc/my-custom-nginx:alpine .`

# 2. Проверка работы контейнера локально на порту 8080
`docker run -d -p 8080:80 --name test-nginx sashapoc/my-custom-nginx:alpine`
# 3. Авторизация в Docker Hub через консоль
`docker login`
# 4. Отправка (пуш) собранного образа в ваш репозиторий Docker Hub
`docker push sashapoc/my-custom-nginx:alpine`

<img width="675" height="192" alt="изображение" src="https://github.com/user-attachments/assets/096325c6-aed1-48ae-b80d-19013ee304e4" />
