# Базовый образ с нужными пакетами
FROM nginx

# Установка нужных зависимостей
RUN apt-get update && \
    apt-get install -y gcc spawn-fcgi libfcgi-dev vim htop && \
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
CMD ["sh", "-c", "spawn-fcgi -p 8080 /usr/local/bin/hello.fcgi || echo 'Failed to start FastCGI' && nginx -g 'daemon off;'"]
# CMD ["sh", "-c", "nginx -g 'daemon off;' && spawn-fcgi -p 8080 /usr/local/bin/hello.fcgi"]


