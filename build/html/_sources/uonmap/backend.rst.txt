.. |br| raw:: html

   <br />

Backend
#######

Заметки по проекту **Smart Safe School** в части backend

Обновление ключа firebase
*************************

Переходим по `ссылке <https://console.firebase.google.com/u/1/project/smart-safe-school-add85/settings/serviceaccounts/adminsdk>`_
получаем новый ключ нажатием на :guilabel:`Generate new private key` 

   .. figure:: backend/firebase_key.png

подменяем файл ``smartdevice-server-credentials/gap-credentials.json``

.. attention::

   В файле ``gap-credentials.json`` содержится строка вида ``"universe_domain": "googleapis.com"`` и с этой строкой почему то
   бэкенд не хочет принимать этот ключ - поэтому эту строку удаляем из файла ключа. В логах ошибок может не быть, но пуши могут не приходить
   из-за этой строки

