# `.htaccess` и `robots.txt` для Joomla 

Заготовка файлов для использования

## .htaccess

**htaccess.txt** -- Стандартный файл 

**htaccess-rus.txt** -- Стандартный файл с переведенными коментариями на русский язык

**htaccess-mod.txt** -- Модифицированный файл с переведенными коментариями на русский язык

### htaccess-mod

Добавлено

- Блокировка запросов без User-Agent или с подозрительными значениями
```apache
    RewriteCond %{HTTP_USER_AGENT} ^$ [OR]
    RewriteCond %{HTTP_USER_AGENT} ^-$ [OR]
    RewriteCond %{HTTP_USER_AGENT} ^[\s\t\r\n]+$
    RewriteRule ^ - [F,L]
```

- Блокировка вредоносных ботов и сканеров
```apache
    RewriteCond %{HTTP_USER_AGENT} (masscan|nikto|sqlmap|nmap|wpscan|dirbuster|havij) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (acunetix|netsparker|nessus|openvas|metasploit) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (burpsuite|weevely|caidao|adminer|hydra|zmeu) [NC]
    RewriteRule ^ - [F,L]
```

- Блокировка прямого доступа к конфигурационным файлам
```apache
    RewriteRule ^(config\.php|composer\.(json|lock)|package\.json|\.env)$ - [F,L,NC]
```

- Блокировка SQL-инъекций
```apache
    RewriteCond %{QUERY_STRING} \b(unions?|drop|delete|insert|update|alter|truncate|exec|execute|load_file|outfile|benchmark)\s+([a-zA-Z0-9_\*]+) [NC,OR]
    RewriteCond %{QUERY_STRING} ([;'"<>])+\s*(unions?|select|drop|delete|insert|update|exec|execute) [NC]
    RewriteRule ^ - [F,L]
```

- Блокировка XSS и JS-вставок
```apache
    RewriteCond %{QUERY_STRING} (eval$|base64_decode|javascript:|on(error|load|click|submit)=|alert$|script|iframe|frame|document\.cookie|window\.location|expression\s*$) [NC]
    RewriteRule ^ - [F,L]
```

- Блокировка LFI/RFI
```apache
    RewriteCond %{QUERY_STRING} (\.\./|%2e%2e|%00) [NC,OR]
    RewriteCond %{QUERY_STRING} ^=(https?|ftp|php|file|data): [NC,OR]
    RewriteCond %{QUERY_STRING} ^\w+=[a-z0-9+.-]+:// [NC,OR]
    RewriteCond %{QUERY_STRING} (etc/passwd|win\.ini|boot\.ini) [NC]
    RewriteRule ^ - [F,L]
```

- Примеры, которые можно легко раскомментировать.

    - настройки распространенных перенаправлений
    ```apache
    ## Редирект с www на без www
    # RewriteCond %{HTTP_HOST} ^www\.(.*)$ [NC]
    # RewriteRule ^(.*)$ https://%1/$1 [R=301,L]

    ## Редирект с http на https
    # RewriteCond %{HTTPS} off
    # RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]

    ## Редирект sitemap.xml → sitemap
    # RewriteRule ^sitemap\.xml$ /sitemap [L,R=301]
    ```

    - Блокировка IP-адресов
    ```apache
    # <IfModule mod_authz_core.c>
    #   <RequireAll>
    #       Require all granted
    #
    #   # Запретить доступ для указанных IP 
    #       Require not ip 123.45.67.89
    #
    #   # Блокировка всей подсети
    #       Require not ip 10.0.0.0/24  
    #   </RequireAll>
    # </IfModule>
    ```
### Редирект с http на https для всего сайта для хостинга Timeweb

У хостинга Timeweb 3 разных варианта которые нужно подбирать

#### Вариант 1
```apache
    RewriteCond %{SERVER_PORT} !^443$
    RewriteRule .* https://%{SERVER_NAME}%{REQUEST_URI} [R=301,L]
```
#### Вариант 2
```apache
    RewriteCond %{HTTPS} =on
    RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI} [QSA,L]
    RewriteCond %{HTTPS} off
    RewriteCond %{HTTP:X-Forwarded-Proto} !https
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

#### Вариант 3
```apache
    RewriteEngine On
    RewriteCond %{SERVER_PORT} !^443$
    RewriteCond %{REQUEST_URI} =/page.php
    RewriteRule .* https://%{SERVER_NAME}%{REQUEST_URI} [R,L]
```

> #### ⚠ ВНИМАНИЕ!
> Условия редирект записывать в блоке IfModule, дабы избежать ошибок при выполнении файла htaccess.
```apache
    <IfModule mod_rewrite.c>

    </IfModule>
```
---

## robots.txt

**robots.txt** -- Стандартный файл 

**robots-mod.txt** -- Модифицированный файл с переведенными коментариями на русский язык

### robots-mod.txt

После общения с более опытными ребятами решил оставить оригинальный вариант
только добавил указание карты сайта

```apache
   Sitemap: https://yourdomain.com/sitemap.xml 
```