# Базовый образ с нужными пакетами
FROM nginx

# Установка нужных зависимостей
RUN apt-get update && \
    apt-get install -y gcc spawn-fcgi libfcgi-dev procps vim htop && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Копирование исходников мини-сервера внутрь контейнера
COPY hello.c /usr/src/hello.c

# Компиляция мини-сервера
RUN gcc /usr/src/hello.c -o /usr/local/bin/hello.fcgi -lfcgi

# Копия конфигурационного файла nginx внутрь контейнера
COPY nginx.conf /etc/nginx/nginx.conf

# Открытие порта 80 для внешних подключений
EXPOSE 80

# Запуск FastCGI сервер на порту 8080 и nginx
ENTRYPOINT ["sh", "-c", "spawn-fcgi -p 8080 /usr/local/bin/hello.fcgi && nginx -g 'daemon off;'"]