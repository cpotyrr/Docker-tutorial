## Part 1. Готовый докер

- **Взял официальный докер образ с nginx и выкачал его при помощи `docker pull`**
![alt text](img/docker_pull.png)
- **Проверил наличие докер образа через `docker images`** 
![alt text](img/docker_images.png)  

- **Запустил докер образ через `docker run -d [image_id|repository]`**
![alt text](img/docker_run.png)
> **-d:** указывает на запуск контейнера в фоновом режиме (detached). Это означает, что контейнер будет работать в фоновом режиме.

- **Проверил, что образ запустился через `docker ps`**
![alt text](img/docker_ps.png)
>Команда `docker ps` используется для вывода списка запущенных контейнеров Docker.  (docker process)


- **Посмотрел информацию о контейнере через `docker inspect [container_id|container_name]`**
![alt text](img/docker_inspect.png)
- **размер контейнера:**  
![alt text](img/size.png)
- **список замапленных портов:**  
![alt text](img/Ports.png)
- **ip контейнера:** 
![alt text](img/ipaddress.png)  

- **Остановил докер образ через `docker stop cranky_khorana` (имя моего контейнера):**
![alt text](img/docker_stop.png)  
- Проверил, что образ остановился через `docker ps`  
![alt text](img/docker_ps_after_stop.png)  

- **Запустил докер с портами 80 и 443 в контейнере, замапленными на такие же порты на локальной машине, через команду `run`** 
![alt text](img/run_80_443.png)  

- **Проверил, что в браузере по адресу *localhost:80* доступна стартовая страница `nginx`** 
- ![alt text](img/localhost.png)  
 
- **Перезапустил докер контейнер через `docker restart [container_id|container_name]` и проверили, что контейнер запустился**  
![alt text](img/docker_restart.png)  

---

## Part 2. Операции с контейнером

- **Прочитал конфигурационный файл `nginx.conf` внутри докер контейнера через команду `docker exec cranky_khorana cat /etc/nginx/nginx.conf`** 

![alt text](img/docker_exec.png)  

- **Создал на локальной машине файл `nginx.conf`**
`vim nginx.conf`  
- **Настроил в нем по пути `/status` отдачу страницы статуса сервера `nginx`**:  
![alt text](img/nginx_conf.png)  

Это минимальный конфиг, который делает следующее:

- listen 80; — сервер слушает порт 80.
- В блоке location /status {} задается доступ к странице статуса.
- stub_status on; — включает статусную страницу.
- allow 127.0.0.1; — разрешает доступ только с локального хоста.
- deny all; — запрещает доступ всем остальным.  

- **Скопировали созданный файл `nginx.conf` внутрь докер образа через команду `docker cp`**  
 ![alt text](img/docker_cp_nginx.png)  

- **Перезапустил `nginx` внутри докер образа через команду `exec`**  
![alt text](img/docker_nginx_reload.png)  

- **Проверил, что по адресу `localhost:80/status` отдается страничка со статусом сервера `nginx`** 
![alt text](img/localhost_80_status.png)
![alt text](img/status_browser.png)  

- **Экспортировал контейнер в файл `container.tar` через команду `export`и остановили контейнер**  
![alt text](img/docker_export_and_stop.png)  

- **Удалил образ через `docker rmi [image_id|repository]`, не удаляя перед этим контейнеры** 
![alt text](img/rmi_f_nginx.png)  

- **Удалил остановленный контейнер**  
![alt text](img/docker_rm.png)  

- **Импортировал контейнер обратно через команду `import`**  
![alt text](img/import.png)  


- **Запустил импортированный контейнер**  
![alt text](img/docker_images_2.png)  
![alt text](img/run_imported.png)  

- **Проверил, что по адресу `localhost:80/status` отдается страничка со статусом сервера `nginx`**  
![alt text](img/status_after_import.png)  
![alt text](img/status_after_import_in_colsole.png)  

---

## Part 3. Мини веб-сервер

- **Написал мини сервер на C и FastCgi, который будет возвращать простейшую страничку с надписью Hello World!** 
![alt text](img/hello_world.png)  

- **Написал свой nginx.conf, который будет проксировать все запросы с 81 порта на 127.0.0.1:8080**  
![alt text](img/nginx_conf_fastcgi.png) 

- Скопировал созданный nginx.conf и мини сервер в контейнер и зашел в него. 
![alt text](img/entered_server.png) 

- Установил требуемые ПО:  
![alt text](img/downloading_soft.png)  

- Скомпилировал и **запустил написанный мини сервер через spawn-fcgi на порту 8080** 
![alt text](img/child_swapned.png) 

- **Проверил, что в браузере по localhost:81 отдается "Hello World!"**  
![alt text](img/hello_world_html.png)  

- **Положил файл nginx.conf по пути ./nginx/nginx.conf:**
`docker cp nginx.conf gifted_payne:/etc/nginx/nginx.conf`  
![alt text](img/copy_nginx_conf.png)  

## Part 4. Свой докер
- **Написал свой докер образ, который:**
	- собирает исходники мини сервера на FastCgi из [Части 3](#part-3-мини-веб-сервер)
	- запускает его на 8080 порту
	- копирует внутрь образа написанный ./nginx/nginx.conf
    - запускает nginx.  

![alt text](img/dockerfile.png)

Dockerfile с объяснениями каждой строчки:
```Dockerfile
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
```

В докере предпочтительно запускать сервисы через команды, которые блокируют основной процесс контейнера Поэтому: `nginx -g 'daemon off;` Эта команда запускает nginx в приоритетном-режиме, что предпочтительнее в контейнерных средах, так как поддерживает жизнеспособность основного процесса. Это гарантирует, что процесс контейнера будет жить до тех пор, пока работает nginx. Контейнер завершится, если основной процесс завершится.

- **Собрал написанный докер образ через docker build при этом указав имя и тег**  
![alt text](img/built_image.png)  

- **Проверил через docker images, что все собралось корректно**
![alt text](img/image_bulat_cpotyr.png)  

- **Запустил собранный докер образ с маппингом 81 порта на 80 на локальной машине и маппингом папки ./nginx внутрь контейнера по адресу, где лежат конфигурационные файлы nginx'а**  
![alt text](img/container_bulat_cpotyr.png)  
	> - **-p 81:80** - это опция для маппинга портов. Она указывает, что порт 80 на локальной машине будет проксироваться на порт 81 внутри контейнера.
	> - **-v** - это опция для маппинга папки. Она указывает, что текущая папка `/home/cpotyr/Desktop/DO5_SimpleDocker-1/src/nginx.conf` на локальной машине будет монтироваться в путь `/etc/nginx/nginx.conf` внутри контейнера.
	>

- **Проверил, что по localhost:80 доступна страничка написанного мини сервера:**  
![alt text](img/curl_localhost_80.png)
![alt text](img/hello_world_in_browser.png)  

- **Дописал в ./nginx/nginx.conf проксирование странички /status, по которой надо отдавать статус сервера nginx**  
![alt text](img/defined_status.png)  

- **Перезапустил докер образ**  
![alt text](img/reload_nginx_after_defining_status_page.png)  

- **Проверил, что теперь по localhost:80/status отдается страничка со статусом nginx**  
- ![alt text](img/status_page.png)  
![alt text](img/curl_localhost_status.png)  

## Part 5. Dockle

- **Просканировал образ из предыдущего задания через `dockle [image_id|repository]` предварительно скачав dockle**  

Мой Dockerfile до исправления ошибок:
```Dockerfile
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
```

Ошибки и предупреждения образа, которые вывел Dockle:  
![alt text](img/dockle_report.png)  

- **Исправил образ так, чтобы при проверке через dockle не было ошибок и предупреждений**  

**Исправленный Dockerfile:**

```Dockerfile
# Базовый образ с нужными пакетами
FROM nginx

# Установка нужных зависимостей
RUN apt-get update && \
    apt-get install -y gcc spawn-fcgi libfcgi-dev procps vim htop; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*; \
    chown -R nginx:nginx /etc/nginx/nginx.conf; \
    chown -R nginx:nginx /var/cache/nginx; \
    chown -R nginx:nginx /home; \
    touch /var/run/nginx.pid; \
    chown -R nginx:nginx /var/run/nginx.pid

USER nginx

# Копирование исходников мини-сервера внутрь контейнера
COPY server/hello.c /usr/src/hello.c

USER root
# Компиляция мини-сервера
RUN gcc /usr/src/hello.c -o /usr/local/bin/hello.fcgi -lfcgi
USER nginx

# Копия конфигурационного файла nginx внутрь контейнера
COPY server/nginx.conf /etc/nginx/nginx.conf

# Открытие порта 80 для внешних подключений
EXPOSE 80

# Запуск FastCGI сервер на порту 8080 и nginx
ENTRYPOINT ["sh", "-c", "spawn-fcgi -p 8080 /usr/local/bin/hello.fcgi && nginx -g 'daemon off;'"]

HEALTHCHECK NONE
```
Отчёт от Dockle после исправления(INFO это не ошибки):  
![alt text](img/dockle_info.png)  

##  Part 6. Базовый Docker Compose

- **Написал файл docker-compose.yml, с помощью которого:**

	- Поднял докер контейнер из Части 5 (он должен работать в локальной сети, т.е. не нужно использовать инструкцию EXPOSE и мапить порты на локальную машину)
		
	- Поднял докер контейнер с nginx, который будет проксировать все запросы с 8080 порта на 81 порт первого контейнера

![alt text](img/cat_nginx_conf_part_6.png)  
![alt text](img/docker_compose_yml.png)  

- **Остановил все запущенные контейнеры**  
![alt text](img/docker_compose_down.png)  

- **Собрал и запустил проект с помощью команд `docker-compose build` и `docker-compose up`**  

![alt text](img/docker_compose_build.png)  
![alt text](img/docker_compose_up_d.png)  
![alt text](img/docker_compose_up_d_2.png)  
![alt text](img/docker_ps_after_compose.png)  


- **Проверил, что в браузере по *localhost:80* отдается написанная вами страничка, как и ранее**  
![alt text](img/hello_world_in_browser_after_docker_compose.png)  
![alt text](img/curl_localhost_80_after_docker_compose.png)  

---
## Всё!
![alt text](img/Docker.png)  
