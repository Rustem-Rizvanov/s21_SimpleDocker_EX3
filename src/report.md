##Part 1. Готовый докер

- **Взяли официальный докер образ с nginx и выкачали его при помощи `docker pull`**
![docker_pull](./images/docker_pull.png)

- **Проверили наличие докер образа через `docker images`**
![docker_images](./images/docker_images.png)

- **Запустили докер образ через `docker run -d [image_id|repository]`**

> **-d:** это флаг, который указывает Docker на запуск контейнера в фоновом режиме (detached mode). Это означает, что контейнер будет работать в фоновом режиме, и командная строка будет освобождена для дальнейшего использования.
>

![docker_run](./images/docker_run.png)

- **Проверили, что образ запустился через `docker ps`**
![docker_ps](./images/docker_ps.png)

>Команда `docker ps` используется для вывода списка запущенных контейнеров Docker.  
>При запуске команды `docker ps` без дополнительных флагов будут отображены только запущенные контейнеры в текущий момент времени.
>

- **Посмотрели информацию о контейнере через `docker inspect [container_id|container_name]`**
![docker_inspect](./images/docker_inspect.png)


- размер контейнера  
![size2](./images/size2.png)

- список замапленных портов  
![ports](./images/ports.png)

>Маппинг нужен для того, чтобы все запросы, проходящие через порт хоста, перенаправлялись в Docker-контейнер. Другими словами, сопоставление портов делает процессы внутри контейнера доступными извне.
>

- ip контейнера
![ip](./images/ip.png)

- **Остановили докер образ через `docker stop [container_id|container_name]` и проверили, что образ остановился через `docker ps`**
![docker_stop_ps](./images/docker_stop_ps.png)

- **Запустили докер с портами 80 и 443 в контейнере, замапленными на такие же порты на локальной машине, через команду `run`**
![80_443](./images/80_443.png)

- **Проверили, что в браузере по адресу *localhost:80* доступна стартовая страница `nginx`**
![check](./images/check.png)

- **Перезапустили докер контейнер через `docker restart [container_id|container_name]` и проверили, что контейнер запустился**
![docker_restart_ps](./images/docker_restart_ps.png)

---

##Part 2. Операции с контейнером

- **Прочитали конфигурационный файл `nginx.conf` внутри докер контейнера через команду `exec`**
![nginx_conf](./images/nginx_conf.png)

- **Создали на локальной машине файл `nginx.conf`**
`nano nginx.conf`

- **Настроили в нем по пути `/status` отдачу страницы статуса сервера `nginx`**
![nano_nginx](./images/nano_nginx.png)

- **Скопировали созданный файл `nginx.conf` внутрь докер образа через команду `docker cp`**
![docker_cp](./images/docker_cp.png)

- **Перезапустили `nginx` внутри докер образа через команду `exec`**
![reload](./images/reload.png)

- **Проверили, что по адресу `localhost:80/status` отдается страничка со статусом сервера `nginx`**
![curl](./images/curl.png)
![curl_br](./images/curl_br.png)

- **Экспортировали контейнер в файл `container.tar` через команду `export`и остановили контейнер**
![export_stop](./images/export_stop.png)

- **Удалили образ через `docker rmi [image_id|repository]`, не удаляя перед этим контейнеры**
![rmi](./images/rmi.png)

- **Удалили остановленный контейнер**
![rm](./images/rm.png)

- **Импортировали контейнер обратно через команду `import`**
![import](./images/import.png)

- **Запустили импортированный контейнер**
![run_again](./images/run_again.png)

- **Проверили, что по адресу `localhost:80/status` отдается страничка со статусом сервера `nginx`**
![new_status](./images/new_status.png)
![new_status_term](./images/new_status_term.png)

---

##Part 3. Мини веб-сервер

- **Написали мини сервер на C и FastCgi, который будет возвращать простейшую страничку с надписью Hello World!**
![server_c](./images/server_c.png)


- **Написали свой nginx.conf, который будет проксировать все запросы с 81 порта на 127.0.0.1:8080**
![nginx_81](./images/nginx_81.png)

	- Скопировали созданный nginx.conf и мини сервер в контейнер и зашли в него. 
	![cp_it](./images/cp_it.png)
	
	> **-y** опция команды `apt-get` указывает на автоматическое подтверждение установки пакетов без запроса подтверждения от пользователя. Это полезно при автоматической установке пакетов в скриптах или в среде контейнеров, где интерактивное взаимодействие с пользователем невозможно.
	>

	- Установили требуемые ПО
	![update](./images/update.png)
	![install](./images/install.png)
	
	- Скомпилировали и **запустили написанный мини сервер через spawn-fcgi на порту 8080**
	![compilation](./images/compilation.png)

	
- **Проверили, что в браузере по localhost:81 отдается написанная вами страничка**
![81](./images/81.png)	

- **Положили файл nginx.conf по пути ./nginx/nginx.conf**
`docker cp nginx.conf silly_shirley:/etc/nginx/nginx.conf`

---

##Part 4. Свой докер

- **Написали свой докер образ, который:**

	- собирает исходники мини сервера на FastCgi из Части 3
	- запускает его на 8080 порту
	- копирует внутрь образа написанный ./nginx/nginx.conf
	- запускает nginx.
	![dockerfile](./images/dockerfile.png)
	![run](./images/run.png)	

- **Собрали написанный докер образ через docker build при этом указав имя и тег**
	![build](./images/build.png)	
	
- **Проверили через docker images, что все собралось корректно**
	![images](./images/images.png)	
	
- **Запустили собранный докер образ с маппингом 81 порта на 80 на локальной машине и маппингом папки ./nginx внутрь контейнера по адресу, где лежат конфигурационные файлы nginx'а (см. Часть 2)**
	![81_80](./images/81_80.png)
	
	> - **-p 81:80** - это опция для маппинга портов. Она указывает, что порт 80 на локальной машине будет проксироваться на порт 81 внутри контейнера.
	> - **-v** - это опция для маппинга папки. Она указывает, что текущая папка `/Users/nealpadm/DO5_SimpleDocker-1/src/04/nginx.conf` на локальной машине будет монтироваться в путь `/etc/nginx/nginx.conf` внутри контейнера.
	>
	
- **Проверили, что по localhost:80 доступна страничка написанного мини сервера**
	![localhost](./images/localhost.png)
	
- **Дописали в ./nginx/nginx.conf проксирование странички /status, по которой надо отдавать статус сервера nginx**
	![root_nginx_conf](./images/root_nginx_conf.png)

- **Перезапустили докер образ**
	![reload_nginx](./images/reload_nginx.png)

- **Проверили, что теперь по localhost:80/status отдается страничка со статусом nginx**
	![curl_status](./images/curl_status.png)
	
---

##Part 5. Dockle

- **Просканировали образ из предыдущего задания через `dockle [image_id|repository]`**
	![dockle](./images/dockle.png)

- **Исправили образ так, чтобы при проверке через dockle не было ошибок и предупреждений**
	![new_dockle](./images/new_dockle.png)
	
---

##Part 6. Базовый Docker Compose

- **Написали файл docker-compose.yml, с помощью которого:**

	- Подняли докер контейнер из Части 5 (он должен работать в локальной сети, т.е. не нужно использовать инструкцию EXPOSE и мапить порты на локальную машину)
		
	- Подняли докер контейнер с nginx, который будет проксировать все запросы с 8080 порта на 81 порт первого контейнера
	![last_nginx](./images/last_nginx.png)
	![yml](./images/yml.png)
	
		- Изменили скрипт для запуска и перезагрузки nginx. Сделали так, чтобы контейнер пребывал в запущенном состоянии. 
	![last_run](./images/last_run.png)

- **Остановили все запущенные контейнеры**
	`docker-compose down`
	![stop_all](./images/stop_all.png)

- **Собрали и запустили проект с помощью команд `docker-compose build` и `docker-compose up`**
	![compose_build](./images/compose_build.png)

- **Проверили, что в браузере по *localhost:80* отдается написанная вами страничка, как и ранее**
	![browser_80](./images/browser_80.png)
	![term_80](./images/term_80.png)