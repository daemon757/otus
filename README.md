**Название:** Vagrant-стенд для обновления ядра и создания образа системы

**Текст задания:**
* Цель: Научиться обновлять ядро в ОС Linux. Получение навыков работы с Vagrant, Packer и публикацией готовых образов в Vagrant Cloud.
* Описание домашнего задания
  1. Обновить ядро ОС из репозитория ELRepo
  2. Создать Vagrant box c помощью Packer
  3. Загрузить Vagrant box в Vagrant Cloud

**Описание команд:**
* Подготовлен Vagrantfile для загрузки vagrantbox
* Запускаем ВМ
```
kdp@S510p:~/otus$ vagrant up
Bringing machine 'kernel-update' up with 'virtualbox' provider...
==> kernel-update: Checking if box 'generic/centos8s' version '4.3.4' is up to date...
==> kernel-update: Clearing any previously set forwarded ports...
==> kernel-update: Clearing any previously set network interfaces...
==> kernel-update: Preparing network interfaces based on configuration...
    kernel-update: Adapter 1: nat
==> kernel-update: Forwarding ports...
    kernel-update: 22 (guest) => 2222 (host) (adapter 1)
==> kernel-update: Running 'pre-boot' VM customizations...
==> kernel-update: Booting VM...
==> kernel-update: Waiting for machine to boot. This may take a few minutes...
    kernel-update: SSH address: 127.0.0.1:2222
    kernel-update: SSH username: vagrant
    kernel-update: SSH auth method: private key
==> kernel-update: Machine booted and ready!
==> kernel-update: Checking for guest additions in VM...
    kernel-update: The guest additions on this VM do not match the installed version of
    kernel-update: VirtualBox! In most cases this is fine, but in rare cases it can
    kernel-update: prevent things such as shared folders from working properly. If you see
    kernel-update: shared folder errors, please make sure the guest additions within the
    kernel-update: virtual machine match the version of VirtualBox you have installed on
    kernel-update: your host and reload your VM.
    kernel-update:
    kernel-update: Guest Additions Version: 6.1.30
    kernel-update: VirtualBox Version: 7.0
==> kernel-update: Setting hostname...
==> kernel-update: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> kernel-update: flag to force provisioning. Provisioners marked to run always will still run.
```
* открываем сессию
```
kdp@S510p:~/otus$ vagrant ssh
Last login: Sun Nov 12 20:39:20 2023 from 10.0.2.2
[vagrant@kernel-update ~]$ uname -r
6.5.10-1.el8.elrepo.x86_64
```

* Проверяем исходную версию ядра
```
[vagrant@kernel-update ~]$ uname -r
4.18.0-277.el8.x86_64
```
* Подключаем репозиторий
```
sudo yum install -y https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
```
* Обновляем ядро
```
sudo yum --enablerepo elrepo-kernel install kernel-ml -y
```
* Правим конфиг grub
```
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
sudo grub2-set-default 0
```
* Перезагружаем ВМ
sudo reboot
* Проверяем версию ядра после обновления
```
[vagrant@kernel-update ~]$ uname -r
6.5.10-1.el8.elrepo.x86_64
```
* Готовим packer.json и ks.cfg для автоматического развертывания Linux
* Создаем образ системы
```
kdp@S510p:~/otus/packer$ packer build centos.json

Build 'virtualbox-iso.centos-8stream' finished after 37 minutes 41 seconds.

==> Wait completed after 37 minutes 41 seconds

==> Builds finished. The artifacts of successful builds are:
--> virtualbox-iso.centos-8stream: 'virtualbox' provider box: centos-8-kernel-6-x86_64-Minimal.box
* kdp@S510p:~/otus/packer$ vagrant box add --name centos8-kernel6 centos-8-kernel-6-x86_64-Minimal.box
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'centos8-kernel6' (v0) for provider:
    box: Unpacking necessary files from: file:///home/kdp/otus/packer/centos-8-kernel-6-x86_64-Minimal.box
==> box: Successfully added box 'centos8-kernel6' (v0) for ''!
```
* проверяем полученый образ
```
kdp@S510p:~/otus/packer$ vagrant box list
bento/ubuntu-16.04 (virtualbox, 202212.11.0)
centos-8-kernel6   (virtualbox, 0)
centos8-kernel6    (virtualbox, 0)
generic/centos8s   (virtualbox, 4.3.4, (amd64))

kdp@S510p:~/otus/packer$ vagrant init centos8-kernel6
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.

kdp@S510p:~/otus/packer$ vagrant up
Bringing machine 'packer-kernel6' up with 'virtualbox' provider...
==> packer-kernel6: Importing base box 'centos8-kernel6'...
==> packer-kernel6: Matching MAC address for NAT networking...
==> packer-kernel6: Setting the name of the VM: packer_packer-kernel6_1699886887125_63548
==> packer-kernel6: Clearing any previously set network interfaces...
==> packer-kernel6: Preparing network interfaces based on configuration...
    packer-kernel6: Adapter 1: nat
==> packer-kernel6: Forwarding ports...
    packer-kernel6: 22 (guest) => 2222 (host) (adapter 1)
==> packer-kernel6: Running 'pre-boot' VM customizations...
==> packer-kernel6: Booting VM...
==> packer-kernel6: Waiting for machine to boot. This may take a few minutes...
    packer-kernel6: SSH address: 127.0.0.1:2222
    packer-kernel6: SSH username: vagrant
    packer-kernel6: SSH auth method: private key
    packer-kernel6:
    packer-kernel6: Vagrant insecure key detected. Vagrant will automatically replace
    packer-kernel6: this with a newly generated keypair for better security.
    packer-kernel6:
    packer-kernel6: Inserting generated public key within guest...
    packer-kernel6: Removing insecure key from the guest if it's present...
    packer-kernel6: Key inserted! Disconnecting and reconnecting using new SSH key...
==> packer-kernel6: Machine booted and ready!
==> packer-kernel6: Checking for guest additions in VM...
    packer-kernel6: No guest additions were detected on the base box for this VM! Guest
    packer-kernel6: additions are required for forwarded ports, shared folders, host only
    packer-kernel6: networking, and more. If SSH fails on this machine, please install
    packer-kernel6: the guest additions and repackage the box to continue.
    packer-kernel6:
    packer-kernel6: This is not an error message; everything may continue to work properly,
    packer-kernel6: in which case you may ignore this message.
==> packer-kernel6: Mounting shared folders...
    packer-kernel6: /vagrant => /home/kdp/otus/packer

kdp@S510p:~/otus/packer$ vagrant ssh
Last login: Sun Nov 12 18:06:07 2023 from 10.0.2.2
[vagrant@otus-c8 ~]$
```

* Подключаемся к Vagrand Cloud
```
kdp@S510p:~/otus/packer$ vagrant cloud auth login
In a moment we will ask for your username and password to HashiCorp's
Vagrant Cloud. After authenticating, we will store an access token locally on
disk. Your login details will be transmitted over a secure connection, and
are never stored on disk locally.

If you do not have an Vagrant Cloud account, sign up at
https://www.vagrantcloud.com

Vagrant Cloud username or email: dem757@gmail.com
Password (will be hidden):
Token description (Defaults to "Vagrant login from S510p"): wlF7nmSim57Apw.atlasv1.22Nm3lPRu8NDz5YqKHDVEzjLMljhyToHXwNvReqCuop5gzMRyvCN12qAlRxfnVNzLvg
You are now logged in.
```
* Загружаем box в Vagrand Cloud
```
kdp@S510p:~/otus/packer$ vagrant cloud publish --release dem757/centos8-kernel6 1.0 virtualbox centos-8-kernel-6-x86_64-Minimal.box
You are about to publish a box on Vagrant Cloud with the following options:
dem757/centos8-kernel6:   (v1.0) for provider 'virtualbox'
Automatic Release:     true
Box Architecture:      amd64
Do you wish to continue? [y/N]y
Saving box information...
Uploading provider with file /home/kdp/otus/packer/centos-8-kernel-6-x86_64-Minimal.box
```

https://app.vagrantup.com/dem757/boxes/centos8-kernel6
