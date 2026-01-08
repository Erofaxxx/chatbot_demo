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

## Способ 2: С использованием Nginx + SSL (для продакшена)

### Подготовка:

**ВАЖНО:** Перед началом убедитесь, что домен **chatbot.asktab.ru** указывает на IP **167.172.44.234** (добавьте A-запись в DNS).

### Установка и настройка:

1. **Подключитесь к серверу:**
   ```bash
   ssh root@167.172.44.234
   ```

2. **Установите Nginx и Certbot:**
   ```bash
   apt update
   apt install nginx certbot python3-certbot-nginx git -y
   ```

3. **Клонируйте проект из GitHub:**
   ```bash
   cd /var/www
   git clone https://github.com/Erofaxxx/chatbot_demo.git
   cd chatbot_demo
   ```

4. **Настройте Nginx (временная конфигурация для получения SSL):**
   ```bash
   nano /etc/nginx/sites-available/chatbot
   ```
   
   Вставьте:
   ```nginx
   server {
       listen 80;
       server_name chatbot.asktab.ru;
       
       root /var/www/chatbot_demo;
       index index.html;
       
       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```

5. **Активируйте конфигурацию:**
   ```bash
   ln -s /etc/nginx/sites-available/chatbot /etc/nginx/sites-enabled/
   nginx -t
   systemctl restart nginx
   ```

6. **Получите SSL сертификат:**
   ```bash
   certbot --nginx -d chatbot.asktab.ru
   ```
   
   Следуйте инструкциям:
   - Введите email для уведомлений
   - Согласитесь с условиями (Y)
   - Выберите редирект HTTP -> HTTPS (рекомендуется: 2)

7. **Проверьте финальную конфигурацию:**
   
   Certbot автоматически обновит конфигурацию. Проверьте:
   ```bash
   cat /etc/nginx/sites-available/chatbot
   ```
   
   Должно быть примерно так:
   ```nginx
   server {
       server_name chatbot.asktab.ru;
       
       root /var/www/chatbot_demo;
       index index.html;
       
       location / {
           try_files $uri $uri/ =404;
       }

       listen 443 ssl;
       ssl_certificate /etc/letsencrypt/live/chatbot.asktab.ru/fullchain.pem;
       ssl_certificate_key /etc/letsencrypt/live/chatbot.asktab.ru/privkey.pem;
       include /etc/letsencrypt/options-ssl-nginx.conf;
       ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
   }

   server {
       if ($host = chatbot.asktab.ru) {
           return 301 https://$host$request_uri;
       }
       
       listen 80;
       server_name chatbot.asktab.ru;
       return 404;
   }
   ```

8. **Перезапустите Nginx:**
   ```bash
   nginx -t
   systemctl restart nginx
   ```

9. **Настройте автообновление SSL:**
   
   Certbot автоматически создает задачу для обновления. Проверьте:
   ```bash
   systemctl status certbot.timer
   ```
   
   Или протестируйте обновление:
   ```bash
   certbot renew --dry-run
   ```

10. **Откройте в браузере:**
    ```
    https://chatbot.asktab.ru
    ```

### Обновление сайта из GitHub:

Для обновления сайта в будущем:
```bash
cd /var/www/chatbot_demo
git pull origin main
systemctl reload nginx
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

После запуска откройте в браузере:
- **Python:** http://167.172.44.234
- **Nginx с SSL:** https://chatbot.asktab.ru

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

Если настроен файервол (ufw), откройте необходимые порты:
```bash
ufw allow 80/tcp    # HTTP
ufw allow 443/tcp   # HTTPS
ufw allow 22/tcp    # SSH (если еще не открыт)
ufw reload
```

## Проверка SSL

Проверьте SSL сертификат:
```bash
curl -I https://chatbot.asktab.ru
```

Или на сайте: https://www.ssllabs.com/ssltest/analyze.html?d=chatbot.asktab.ru
