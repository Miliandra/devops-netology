### 1. Опишите своими словами основные преимущества применения на практике IaaC паттернов.
* Основное преимущество применения IaaC паттернов, по-моему, мнению - это в том или ином виде меньшее использование человеческих ресурсов на однотипные/повторяющиеся задачи. А из этого исходит, что мы можем реально перераспределить ресурсы на важные задачи для нашего проекта/бизнеса.
* Еще одно преимущество, вытекающее из первого - это понятная форма/структура/схема/вид нашей инфраструктуры.
* Так же сюда для более развернутого ответа из двух вышеперечисленных вариантов можно включить:
  * легкое масштабирование инфраструктуры
  * Отслеживание изменений и откат к предыдущим версиям
  * Удобный формат разработки приложений и отправка их в репозиторий/сервер(CI/CD)
  * Создание однотипной среды, как для прода, так и для среды разработки и тестирования.
### Какой из принципов IaaC является основополагающим?
* Основополагающим IaaC принципом является получить понятный и предсказуемый, и легко повторяемый результат = Идемпотентность!
### 2. Чем Ansible выгодно отличается от других систем управление конфигурациями?
* Не нужно ставить клиента(Агента) на хосты/сервера, плюс к этому написан на Python и YAML плейбуки, которые просты и читаемы.
### Какой, на ваш взгляд, метод работы систем конфигурации более надёжный push или pull?
* Для определенной задачи/ситуации, метод работы подходит свой. Если у нас, например, небольшая инфраструктура, то целесообразно использовать метод `Push` - мы сами инициируем отправку конфигурации.\
А вот если у нас инфраструктура на несколько тысяч серверов, инициировать отправку с управляющего сервера такая себе задача, лучше использовать метод `Pull`. Но тут опять же встает вопрос надежности. И в данном случае, я бы разделил два метода по критичности для инфраструктуры. То есть, для более важной части инфраструктуры в плане отказа и выхода из строя(А значит простоя бизнес-процессов) использовать метод `Push`, ну а для менее важной части использовать метод `Pull`.\
Я думаю, все-таки, метод `Push` надежнее метода `Pull`(Если не так, то прошу дать разъяснение)
### 3. Vagrant
```
[root@localhost tmp]# vagrant --version
Vagrant 2.2.19
```
VirtualBox
```
[root@localhost ~]# virtualbox --help
Oracle VM VirtualBox VM Selector v6.1.32
(C) 2005-2022 Oracle Corporation
All rights reserved.

No special options.

If you are looking for --startvm and related options, you need to use VirtualBoxVM.
[root@localhost ~]# vboxmanage --version
6.1.32r149290
```
Ansible
```
[root@localhost tmp]# ansible --version
ansible 2.9.25
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Oct 14 2020, 14:45:30) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]
```
### 4. Создали ВМ с помощью Vagrant:
```
[root@dz vagrant]# vagrant up
Bringing machine 'server1.netology' up with 'virtualbox' provider...
==> server1.netology: Importing base box 'bento/ubuntu-20.04'...
==> server1.netology: Matching MAC address for NAT networking...
==> server1.netology: Checking if box 'bento/ubuntu-20.04' version '202112.19.0' is up to date...
==> server1.netology: Setting the name of the VM: server1.netology
==> server1.netology: Clearing any previously set network interfaces...
==> server1.netology: Preparing network interfaces based on configuration...
    server1.netology: Adapter 1: nat
    server1.netology: Adapter 2: hostonly
==> server1.netology: Forwarding ports...
    server1.netology: 22 (guest) => 20011 (host) (adapter 1)
    server1.netology: 22 (guest) => 2222 (host) (adapter 1)
==> server1.netology: Running 'pre-boot' VM customizations...
==> server1.netology: Booting VM...
==> server1.netology: Waiting for machine to boot. This may take a few minutes...
    server1.netology: SSH address: 127.0.0.1:2222
    server1.netology: SSH username: vagrant
    server1.netology: SSH auth method: private key
    server1.netology:
    server1.netology: Vagrant insecure key detected. Vagrant will automatically replace
    server1.netology: this with a newly generated keypair for better security.
    server1.netology:
    server1.netology: Inserting generated public key within guest...
    server1.netology: Removing insecure key from the guest if it's present...
    server1.netology: Key inserted! Disconnecting and reconnecting using new SSH key...
==> server1.netology: Machine booted and ready!
==> server1.netology: Checking for guest additions in VM...
==> server1.netology: Setting hostname...
==> server1.netology: Configuring and enabling network interfaces...
==> server1.netology: Mounting shared folders...
    server1.netology: /vagrant => /opt/vagrant
==> server1.netology: Running provisioner: ansible...
    server1.netology: Running ansible-playbook...

PLAY [nodes] *******************************************************************

TASK [Gathering Facts] *********************************************************
ok: [server1.netology]

TASK [Create directory for ssh-keys] *******************************************
ok: [server1.netology]

TASK [Adding rsa-key in /root/.ssh/authorized_keys] ****************************
changed: [server1.netology]

TASK [Checking DNS] ************************************************************
changed: [server1.netology]

TASK [Installing tools] ********************************************************
[DEPRECATION WARNING]: Invoking "apt" only once while using a loop via
squash_actions is deprecated. Instead of using a loop to supply multiple items
and specifying `package: "{{ item }}"`, please use `package: ['git', 'curl']`
and remove the loop. This feature will be removed in version 2.11. Deprecation
warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [server1.netology] => (item=[u'git', u'curl'])

TASK [Installing docker] *******************************************************
changed: [server1.netology]
[WARNING]: Consider using the get_url or uri module rather than running 'curl'.
If you need to use command because get_url or uri is insufficient you can add
'warn: false' to this command task or set 'command_warnings=False' in
ansible.cfg to get rid of this message.

TASK [Add the current user to docker group] ************************************
changed: [server1.netology]

PLAY RECAP *********************************************************************
server1.netology           : ok=7    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
Зашли в нее и удостоверились, что Docker установлен:
```
vagrant@server1:~$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
