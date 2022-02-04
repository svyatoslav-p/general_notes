.. |br| raw:: html

   <br />

Kotlin
######

Ниже описаны особенности работы в **Intellij Idea Ultimate**

Среда разработки
****************

Установка 
=========

Следовать инструкции на `странице <https://snapcraft.io/install/intellij-idea-ultimate/manjaro>`_

Активация 
=========

#. Переходим на `страницу <https://jetbra.in/s>`_ и скачиваем файл ``ja-netfilter-all.zip``

   .. figure:: kotlin/ja_netfilter_all.png

#. Распакуем в любой каталог, но путь недолжен содержать пробелы и кириллицу. Например ``/home/svyatoslav/system/jetbrains/ja-netfilter-all/`` 
#. Выполним генерацию конфигурации ``/home/svyatoslav/system/jetbrains/ja-netfilter-all/scripts/install.sh``
#. Выйдем из системы и снова залогинимся
#. Скопируем обновленную необходимую конфигурацию (в нашем случае для **Intellij Idea Ultimate**) из каталога
   ``/home/svyatoslav/system/jetbrains/ja-netfilter-all/vmoptions/idea.vmoptions``  в каталог ``/home/svyatoslav/.config/JetBrains/IntelliJIdea2021.3/idea.vmoptions``
#. Открываем файл ``/home/svyatoslav/system/jetbrains/ja-netfilter-all/config-jetbrains/mymap.conf`` устанавливаем любой срок действия лицензии в поле ``EQUAL,paidUpTo->``
#. Снова открываем `страницу <https://jetbra.in/s>`_ и копируем ключь продукта (кликнуть на нужной карточке)
#. Открываем **Intellij Idea Ultimate** и вставляем ключ в поле **Active Code**