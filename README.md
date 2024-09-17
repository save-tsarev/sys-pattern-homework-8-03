*Задание 1

Шаг 1: Подготовка Docker и Docker Compose

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

3. После успешного входа сделайте скриншот, чтобы прикрепить его к вашему домашнему заданию.

### Шаг 5: Заключительные шаги

1. **Сохраните изменения** в вашем проекте, если вы работали в репозитории:

   ```bash
   git add .
   git commit -m "Добавил решение с настройкой Zabbix через Docker"
   git push origin main
   ```

2. **Отправьте ссылку на ваш репозиторий** для проверки домашнего задания.

Если возникнут дополнительные вопросы или ошибки в процессе, дайте знать, и я помогу вам!
