## Задание 1

### Шаг 1: Подготовка Docker и Docker Compose

1. **Убедитесь, что Docker и Docker Compose установлены**:
   Если Docker не установлен, выполните следующие команды для его установки:

   ```bash
   sudo apt update
   sudo apt install docker.io -y
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

2. **Установите Docker Compose (если не установлен)**:
   Если у вас уже установлена версия Compose, можете пропустить этот шаг. Если нет, выполните:
   
   ```bash
   sudo apt install docker-compose-plugin
   ```

3. **Проверьте версии**:
   Убедитесь, что Docker и Docker Compose установлены корректно:
   
   ```bash
   docker --version
   docker compose version
   ```

### Шаг 2: Создание Docker Compose файла для Zabbix

1. Перейдите в каталог, где вы хотите создать файл `docker-compose.yml`:

   ```bash
   cd ~/Documents/netology
   ```

2. Создайте файл `docker-compose.yml`:

   ```bash
   vim docker-compose.yml
   ```

3. Вставьте следующий контент в `docker-compose.yml`:

   ```yaml
   version: '3'

   services:
     zabbix-db:
       image: postgres:16
       container_name: zabbix-db
       environment:
         POSTGRES_DB: zabbix
         POSTGRES_USER: zabbix
         POSTGRES_PASSWORD: zabbixpass
       volumes:
         - ./zabbix-postgres-data:/var/lib/postgresql/data
       restart: unless-stopped

     zabbix-server:
       image: zabbix/zabbix-server-pgsql:latest
       container_name: zabbix-server
       environment:
         DB_SERVER_HOST: zabbix-db
         POSTGRES_USER: zabbix
         POSTGRES_PASSWORD: zabbixpass
         POSTGRES_DB: zabbix
       links:
         - zabbix-db
       ports:
         - "10051:10051"
       restart: unless-stopped

     zabbix-web:
       image: zabbix/zabbix-web-apache-pgsql:latest
       container_name: zabbix-web
       environment:
         DB_SERVER_HOST: zabbix-db
         POSTGRES_USER: zabbix
         POSTGRES_PASSWORD: zabbixpass
         POSTGRES_DB: zabbix
         ZBX_SERVER_HOST: zabbix-server
       links:
         - zabbix-server
       ports:
         - "8080:8080"
       restart: unless-stopped

     zabbix-web-service:
       image: zabbix/zabbix-web-service:latest
       container_name: zabbix-web-service
       links:
         - zabbix-server
       restart: unless-stopped
   ```

   Если порт 8080 уже занят, измените его на другой (например, `8081:8080`).

4. Сохраните и выйдите из редактора.

### Шаг 3: Запуск Zabbix

1. **Запустите контейнеры**:

   Введите команду для запуска Zabbix с PostgreSQL:
   
   ```bash
   docker compose up -d
   ```

2. **Проверьте статус контейнеров**:

   Убедитесь, что все контейнеры работают:
   
   ```bash
   docker ps
   ```

   Вы должны увидеть контейнеры `zabbix-db`, `zabbix-server`, `zabbix-web`, и `zabbix-web-service` в состоянии `Up`.

### Шаг 4: Доступ к веб-интерфейсу Zabbix

1. Откройте браузер и перейдите по следующему адресу:
   
   ```
   http://<IP вашего сервера>:8080
   ```

   Если вы изменили порт, используйте его вместо `8080`.

2. Войдите в систему, используя стандартные учетные данные:

   - Логин: `Admin`
   - Пароль: `zabbix`


![Авторизация в Zabbix](https://github.com/save-tsarev/sys-pattern-homework-8-03/blob/main/img/zadanie1_login.png)

## Задание 2

Шаг 1: Установка Zabbix Agent

1. **Установите Zabbix Agent на оба хоста**:
   - Один из хостов может быть вашим Zabbix Server.
   - Используйте следующие команды для установки Zabbix Agent на каждом из хостов (например, на Ubuntu):

   **Для Zabbix Server и второго хоста**:
   
   1. **Добавьте репозиторий Zabbix**:
   
      ```bash
      wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4+ubuntu20.04_all.deb
      sudo dpkg -i zabbix-release_6.0-4+ubuntu20.04_all.deb
      sudo apt update
      ```

   2. **Установите Zabbix Agent**:
   
      ```bash
      sudo apt install zabbix-agent
      ```

2. **Настройте Zabbix Agent**:
   
   После установки нужно настроить Zabbix Agent. Откройте файл конфигурации:

   ```bash
   sudo vim /etc/zabbix/zabbix_agentd.conf
   ```

   - В файле найдите строку `Server=` и укажите IP адрес вашего Zabbix Server:
     
     ```
     Server=IP_адрес_вашего_Zabbix_Server
     ```

   - Также настройте `ServerActive=` с тем же IP:
     
     ```
     ServerActive=IP_адрес_вашего_Zabbix_Server
     ```

   - В строке `Hostname=` укажите уникальное имя для каждого агента:
     
     ```
     Hostname=имя_вашего_хоста
     ```

3. **Запустите и включите агент**:

   Запустите и убедитесь, что агент автоматически запускается при загрузке:

   ```bash
   sudo systemctl restart zabbix-agent
   sudo systemctl enable zabbix-agent
   ```

4. **Проверьте статус агента**:

   Убедитесь, что Zabbix Agent работает корректно:

   ```bash
   sudo systemctl status zabbix-agent
   ```

### Шаг 2: Настройка агентов на Zabbix Server

1. **Добавьте хосты в Zabbix Server**:

   - Зайдите в веб-интерфейс Zabbix Server.
   - Перейдите в раздел **Configuration > Hosts**.
   - Нажмите на **Create host** и добавьте оба хоста:
     - Укажите имя хоста.
     - Укажите IP-адрес хоста.
     - Убедитесь, что выбран правильный шаблон для Zabbix Agent (например, **Template OS Linux by Zabbix agent**).

2. **Проверьте подключение**:

   После добавления хостов перейдите в раздел **Monitoring > Latest Data** и убедитесь, что данные с агентов поступают.

### Шаг 3: Сбор данных и проверка

1. **Проверьте данные в разделе "Latest Data"**:
   - Перейдите в **Monitoring > Latest Data** и убедитесь, что данные с обоих агентов поступают.

2. **Проверьте лог Zabbix Agent**:
   
   На каждом хосте выполните команду для просмотра лога агента:
   
   ```bash
   sudo tail -f /var/log/zabbix/zabbix_agentd.log

![Узлы сети](https://github.com/save-tsarev/sys-pattern-homework-8-03/blob/main/img/zadanie2_agents.png)

![Лог с первого агента](https://github.com/save-tsarev/sys-pattern-homework-8-03/blob/main/img/zadanie2_agentlog1.png

![Лог со второго агента](https://github.com/save-tsarev/sys-pattern-homework-8-03/blob/main/img/zadanie2_agentlog2.png

![Мониторинг 1 агента](https://github.com/save-tsarev/sys-pattern-homework-8-03/blob/main/img/zadanie2_agentmonitoring1.png

![Мониторинг 2 агента](https://github.com/save-tsarev/sys-pattern-homework-8-03/blob/main/img/zadanie2_agentmonitoring2.png
