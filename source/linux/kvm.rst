.. |br| raw:: html

   <br />

KVM
###

Увеличить место на диске
************************

#. Отключаем виртуальную машину ``sudo virsh shutdown NAME`` (желательно если есть Docker
   внутри машины остановить контейнеры до выключения).

#. Найдем расположение диска виртуалки ``sudo virsh domblklist NAME`` и можем посмотреть информацию о нем ``sudo qemu-img info /path/disk.qcow2``
#. Добавим место на диск ``sudo qemu-img resize /path/disk.qcow2 +10G``
#. Запустим виртуальную машину ``sudo virsh start NAME`` и дальнейшие действия выполняем уже на виртуалке. 

Далее рассмотрим процесс увеличения простарнсва диска для **зашифрованного LVM** диска.

#. Посмотрим информацию о текущих дисках

   .. code-block:: console

      [root@git2 ~]# lsblk
      NAME                                          MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
      sr0                                            11:0    1 1024M  0 rom
      vda                                           252:0    0  170G  0 disk
      ├─vda1                                        252:1    0    1G  0 part  /boot
      └─vda2                                        252:2    0  149G  0 part
        └─luks-c53ff21a-b967-4ce9-ac91-1a183997f769 253:0    0  149G  0 crypt
          ├─centos-root                             253:1    0  140G  0 lvm   /
          ├─centos-swap                             253:2    0    7G  0 lvm   [SWAP]
          └─centos-home                             253:3    0    2G  0 lvm   /home
      [root@git2 ~]# pvs
        PV                                                    VG     Fmt  Attr PSize   PFree
        /dev/mapper/luks-c53ff21a-b967-4ce9-ac91-1a183997f769 centos lvm2 a--  148.99g    0

#. Установим ``sudo yum -y install cloud-utils-growpart``
#. Увеличим раздел vda2 ``growpart /dev/vda 2`` посмотрим информацию

   .. code-block:: console

      [root@git2 ~]# lsblk
      NAME                                          MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
      sr0                                            11:0    1 1024M  0 rom
      vda                                           252:0    0  170G  0 disk
      ├─vda1                                        252:1    0    1G  0 part  /boot
      └─vda2                                        252:2    0  169G  0 part
        └─luks-c53ff21a-b967-4ce9-ac91-1a183997f769 253:0    0  149G  0 crypt
          ├─centos-root                             253:1    0  140G  0 lvm   /
          ├─centos-swap                             253:2    0    7G  0 lvm   [SWAP]
          └─centos-home                             253:3    0    2G  0 lvm   /home

   Видим раздел vda2 увеличился.

#. Если диск зашифрован, то необходимо выполнить ``cryptsetup resize luks-c53ff21a-b967-4ce9-ac91-1a183997f769``
   и убедиться, что зашифрованный раздел увеличен. Если диск не шифрован (отсутсвует строка ``luks-c53ff21a-b967-4ce9-ac91-1a183997f769``) 
   выполним ``lvextend -r -l +100%FREE /dev/centos/root`` (выбрав нужный раздел)

   .. code-block:: console

      [root@git2 ~]# lsblk
      NAME                                          MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
      sr0                                            11:0    1 1024M  0 rom
      vda                                           252:0    0  170G  0 disk
      ├─vda1                                        252:1    0    1G  0 part  /boot
      └─vda2                                        252:2    0  169G  0 part
        └─luks-c53ff21a-b967-4ce9-ac91-1a183997f769 253:0    0  169G  0 crypt
          ├─centos-root                             253:1    0  140G  0 lvm   /
          ├─centos-swap                             253:2    0    7G  0 lvm   [SWAP]
          └─centos-home                             253:3    0    2G  0 lvm   /home

#. Добавим место к необходимому разделу LVM ``lvresize -l+100%FREE /dev/centos/root``
   Теперь раздел ``centos-root`` должен быть увеличен

#. Для файловой системы **xfs** необходимо обновить таблицу ``xfs_growfs /dev/centos/root``. Для 
   других ФС команда будет иной 

#. Убедимся что все прошло успешно

   .. code-block:: console

      [root@git2 ~]# df -h
      Filesystem               Size  Used Avail Use% Mounted on
      devtmpfs                 3.8G     0  3.8G   0% /dev
      tmpfs                    3.8G     0  3.8G   0% /dev/shm
      tmpfs                    3.8G  8.8M  3.8G   1% /run
      tmpfs                    3.8G     0  3.8G   0% /sys/fs/cgroup
      /dev/mapper/centos-root  160G   60G  101G  37% /
      /dev/vda1               1014M  198M  817M  20% /boot
      /dev/mapper/centos-home  2.0G   33M  2.0G   2% /home
      none                     3.8G  4.0K  3.8G   1% /etc/resolv.conf
      overlay                  160G   60G  101G  37% /var/lib/docker/overlay2/e6f07e5ba823a6e7dab6ec02291e264eb63b8d1e3adccc9879960344a07cd457/merged
      overlay                  160G   60G  101G  37% /var/lib/docker/overlay2/e687735d29d5d47cc70442f796db2f847f9936034e700c6632e0da11893326b9/merged
      overlay                  160G   60G  101G  37% /var/lib/docker/overlay2/2d8c94edf346d9ea67842b3f5f26f0311a099c4fd6a5c0087ee9ca8ffa67cd20/merged
      overlay                  160G   60G  101G  37% /var/lib/docker/overlay2/f973a411c61e16be7aa8a15289f2e16de01d8cee05a62d57d27006baa59f1728/merged
      tmpfs                    764M     0  764M   0% /run/user/0

Проброс PCI видеокарты
**********************

Проще всего пробрасывать карты **ATI Radeon** на них меньше проблем. Инструкция построена на информации со страницы
`GPU passthrough mac kvm <https://github.com/kholia/OSX-KVM/blob/master/notes.md#gpu-passthrough-notes/>`_  Так же полезным может быть старница 
`Single GPU Passthrough <https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/home/>`_

Подготовка ОС
=============

#. Включить в биос поддержку виртуализации **Intel®Virtualization Technology** и поддержку **iommu VT-x**

#. Запрещаем загрузку драйвера модуля ядра для видеокарты (для Nvidia другой модуль)

   .. code-block:: console

      Добавить в файл /etc/modprobe.d/blacklist.conf строки

      blacklist amdgpu
      blacklist radeon

#. Найдем id видеокарты на шине pci 

   .. code-block:: console

      $ lspci -nnk | grep AMD
      01:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Ellesmere [Radeon RX 470/480/570/570X/580/580X/590] [1002:67df] (rev ef)
      01:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Ellesmere HDMI Audio [Radeon RX 470/480 / 570/580/590] [1002:aaf0]

   в примере выше это ``1002:67df`` и ``1002:aaf0``

#. Включаем поддержку IOMMU и конфигурируем VFIO

   Редактируем ``sudo nano /etc/default/grub`` добавив в строку ``GRUB_CMDLINE_LINUX_DEFAULT`` параметры 
   (для cpu intel) и заменив **vfio-pci.ids** на свои
   ``iommu=pt intel_iommu=on vfio-pci.ids=1002:67df,1002:aaf0 kvm.ignore_msrs=1 video=vesafb:off,efifb:off``

#. Настроим **vfio** ``sudo nano /etc/modprobe.d/vfio.conf``

   .. code-block:: console

      options vfio-pci ids=1002:67df,1002:aaf0 disable_vga=1
      softdep radeon pre: vfio-pci
      softdep amdgpu pre: vfio-pci
      softdep nouveau pre: vfio-pci
      softdep drm pre: vfio-pci

#. Обновим конфигурацию загрузчика и initramfs ``sudo update-grub2`` ``sudo update-initramfs -k all -u``

#. Перезагружаемся и проверяем

#. Проверка модуля **iommu**

   .. code-block:: console

      $ dmesg | grep -i iommu
      [    0.076879] DMAR: IOMMU enabled
      [    0.183732] DMAR-IR: IOAPIC id 2 under DRHD base  0xfed91000 IOMMU 1
      [    0.330654] iommu: Default domain type: Passthrough (set via kernel command line)
      [    0.489615] pci 0000:00:00.0: Adding to iommu group 0
      [    0.489627] pci 0000:00:01.0: Adding to iommu group 1
      [    0.489634] pci 0000:00:02.0: Adding to iommu group 2
      [    0.489643] pci 0000:00:14.0: Adding to iommu group 3

#. Проверка **vfio**

   .. code-block:: console

      $ dmesg | grep vfio
      [    0.526198] vfio-pci 0000:01:00.0: vgaarb: changed VGA decodes: olddecodes=io+mem,decodes=io+mem:owns=io+mem
      [    0.543768] vfio_pci: add [1002:67df[ffffffff:ffffffff]] class 0x000000/00000000
      [    0.563765] vfio_pci: add [1002:aaf0[ffffffff:ffffffff]] class 0x000000/00000000
      [    3.384597] vfio-pci 0000:01:00.0: vgaarb: changed VGA decodes: olddecodes=io+mem,decodes=io+mem:owns=io+mem

#. Проверка того что видеокарта использует драйвер **vfio-pci**

   .. code-block:: console

      $ lspci -nkk -d 1002:67df
      01:00.0 0300: 1002:67df (rev ef)
              Subsystem: 1da2:e366
              Kernel driver in use: vfio-pci
              Kernel modules: amdgpu

#. Исправим права для ``/dev/vfio/1`` :download:`Скачать vfio-kvm.rules <linux_files/kvm/vfio-kvm.rules>` 

   .. code-block:: console

      sudo cp vfio-kvm.rules /etc/udev/rules.d/vfio-kvm.rules

      sudo udevadm control --reload
      sudo udevadm trigger

#. Добавим в файл ``/etc/security/limits.conf`` 

   .. code-block:: console

      @kvm            soft    memlock         unlimited
      @kvm            hard    memlock         unlimited
      @libvirt        soft    memlock         unlimited
      @libvirt        hard    memlock         unlimited

Работа с BIOS видеокарты
========================

Карта может быть проброшена в виртуальную машину запущенную в режиме **BIOS** или в режиме **UEFI**
Для виртуальных машин **UEFI** (такие как Windows, MacOS) видеокарта обязательно должна поддерживать **UEFI GOP** иначе при старте виртуалки 
на мониторе не будет никакого изображения. Для проверки можно использовать утилиту **GPU-Z** где должен быть отмечен
чекбокс **UEFI**. Если поддержка отсутсвует можно попробовать пропатчить BIOS видеокарты

#. Сделать дамп текущего BIOS GPU (например так же через **GPU-Z**)
#. Пропатчить с помощью  `GOPUpd <https://www.win-raid.com/t892f16-AMD-and-Nvidia-GOP-update-No-requests-DIY-69.html/>`_ дамп BIOS GPU
#. Прошить новый BIOS через `ATI ATIFlash <https://www.techpowerup.com/download/ati-atiflash//>`_
#. Проверить появилась ли поддержка (например так же через **GPU-Z**)

Создание виртуальной машины
===========================

#. Создаем виртуальную машину как обычно (например через **Virsh Manager**). 
#. Удаляем все устройства **Video**, **Graphics**
#. Добавляем PCI устройство нашей видео карты (обычно 2 шт. **VGA ATI RADEON** и **AUDIO ATI RADEON**)
#. Подсовываем BIOS GPU нашей видеокарты (может запуститься и без этого шага)

   #. Копируем дамп BIOS GPU по пути ``/usr/share/vgabios/vgpu.rom``
   #. Прописываем в каждом выше добавленном PCI устройстве (для **VGA ATI RADEON** и **AUDIO ATI RADEON**) строку ``<rom file='/usr/share/vgabios/vgpu.rom'/>``

#. Если виртуальная машина запускается в режиме BIOS (не в UEFI), то так же необходимо указать параметры:

   .. code-block:: console

      <qemu:commandline>                                              
        <qemu:arg value='-set'/>                                          
        <qemu:arg value='device.hostdev0.x-vga=on'/>      
      </qemu:commandline>