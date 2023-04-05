.. |br| raw:: html

   <br />

CI/CD
#####

iOS
***

Сборка проекта проводится на  `виртуальной машине MacOS <https://github.com/kholia/OSX-KVM>`_ с помощью:

#. Fastlane
#. Gitlab Runner (развернут на хосте виртуалки MacOS)

Установка/настройка Fastlane
============================

Необходимо обновить ruby до последней версии. Для этого есть несколько методов. Выбран `RVM <https://github.com/rvm/rvm>`_ 

#. Выполнить ``curl -sSL https://raw.githubusercontent.com/rvm/rvm/master/binscripts/rvm-installer | bash -s stable``
#. Установим последную версию ``ruby rvm install ruby@latest`` либо ``rvm install ruby-X.X.X`` (первая команда выдавала ошибку поэтому лучше использовать вторую)
#. Установить версию по умолчанию ``rvm use ruby-X.X.X --default``
#. Проверить текущую версию ``ruby -v``

Установим **Fastlane**. Есть `несколько методов <https://docs.fastlane.tools/getting-started/ios/setup/>`_  - используем brew как самый простой: 

#. Установим Fastlane ``brew install fastlane``