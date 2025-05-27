# `.htaccess` и `robots.txt` для Joomla 

Заготовка файлов для использования

## .htaccess

**htaccess.txt** -- Стандартный файл 

**htaccess-rus.txt** -- Стандартный файл с переведенными коментариями на русский язык

**htaccess-mod.txt** -- Модифицированный файл с переведенными коментариями на русский язык

### htaccess-mod

Добавлено

- Блокировка пустых User-Agent
```apache
    RewriteCond %{HTTP_USER_AGENT} ^$ [OR]
    RewriteCond %{HTTP_USER_AGENT} ^\s+$ [OR]
    RewriteCond %{HTTP_USER_AGENT} ^- [OR] 
    RewriteCond %{HTTP_USER_AGENT} ^-$
    RewriteRule ^ - [F,L]
```

- Блокировка вредоносных ботов и сканеров
```apache
    RewriteCond %{HTTP_USER_AGENT} (ALittle\sClient|keys-so-bot|Go-http-client|masscan|nikto|sqlmap|nmap|scan|wpscan) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (acunetix|netsparker|nessus|openvas|metasploit|burpsuite|dirbuster|havij) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (zmeu|sqlninja|hydra|weevely|caidao|adminer|phpmyadmin) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (winhttp|HTTrack|clshttp|archiver|loader|email|harvest) [NC,OR]
    RewriteCond %{HTTP_USER_AGENT} (extract|grab|miner|python-requests) [NC]
    RewriteRule ^ - [F,L]
```

- Блокировка доступа к системным файлам
```apache
    RewriteCond %{REQUEST_FILENAME} -f [OR]
    RewriteCond %{REQUEST_FILENAME} -d
    RewriteRule ^(.*/)?(\.|backup|dump|sql|config\.php|composer\.(json|lock)|package\.json)/ - [F,L,NC]
```

- Защищает от атак типа **RFI** (Remote File Inclusion) и **LFI** (Local File Inclusion), которые могут использоваться для выполнения произвольного кода.
```apache
    RewriteCond %{QUERY_STRING} (https?|ftp):(\/|%2F){2,} [NC,OR]
    RewriteCond %{QUERY_STRING} ^=(https?|ftp|php|file|data):? [NC,OR]
    RewriteCond %{QUERY_STRING} (\.\.\/|\.\.%2f|%2e%2e%2f|\.%2f) [NC]
    RewriteRule .* - [F,L]
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

- Разрешенные боты
```apache
    User-agent: Googlebot
    User-agent: Googlebot-Image
    User-agent: Googlebot-News
    User-agent: Bingbot
    User-agent: MSNBot 
    User-agent: DuckDuckBot
    User-agent: Yandex
    User-agent: YandexBot
    User-agent: YandexAccessibilityBot
    User-agent: YandexMobileBot
    User-agent: YandexMetrika
    User-agent: YandexImages
    Allow: /
```

- Правил для статических файлов
```apache
    Allow: /*.jpg$
    Allow: /*.png$
    Allow: /*.gif$
    Allow: /*.webp$
    Allow: /*.svg$
    Allow: /*.woff$
    Allow: /*.woff2$
    Allow: /*.ttf$
    Allow: /*.eot$
```

- Блокирует доступ к критическим файлам идиректориям:
```apache
    Disallow: /configuration.php
    Disallow: /.htaccess
    Disallow: /error_log
    Disallow: /index.php?option=com_search
    Disallow: /component/mailto/
```

