#### Порядок действий

Стартуем сервис `nginx`
```bash
$ sudo systemctl start nginx
```
Включаем старт `nginx` при загрузке ОС
```bash
$ sudo systemctl enable nginx.service
```

Добавляем минимальный конфиг по пути `/etc/nginx/conf.d/1-test.conf`

```
server {
        server_name  test-1;
        listen       80;
        root /var/www/test-1;
}
```

Разместим тестовую индекс страницу по адресу:
```bash
$ cat /var/www/test-1/index.html
Yohoo, it works
```

Добавляем прав SELinux на каталоги, ианче у пользователя Nginx Не будет прав читать файлы
```bash
$ sudo chcon -Rt httpd_sys_content_t /var/www
```

Перезагружаем nginx, чтобы применить конифгурацию
```bash
$ sudo nginx -s reload
```
Сделаем так, чтобы запросы с этой же машины по имени `test-1` и `test-2` приходили на неё же.
Для этого в файл `/etc/hosts` добавим `127.0.0.1 test-1 test-2` и в файле `/etc/nginx/conf.d/1-test.conf`
заменим `server_name  test-1;` на `server_name  test-1 test-2;`

Проверяем, что всё работает:
```bash
$ curl test-1
Yohoo, it works
```
nginx подтягивает наш конфиг, потому что в его конфиге `/etc/nginx/nginx.conf` прописано `include /etc/nginx/conf.d/*.conf` 
а наш конфиг как раз лежит в папке `/etc/ngix/conf.d` и имеет вид `*.conf`

Тестируем дальше:
```bash
$ curl test-2
Yohoo, it works
```
* Подправить конфиг `/etc/nginx/conf.d/default.conf` таким образом, чтобы:
  * Он был конфигом по умолчанию (отдавался на все явно не указанные server_name на 80-м порту)
  * Минимальным (убрать из него лишние комментарии и секции)
  * На все запросы отдавался бы код `404` с телом `'No <имя сервера> server config found` (подсказка см. `return`)

Убеждаемся что теперь запрос `curl test-2` не отдаёт нам первый попавшийся сайт.
