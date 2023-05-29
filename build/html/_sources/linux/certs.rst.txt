.. |br| raw:: html

   <br />

Сертификаты
###########


Получение Wildcard certbot
**************************

`Первоисточник <https://codex.so/wildcard-ssl-certificate-by-let-s-encrypt/>`_

 #. Установить и настроить **certbot**
 #. Выполнить в терминале (на примере домена **uonmap.com**)

    .. code-block:: console

       [root@prod archive]# DOMAIN=uonmap.com
       [root@prod archive]# certbot certonly --manual -d *.$DOMAIN -d $DOMAIN --agree-tos --manual-public-ip-logging-ok --preferred-challenges dns-01 --server https://acme-v02.api.letsencrypt.org/directory --register-unsafely-without-email --rsa-key-size 4096
 #. На этапе когда будет сказано добавить DNS TXT

    .. code-block:: console

       Please deploy a DNS TXT record under the name
       _acme-challenge.uonmap.com with the following value:
       
       xW0Mzm1RcfF4V-v8yZZS0zz3Bl2HvWPYUHCuzYocsMU
       
       Before continuing, verify the record is deployed.
       - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
       Press Enter to Continue
       
       - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
       Please deploy a DNS TXT record under the name
       _acme-challenge.uonmap.com with the following value:
       
       BR1iP57B9SSST15RQcymJy-Bj52yD_59V2JcFt2IbMo
       
       Before continuing, verify the record is deployed.
       (This must be set up in addition to the previous challenges; do not remove,
       replace, or undo the previous challenge tasks yet. Note that you might be
       asked to create multiple distinct TXT records with the same name. This is
       permitted by DNS standards.)
       
       - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
       Press Enter to Continue

   добавим в личном кабинет своего DNS провайдера две записи с параметрами из вывода. 
   **Важно не нажимать Enter** пока домены не добавятся. Для проверки можно открыть новую сессию
   и выполнить команду ниже. 

   .. code-block:: console

       [root@prod archive]# host -t txt _acme-challenge.uonmap.com

       _acme-challenge.example.com descriptive text "xW0Mzm1RcfF4V-v8yZZS0zz3Bl2HvWPYUHCuzYocsMU"
       _acme-challenge.example.com descriptive text "BR1iP57B9SSST15RQcymJy-Bj52yD_59V2JcFt2IbMo"

   При удачном ответе можно снова вернуться на предыдущую сессию и нажать
   Enter.

.. attention::

   Сертификат из примера выше подходит только для доменов 3-го уровня. Для работы 
   с доменами 4-го и далее нужно выпускать другой сертификат либо добавить альтернативные dns в текущий

Для домена 4-го уровня ``*.sub.uonmap.com`` команда может выглядеть так: 

.. code-block:: console

   [root@prod archive]# DOMAIN=uonmap.com
   [root@prod archive]# certbot certonly --manual -d *.$DOMAIN -d $DOMAIN -d *.sub.$DOMAIN -d sub.$DOMAIN --agree-tos --manual-public-ip-logging-ok --preferred-challenges dns-01 --server https://acme-v02.api.letsencrypt.org/directory --register-unsafely-without-email --rsa-key-size 4096

Конвертирование в jks
*********************

#. Выполнить скрипт :download:`Скачать <linux_files/cert/pem2jks.sh>` 

   .. literalinclude:: linux_files/cert/pem2jks.sh
     :language: bash

Покупка сертификата
*******************

Получить fullchain сертификат
=============================

После покупки сертификата, продавец может прислать архив в котором будут файлы вида:

1. Server Certificate - ``example.com.crt``
2. Intermediate CA Certificate - ``SectigoRSADomainValidationSecureServerCA.crt``
3. Intermediate CA Certificate - ``USERTrustRSAAAACA.crt``
4. Root CA Certificate - ``AAACertificateServices.crt``

Для получения полноценного сертификата не достаточно использовать ``example.com.crt``
Нужно их объединить в один:
``cat example.com.crt SectigoRSADomainValidationSecureServerCA.crt USERTrustRSAAAACA.crt AAACertificateServices.crt > ca-bundle.crt``


