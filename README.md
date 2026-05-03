# Lesson-7-Package-management.-Software-distribution.

## Сборка RPM-пакета и создание репозитория

## Задание 
- создать свой RPM (можно взять свое приложение, либо собрать к примеру Apache с определенными опциями);
- cоздать свой репозиторий и разместить там ранее собранный RPM;
- реализовать это все либо в Vagrant, либо развернуть у себя через Nginx и дать ссылку на репозиторий.

  
```
#ставим пакеты которые будут необходимы
yum install -y wget rpmdevtools rpm-build createrepo yum-utils cmake gcc git nano

#качаем srpm пакет
[root@localhost rpm]# yumdownloader --source nginx
подключение репозитория appstream-source
подключение репозитория baseos-source
подключение репозитория extras-source
AlmaLinux 9 - AppStream - Source                                                                                                                             971 kB/s | 925 kB     00:00
AlmaLinux 9 - BaseOS - Source                                                                                                                                490 kB/s | 380 kB     00:00
AlmaLinux 9 - Extras - Source                                                                                                                                 12 kB/s | 8.7 kB     00:00
nginx-1.20.1-24.el9_7.2.alma.1.src.rpm                                                                                                                       1.3 MB/s | 1.1 MB     00:00

#устанавливаем пакет
 rpm -Uvh nginx*.src.rpm

#устанавливаем дополнения
yum-builddep nginx

cd /root
git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli
[root@localhost ~]# cd ngx_brotli/deps/brotli/
[root@localhost brotli]# mkdir out && cd out


#собираем модуль
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_C_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_CXX_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_INSTALL_PREFIX=./installed ..
-- The C compiler identification is GNU 11.5.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Build type is 'Release'
-- Performing Test BROTLI_EMSCRIPTEN
-- Performing Test BROTLI_EMSCRIPTEN - Failed
-- Compiler is not EMSCRIPTEN
-- Looking for log2
-- Looking for log2 - not found
-- Looking for log2
-- Looking for log2 - found
-- Configuring done (4.3s)
-- Generating done (0.0s)
CMake Warning:
  Manually-specified variables were not used by the project:

    CMAKE_CXX_FLAGS


-- Build files have been written to: /root/ngx_brotli/deps/brotli/out

# добавил опцию в файл
nginx_ldopts="$RPM_LD_FLAGS -Wl,-E"
if ! ./configure \
    --add-module=/root/ngx_brotli \
    --prefix=%{_datadir}/nginx \
    --sbin-path=%{_sbindir}/nginx \
    --modules-path=%{nginx_moduledir} \
    --conf-path=%{_sysconfdir}/nginx/nginx.conf \
    --error-log-path=%{_localstatedir}/log/nginx/error.log \
    --http-log-path=%{_localstatedir}/log/nginx/access.log \
    --http-client-body-temp-path=%{_localstatedir}/lib/nginx/tmp/client_body \
    --http-proxy-temp-path=%{_localstatedir}/lib/nginx/tmp/proxy \
    --http-fastcgi-temp-path=%{_localstatedir}/lib/nginx/tmp/fastcgi \
    --http-uwsgi-temp-path=%{_localstatedir}/lib/nginx/tmp/uwsgi \
    --http-scgi-temp-path=%{_localstatedir}/lib/nginx/tmp/scgi \
    --pid-path=/run/nginx.pid \
    --lock-path=/run/lock/subsys/nginx \
    --user=%{nginx_user} \
    --group=%{nginx_user} \
    --with-compat \


# собираем пакет
rpmbuild -ba nginx.spec -D 'debug_package %{nil}'

#проверка создания пакетов
 anaconda-ks.cfg  ngx_brotli  rpmbuild
[root@localhost ~]#  ll rpmbuild/RPMS/x86_64/
итого 2004
-rw-r--r--. 1 root root   37426 мая  3 18:47 nginx-1.20.1-24.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root 1028281 мая  3 18:47 nginx-core-1.20.1-24.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root  761815 мая  3 18:47 nginx-mod-devel-1.20.1-24.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   20641 мая  3 18:47 nginx-mod-http-image-filter-1.20.1-24.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   32231 мая  3 18:47 nginx-mod-http-perl-1.20.1-24.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   19364 мая  3 18:47 nginx-mod-http-xslt-filter-1.20.1-24.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   54971 мая  3 18:47 nginx-mod-mail-1.20.1-24.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   81570 мая  3 18:47 nginx-mod-stream-1.20.1-24.el9.2.alma.1.x86_64.rpm

ставим пакет
yum localinstall *.rpm
[root@localhost x86_64]# systemctl start nginx
[root@localhost x86_64]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Sun 2026-05-03 19:05:00 MSK; 19s ago
    Process: 70088 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 70089 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 70090 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 70091 (nginx)
      Tasks: 2 (limit: 10464)
     Memory: 7.2M (peak: 7.2M)
        CPU: 40ms
     CGroup: /system.slice/nginx.service
             ├─70091 "nginx: master process /usr/sbin/nginx"
             └─70092 "nginx: worker process"

мая 03 19:04:59 localhost.localdomain systemd[1]: Starting The nginx HTTP and reverse proxy server...
мая 03 19:05:00 localhost.localdomain nginx[70089]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
мая 03 19:05:00 localhost.localdomain nginx[70089]: nginx: configuration file /etc/nginx/nginx.conf test is successful
мая 03 19:05:00 localhost.localdomain systemd[1]: Started The nginx HTTP and reverse proxy server.


```
##Создаем репозиторий
```
#копируем пакеты
[root@localhost ~]# cp ~/rpmbuild/RPMS/noarch/* ~/rpmbuild/RPMS/x86_64/
[root@localhost ~]# cd ~/rpmbuild/RPMS/x86_64
createrepo /usr/share/nginx/html/repo/

#в  nginx.conf добавлем параметры
index index.html index.htm;
	autoindex on;

#проверяем
[root@localhost x86_64]# curl -a http://localhost/repo/
<html>
<head><title>Index of /repo/</title></head>
<body>
<h1>Index of /repo/</h1><hr><pre><a href="../">../</a>
<a href="repodata/">repodata/</a>                                          03-May-2026 16:07                   -
<a href="nginx-1.20.1-24.el9.2.alma.1.x86_64.rpm">nginx-1.20.1-24.el9.2.alma.1.x86_64.rpm</a>            03-May-2026 16:06               37426
<a href="nginx-all-modules-1.20.1-24.el9.2.alma.1.noarch.rpm">nginx-all-modules-1.20.1-24.el9.2.alma.1.noarch..&gt;</a> 03-May-2026 16:06                8577
<a href="nginx-core-1.20.1-24.el9.2.alma.1.x86_64.rpm">nginx-core-1.20.1-24.el9.2.alma.1.x86_64.rpm</a>       03-May-2026 16:06             1028281
<a href="nginx-filesystem-1.20.1-24.el9.2.alma.1.noarch.rpm">nginx-filesystem-1.20.1-24.el9.2.alma.1.noarch.rpm</a> 03-May-2026 16:06               10178
<a href="nginx-mod-devel-1.20.1-24.el9.2.alma.1.x86_64.rpm">nginx-mod-devel-1.20.1-24.el9.2.alma.1.x86_64.rpm</a>  03-May-2026 16:06              761815
<a href="nginx-mod-http-image-filter-1.20.1-24.el9.2.alma.1.x86_64.rpm">nginx-mod-http-image-filter-1.20.1-24.el9.2.alm..&gt;</a> 03-May-2026 16:06               20641
<a href="nginx-mod-http-perl-1.20.1-24.el9.2.alma.1.x86_64.rpm">nginx-mod-http-perl-1.20.1-24.el9.2.alma.1.x86_..&gt;</a> 03-May-2026 16:06               32231
<a href="nginx-mod-http-xslt-filter-1.20.1-24.el9.2.alma.1.x86_64.rpm">nginx-mod-http-xslt-filter-1.20.1-24.el9.2.alma..&gt;</a> 03-May-2026 16:06               19364
<a href="nginx-mod-mail-1.20.1-24.el9.2.alma.1.x86_64.rpm">nginx-mod-mail-1.20.1-24.el9.2.alma.1.x86_64.rpm</a>   03-May-2026 16:06               54971
<a href="nginx-mod-stream-1.20.1-24.el9.2.alma.1.x86_64.rpm">nginx-mod-stream-1.20.1-24.el9.2.alma.1.x86_64.rpm</a> 03-May-2026 16:06               81570
</pre><hr></body>
</html>
[root@localhost x86_64]#


#добавляем репозиторий в /etc/yum.repos.d

cat >> /etc/yum.repos.d/otus.repo << EOF
[otus]
name=otus-linux
baseurl=http://localhost/repo
gpgcheck=0
enabled=1
EOF

[root@localhost x86_64]# yum repolist enabled | grep otus
otus                                     otus-linux
[root@localhost x86_64]#


cd /usr/share/nginx/html/repo/
[root@localhost repo]# wget https://repo.percona.com/yum/percona-release-latest.noarch.rpm
--2026-05-03 19:16:51--  https://repo.percona.com/yum/percona-release-latest.noarch.rpm
Распознаётся repo.percona.com (repo.percona.com)… 49.12.125.205, 2a01:4f8:242:5792::2
Подключение к repo.percona.com (repo.percona.com)|49.12.125.205|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 28152 (27K) [application/x-redhat-package-manager]
Сохранение в: «percona-release-latest.noarch.rpm»

percona-release-latest.noarch.rpm 100%[==========================================================>]  27,49K  --.-KB/s    за 0s

2026-05-03 19:16:52 (186 MB/s) - «percona-release-latest.noarch.rpm» сохранён [28152/28152]



[root@localhost repo]# createrepo /usr/share/nginx/html/repo/
Directory walk started
Directory walk done - 11 packages
Temporary output repo path: /usr/share/nginx/html/repo/.repodata/
Preparing sqlite DBs
Pool started (with 5 workers)
Pool finished
[root@localhost repo]#
[root@localhost repo]#
[root@localhost repo]#
[root@localhost repo]#
[root@localhost repo]# yum makecache
AlmaLinux 9 - AppStream                                                                              8.5 kB/s | 4.2 kB     00:00
AlmaLinux 9 - BaseOS                                                                                 9.4 kB/s | 3.8 kB     00:00
AlmaLinux 9 - Extras                                                                                 7.7 kB/s | 3.3 kB     00:00
otus-linux                                                                                            74 kB/s | 7.2 kB     00:00
Создан кэш метаданных.
[root@localhost repo]# yum list | grep otus


percona-release.noarch                               1.0-33                             otus
[root@localhost repo]#
[root@localhost repo]#
[root@localhost repo]#









```
