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



```
