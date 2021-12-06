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
