[На главную](/){ .md-button }
##

# Публикуем сайт на vsp сервере. nginx. ssl


!!! note "steps"
    1. Купить сервер
    2. Настроить ресурсные записи
    3. Рассмотрим публикацию сайта (статические файлы) на уделенный сервер. 
    4. Настроим nginx 





---
## #Ресурсные записи -> домен -> сервер
после покупки сервера мы связываем доменное имя с нашим сервером, данные по работе с сервером приходят на указанную вами почту или их можно взять в личном кабинете сервиса в котором покупался сервер
![ip](assets/img/mainip.png)

* `настроить ресурсные записи в regru` - Позовлит соединить доменное имя с купленным сервером

заходим в регру, выбираем доменное имя, изменить ресурсные записи, добавляем ip нашего сервера: 

* `в запись @` 
* `и в www`
![server](assets/img/resursdata.png)




---
## ssh

### Сгенерировать ключ
```
ssh-keygen -t rsa
```

### Вывести ключ в терминал
```
cat ~/.ssh/id_rsa.pub
```

### Скопировать ключ на удаленный сервер
После ввода команды, введите пароль (не будет отображаться) и нажмите ENTER. Утилита скопирует содержимое открытого ключа (~/.ssh/id_rsa.pub) на удаленный сервер в файл authorized_keys.
```
ssh-copy-id root@00.000.000.000
```

### Если соединение быстро обрывается можно использовать эту команду
```
ssh -o ServerAliveInterval=60 root@00.000.000.000
```

### Зайти на сервер по ssh
```
ssh root@00.000.000.000
```



---

## Обновите список пакетов
    sudo apt update



---


## Учетные записи

### меняем пароль пользователя ROOT
при вво­де сим­волы не отоб­ража­ются — нет ни букв, ни цифр, ни звез­дочек, это нор­маль­но, вве­ди новый пароль и наж­ми Enter
```
passwd
```

---

### добавить учетную запись
следуйте инструкциям, чтобы установить пароль и заполнить другую информацию
```
adduser coder
```

### добавить права суперпользователя для пользователя coder
добавьте пользователя coder в группу sudo, используя команду
```
usermod -aG sudo coder
```

### проверьте, что пользователь добавлен в группу sudo, выполните команду
```
groups coder
```



---
## #Установка программ

### Обновите список пакетов
    sudo apt update

### MCeditor
Midnight Commander — один из файловых менеджеров с текстовым интерфейсом типа Norton Commander для UNIX-подобных операционных
```
sudo apt install mc
```

### Установите git
* `git --version` - проверить установлен ли git
```
git --version
```

```
sudo apt install git
```

### Установите Nginx
    sudo apt install nginx

### Полезные команды Nginx
```
sudo nginx -t
```

```
sudo systemctl start nginx
```

```
sudo systemctl restart nginx
```

```
sudo systemctl stop nginx
```

```
sudo service nginx reload
```

```
sudo service nginx status
```



---
## Склонировать проект 
перейти в папку www
```
cd /var/www/
```
склонировать сюда проект

```
git clone gitLink
```

* `например` - шаблон html для тестов
```
git clone git@github.com:pavelcity/furni-example.git
```



### проблема с клонированием / Permission denied
* `Ошибка "Permission denied" при клонировании репозитория Git может возникнуть из-за недостаточных прав доступа к директории, в которую вы пытаетесь склонировать проект. В данном случае, вам не хватает прав на запись в директорию, где вы пытаетесь выполнить клонирование.`
* `Для решения этой проблемы вам следует убедиться, что у вас есть достаточные права доступа к директории /var/www/ или создать новую директорию, куда вы сможете клонировать проект. Вы можете выполнить следующие шаги:`

Убедитесь, что у вас есть права на запись в директорию /var/www/. Для этого выполните команду:
```
ls -ld /var/www/

```

Если у вас нет прав на запись в эту директорию, выполните команду для изменения прав доступа:
```
sudo chmod o+w /var/www/
```





## Настройка nginx.	Добавить файл в nginx каталог 
обычно по названию папки вашего проекта
```
sudo nano /etc/nginx/sites-available/urni-example
```
```
sudo mcedit /etc/nginx/sites-available/urni-example
```


### добавляем в файл /etc/nginx/sites-available/urni-example эти данные (ставим свой ip и домен в строку server_name)
``` linenums="1"

server {
    listen 80;
    listen [::]:80;

    root /var/www/urni-example;

    index index.html index.htm index.nginx-debian.html;

    server_name app-exmpl.ru www.app-exmpl.ru 79.174.83.12;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        try_files $uri $uri/ =404;
    }


}

server {
    listen 443 ssl;
    server_name app-exmpl.ru www.app-exmpl.ru;

    root /var/www/urni-example;

    index index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

server {
    listen 80 ;
    listen [::]:80 ;
    server_name app-exmpl.ru www.app-exmpl.ru 79.174.83.12;

    return 301 https://$host$request_uri;
}


```



### Создаем симлинк
```
sudo ln -s /etc/nginx/sites-available/urni-example /etc/nginx/sites-enabled/
```

### Перегрузите nginx
```
sudo systemctl restart nginx
```

### проверим конфигурацию nginx
```
sudo nginx -t
```


---


## #Letsencrypt - ssl

Подключение сертификата ssl
```
sudo apt install certbot python3-certbot-nginx
```
```
sudo certbot
```

!!! note "выпуск ssl"
    1. вводим свой емейл
    2. `y` - соглашаемся с инструкцией
    3. `выпустить ssl сертификат` - вводим свой домен, например - app-exmpl.ru
    4. или - по настройке nginx будет преложено выпостить ssl для домена с www  и без www
    5. `1 2` - вводим цифры через пробел