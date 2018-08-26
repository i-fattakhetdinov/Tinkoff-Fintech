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

---

#### Задание 2

Поключаем репу epel и базовые пакеты сборки 
```bash
sudo yum install -y epel-release yum-utils rpm-build rpmdevtools
```

Ставим средств разработки 
```bash
sudo yum groupinstall -y 'Development Tools'
```

Скачиваем `nginx.src.rpm`

```bash
$ cd ~
$ yumdownloader --source nginx
```
Устанавливаем
```bash
$ rpm -i nginx-1.14.0-1.el7_4.ngx.src.rpm
```

Создаём структуру каталогов для rpm-пакета
```bash
$ rpmdev-setuptree
```
Устанавливаем зависимости для сборки пакета nginx
```bash
$ sudo yum-builddep rpmbuild/SPECS/nginx.spec
```
Убеждаемся, что Nginx Успешно собирается без наших правок
```bash
$ rpmbuild -ba rpmbuild/SPECS/nginx.spec
```

Клонируем VTS-модуль
```bash
$ git clone https://github.com/vozlt/nginx-module-vts.git
```

Упаковываем в архив
```bash
$ tar -czf rpmbuild/SOURCES/nginx-module-vts.tar.gz nginx-module-vts/
```
Правим spec-файл, чтобы привести к виду `nginx.spec`
```bash
vim rpmbuild/SPECS/nginx.spec 
```

Проверяем что всё ок вставили
```bash
rpmbuild -bp rpmbuild/SPECS/nginx.spec
```
Добавим сборку нашего модуля (статически) в nginx
```bash
$ vim rpmbuild/SPECS/nginx.spec
```
Запускаем сборку
```bash
$ rpmbuild -ba rpmbuild/SPECS/nginx.spec
```

Устанавлиаем наш собранный пакет:
```bash
sudo yum localinstall rpmbuild/RPMS/x86_64/nginx-1.14.0-1.el7_4.ngx_ft.x86_64.rpm
sudo systemctl restart nginx
```
Проверяем работоспособность
```bash
$ curl test-1
Yohoo, it works
```

Чтобы получить метрики редактируем `1-test.conf` и выполняем
```bash
$ curl test-1/status
```
