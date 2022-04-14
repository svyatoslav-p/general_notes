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