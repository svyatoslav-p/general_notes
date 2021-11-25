.. |br| raw:: html

   <br />

LDAP
####

Глава посвящена настройке LDAP сервера на основе Samba4 (с поддержкой Active Directory и Domain Controller).
Система на которой разворачивается сервер CentOS 8 (на CentOS 7 проблемы из-за старых версий компонентов)
Во время установки желательно сразу прописать статический настройки сети и имя ПК **Hostname**
будет использовано как префикс домена вида ``Hostname.uonmap.com`` Базовая `статься по конфигурации <https://www.golinuxcloud.com/samba-active-directory/>`_

.. note::
   Для изменения имени хоста выполнить ``hostnamectl set-hostname dctr.uonmap.com``. Желательно
   давать имя хосту по шаблону ``name-pc.domain.prefix``

**Входные данные** IP по которому доступен хост ``192.168.1.108`` Имя домена ``samba.uonmap.com``


Подготовка системы
******************

#. Отключение **SELinux** ``setenforce 0`` ``nano /etc/selinux/config`` установить параметр ``SELINUX=disabled``
#. Отключение **Firewalld** ``systemctl stop firewalld`` ``systemctl disable firewalld``
#. Подключение репозиториев и обновление системы

   - ``yum -y install epel-release``
   - ``yum -y install dnf-plugins-core``
   - ``yum -y install wget mc nano``
   - ``dnf config-manager --set-enabled powertools``
   - ``yum update``
   - ``shutdown -r now``
   - 
Установка и запуск Samba4
*************************

#. Установка **Samba4**

   - Пакеты для сборки: ``yum -y install docbook-style-xsl gcc gdb gnutls-devel gpgme-devel jansson-devel`` 
     ``keyutils-libs-devel krb5-workstation libacl-devel libaio-devel libarchive-devel``
     ``libattr-devel libblkid-devel libtasn1 libtasn1-tools libxml2-devel libxslt lmdb-devel``
     ``openldap-devel pam-devel perl perl-ExtUtils-MakeMaker perl-Parse-Yapp popt-devel python3-cryptography``
     ``python3-dns python3-gpg python36-devel readline-devel rpcgen systemd-devel tar zlib-devel flex bison dbus-devel python3-markdown``
   - ``wget https://download.samba.org/pub/samba/samba-latest.tar.gz``
   - ``tar -xzvf samba-latest.tar.gz``
   - ``cd ./samba-4.xx.x/``
   - Проверка всей системы на готовность к сборке, на этом этапе возможно нужно 
     будет исправлять ошибки связанные с пакетами для сборки ``./configure``
   - ``make -j 2`` где ``2`` кол-во CPU для параллельной сборки
   - ``make install``

#. Преднастройка хоста

   - Прописать домены добавив строку в ``nano /etc/hosts`` вида ``192.168.1.108 samba samba.uonmap.com``
   - Установим переменную среды в ``nano /etc/profile`` прописав ``export PATH=/usr/local/samba/bin/:/usr/local/samba/sbin/:$PATH``

#. Запуск базовой конфигурации ``samba-tool domain provision --use-rfc2307 --interactive``

   .. code-block:: console
     :linenos:
   
      [root@samba]# samba-tool domain provision --use-rfc2307 --interactive
      Realm [EXAMPLE.COM]:  UONMAP.COM  <-- provide the realm name
      Domain [EXAMPLE]:  UONMAP  <-- provide the domain name
      Server Role (dc, member, standalone) [dc]:  dc   <-- Since we are configuring samba active directory, we use dc
      DNS backend (SAMBA_INTERNAL, BIND9_FLATFILE, BIND9_DLZ, NONE) [SAMBA_INTERNAL]:  SAMBA_INTERNAL   <-- We will let samba configure it's own DNS and zone files
      DNS forwarder IP address (write 'none' to disable forwarding) [192.168.1.108]:  8.8.8.8   <-- We will use google's dns
      Administrator password:   <-- Provide the Administrator user's password
      Retype password:

#. Настройка DNS

   - ``nano /etc/resolv.conf`` привести строку к виду ``nameserver 192.168.1.108``
   - Так же что бы DNS не сбрасывался установим его для сетевого адаптера 
     ``nano /etc/sysconfig/network-scripts/ifcfg-<имя>`` 
     приведем параметр к виду ``DNS1=192.168.1.108`` **так же необходимо прописать статические параметры сети**
     если этого не было сделано

#. Запуск сервера ``samba``.
#. Просмотр состояния ``testparm``
#. Базовая конфигурация **Kerberos** ``cp /usr/local/samba/private/krb5.conf /etc/krb5.conf``
#. Проверка **Kerberos** ``kinit Administrator`` вводим пароль и смотрим результат

Пример конфигураций
*******************

#. Файл ``/usr/local/samba/etc/smb.conf``

   .. literalinclude:: linux_files/ldap/smb.conf
     :language: ini

#. Файл ``/etc/krb5.conf``

   .. literalinclude:: linux_files/ldap/krb5.conf
     :language: ini

Опциональные действия
*********************

#. Добавить правила **firewalld** 

   - ``firewall-cmd --add-service={dns,ldap,ldaps,kerberos}``
   - ``firewall-cmd --add-port={389/udp,135/tcp,135/udp,138/udp,138/tcp,137/tcp,137/udp,139/udp,139/tcp,445/tcp,445/udp,3268/udp,3268/tcp,3269/tcp,3269/udp,49152/tcp}``

#. Добавить в автозапуск

   - Создадим сервис (проверьте корректность путей): **/etc/systemd/system/samba.service** :download:`Скачать <linux_files/ldap/samba.service>` 

   .. literalinclude:: linux_files/ldap/samba.service
     :language: ini

Решение проблем
***************

Упрощенная авторизация
======================

По умолчанию samba устанавливает максимальный уровень безопасности поэтому возникают сложности с авторизацией 
клиентов, можно активировать упрощенный режим (без сертификатов и прочего) добавив в ``nano /usr/local/samba/etc/smb.conf``
в раздел ``[global]`` параметры ``ldap server require strong auth = no`` и ``client ldap sasl wrapping = plain`` в таком режиме при авторизации может возникнуть ошибка ``AcceptSecurityContext error, data 52e, v1db1``
в таком случае логин необходимо указывать полностью например ``administrator@uonmap.com``

FAQ
***
Где посмотреть атрибуты Active Directory
========================================

Один из источников `атрибуты <http://www.selfadsi.org/user-attributes.htm/>`_

Как проверить доступность сервера
=================================

Выполнить команду на клиенте вида ``ldapsearch  -b "DC=uonmap,DC=com" -s sub -D``
``"CN=Administrator,CN=Users,DC=uonmap,DC=com" -H ldap://192.168.1.108:389 -w "Pass" -x "(uid=git_test@test.com)"``
