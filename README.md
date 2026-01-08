# Инструкция по запуску сайта с чатботом

## Способ 1: Быстрый запуск с Python (рекомендуется)

### На вашем сервере выполните:

1. **Подключитесь к серверу:**
   ```bash
   ssh root@167.172.44.234
   ```

2. **Создайте папку и загрузите файл:**
   ```bash
   mkdir -p /var/www/chatbot
   cd /var/www/chatbot
   ```

3. **Скопируйте файл index.html на сервер:**
   
   С вашего локального компьютера выполните:
   ```bash
   scp index.html root@167.172.44.234:/var/www/chatbot/
   ```

4. **Запустите простой веб-сервер на Python:**
   ```bash
   cd /var/www/chatbot
   python3 -m http.server 80
   ```

5. **Откройте в браузере:**
   ```
   http://167.172.44.234
   ```

---

## Способ 2: С использованием Nginx (для продакшена)

### Установка и настройка:

1. **Подключитесь к серверу:**
   ```bash
   ssh root@167.172.44.234
   ```

2. **Установите Nginx:**
   ```bash
   apt update
   apt install nginx -y
   ```

3. **Создайте папку для сайта:**
   ```bash
   mkdir -p /var/www/chatbot
   ```

4. **Скопируйте файл на сервер:**
   
   С вашего локального компьютера:
   ```bash
   scp index.html root@167.172.44.234:/var/www/chatbot/
   ```

5. **Настройте Nginx:**
   ```bash
   nano /etc/nginx/sites-available/chatbot
   ```
   
   Вставьте:
   ```nginx
   server {
       listen 80;
       server_name 167.172.44.234;
       
       root /var/www/chatbot;
       index index.html;
       
       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```

6. **Активируйте конфигурацию:**
   ```bash
   ln -s /etc/nginx/sites-available/chatbot /etc/nginx/sites-enabled/
   nginx -t
   systemctl restart nginx
   ```

7. **Откройте в браузере:**
   ```
   http://167.172.44.234
   ```

---

## Способ 3: С использованием Docker (самый простой)

1. **Подключитесь к серверу:**
   ```bash
   ssh root@167.172.44.234
   ```

2. **Установите Docker (если не установлен):**
   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sh get-docker.sh
   ```

3. **Создайте папку и скопируйте файл:**
   ```bash
   mkdir -p /var/www/chatbot
   ```
   
   С локального компьютера:
   ```bash
   scp index.html root@167.172.44.234:/var/www/chatbot/
   ```

4. **Запустите контейнер с Nginx:**
   ```bash
   docker run -d -p 80:80 -v /var/www/chatbot:/usr/share/nginx/html nginx:alpine
   ```

5. **Откройте в браузере:**
   ```
   http://167.172.44.234
   ```

---

## Проверка работы

После запуска откройте в браузере: **http://167.172.44.234**

Вы должны увидеть страницу с чатботом в правом нижнем углу.

---

## Остановка сервера

### Python:
Нажмите `Ctrl + C` в терминале

### Nginx:
```bash
systemctl stop nginx
```

### Docker:
```bash
docker ps  # найдите CONTAINER ID
docker stop <CONTAINER_ID>
```

---

## Примечания

- Убедитесь, что порт 80 открыт в файерволе сервера
- Для проверки порта: `netstat -tulpn | grep :80`
- Если порт занят, используйте другой (например, 8080) и меняйте в командах

---

## Безопасность

Если настроен файервол (ufw):
```bash
ufw allow 80/tcp
ufw reload
```
