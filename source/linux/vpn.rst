.. |br| raw:: html

   <br />

WireGuard
#########

Ниже описаны особенности настройки VPN сервера **WireGuard**. Так как в открытых источниках довольно
много информации, описаны только особенности.

Настройка сервера
*****************

Рассмотрим настройку на примере Ubuntu запущенной на VPS машине сервиса **Oracle Cloud**.

#. Добавим правило для **Firewall**. В настройках облачной машины переходим  :guilabel:`Networking` -> :guilabel:`Virtual Cloud Networks`  -> :guilabel:`NAME_VCN`
   и добавим порт на котором будет работать **WireGuard** (по умолчанию ``51820``, но в рамках безопасности можно изменить например ``52820``). Если настравивате
   на локальной машине нужно тоже самое сделать на маршрутизаторе.

   .. figure:: linux_image/vpn/oracle_rules_wg.png

#. Установку **WireGuard** выполним через скрипт `WireGuard installer <https://github.com/angristan/wireguard-install/>`_ 

   .. note:: 
      Можно установить любым удобным способом, но с **Oracle Cloud** настроить корректную работу удалось только с использованием этого скрипта
      Так же работает со скриптом **PiVPN**, но с ним не получилось настроить доступность клиентов между собой, эти трудности связаны именно с 
      **Oracle Cloud** вероятнее всего необходимо что то открыть еще.

   #. Копируем скрипт ``curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh``
   #. Делаем его исполняемым ``chmod +x wireguard-install.sh``
   #. Запускаем ``sudo ./wireguard-install.sh``
   
   .. figure:: linux_image/vpn/wg_angristan_script.png

#. Остановим интерфейс ``sudo wg-quick down wg0`` и добавим правила маршрутизации

   #. Открываем конфигурацию ``sudo nano /etc/wireguard/wg0.conf``. Редактируем поля ``PostUp`` и ``PostDown``

      .. code-block:: ini
   
         # ВНИМАНИЕ! замените адаптер enp0s3, wg0 и порт 52820 на свой
         # Для работы на Oracle Cloud
         PostUp = iptables -I FORWARD -i enp0s3 -o wg0 -j ACCEPT; iptables -I FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE; ip6tables -A FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE; iptables -I INPUT -i enp0s3 -p udp --dport 52820 -m state --state NEW,ESTABLISHED -j ACCEPT
         PostDown = iptables -D FORWARD -i enp0s3 -o wg0 -j ACCEPT; iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o enp0s3 -j MASQUERADE; ip6tables -D FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -D POSTROUTING -o enp0s3 -j MASQUERADE; sudo iptables -D INPUT -i enp0s3 -p udp --dport 52820 -m state --state NEW,ESTABLISHED -j ACCEPT

         # В большенстве случаев достаточно этих правил
         PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
         PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o enp0s3 -j MASQUERADE

#. Запускаем интерфейс интерфейс ``sudo wg-quick up wg0``        

#. Добавим в автозапуск ``sudo systemctl enable wg-quick@wg0.service``

Далее если повторно вызвать скрипт ``sudo ./wireguard-install.sh`` можно добавить клиента.

FAQ
***

Перенапрявлять весь трафик через VPN
====================================

Если нужно отправлять весь трафик через туннел (нетолько трафик хоста), то в конфигурации клиента заменить строку 
``AllowedIPs = 0.0.0.0/0, ::/0`` на ``AllowedIPs = 0.0.0.0/1, 128.0.0.0/1, ::/1, 8000::/1``

Как запустить более одного интерфейса wg*
=========================================

Для последующих интерфейсов все делается аналогично. **Очень важный момент** только для одной кофигурации
можно задать перенапрявлять весь трафик ``AllowedIPs = 0.0.0.0/0, ::/0`` на остальных должны быть прописаны конкретные адреса.
Например второй интерфейс имеет адрес ``10.8.0.4/32`` и используется как клиент тогда параметр ``AllowedIPs = 10.8.0.1/32``
Это означает что все запросы хоста на адрес ``10.8.0.1`` будут идти через этот тунель (адреса можно указать любые)