Ubuntu
######

Установка AnyDesk 22.04
***********************

#. Переключаемся с **wayland** на **x11**. Для этого открываем ``sudo nano /etc/gdm3/custom.conf``
   и раскомментируем строку ``WaylandEnable=false``, перезагружаемся и проверяем ``echo $XDG_SESSION_TYPE``
#. Установим anydesk любым из способов (необходимо помнить что с версии 7.0 он уже не бесплатен)
#. Установим необходимый пакет для anydesk ``wget http://ftp.us.debian.org/debian/pool/main/p/pangox-compat/libpangox-1.0-0_0.0.2-5.1_amd64.deb``
   ``sudo apt install ./libpangox-1.0-0_0.0.2-5.1_amd64.deb``
