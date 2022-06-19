1. Попробуйте запустить playbook на окружении из `test.yml`, зафиксируйте какое значение имеет факт `some_fact` для указанного хоста при выполнении playbook'a.
```bash
[root@dz 1]# ansible-playbook site.yml -i inventory/test.yml

PLAY [Print os facts] **************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************
ok: [localhost]

TASK [Print OS] ********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "CentOS"
}

TASK [Print fact] ******************************************************************************************************************************************
ok: [localhost] => {
    "msg": 12
}

PLAY RECAP *************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
`some_fact`=12

2. Найдите файл с переменными (group_vars) в котором задаётся найденное в первом пункте значение и поменяйте его на 'all default fact'.
Файл находился в `group_vars/all/examp.yml`, изменения внесли:
```bash
[root@dz 1]# ansible-playbook site.yml -i inventory/test.yml

PLAY [Print os facts] **************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************
ok: [localhost]

TASK [Print OS] ********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "CentOS"
}

TASK [Print fact] ******************************************************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}

PLAY RECAP *************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
3. Воспользуйтесь подготовленным (используется `docker`) или создайте собственное окружение для проведения дальнейших испытаний.
Скачиваем образы с поддержкой python:
```bash
[root@dz 1]# docker images | grep pycontribs
pycontribs/centos                                        7               bafa54e44377   14 months ago   488MB
pycontribs/ubuntu                                        latest          42a4e3b21923   2 years ago     664MB
```
```bash
[root@dz ~]# docker ps
CONTAINER ID   IMAGE                      COMMAND                  CREATED         STATUS         PORTS                                                                                      NAMES
cf2bd6ee5d91   pycontribs/ubuntu:latest   "/bin/sleep infinity"    5 seconds ago   Up 4 seconds                                                                                              ubuntu
07bd11ab3990   pycontribs/centos:7        "/bin/sleep infinity"    5 seconds ago   Up 4 seconds                                                                                              centos7
```
4. Проведите запуск playbook на окружении из `prod.yml`. Зафиксируйте полученные значения `some_fact` для каждого из `managed host`.
```bash
[root@dz 1]# ansible-playbook site.yml -i inventory/prod.yml

PLAY [Print os facts] ***************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *********************************************************************************************************************************************************************************************************************
ok: [ubuntu] => {
    "msg": "Ubuntu"
}
ok: [centos7] => {
    "msg": "CentOS"
}

TASK [Print fact] *******************************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el"
}
ok: [ubuntu] => {
    "msg": "deb"
}

PLAY RECAP **************************************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
Значения `some_fact`:
```
Для Centos - el
Для Ubuntu - deb
```
5. Добавьте факты в `group_vars` каждой из групп хостов так, чтобы для `some_fact` получились следующие значения: для `deb` - 'deb default fact', для `el` - 'el default fact'.
```bash
[root@dz 1]# cat group_vars/deb/examp.yml && cat group_vars/el/examp.yml
---
  some_fact: "deb default fact"---
  some_fact: "el default fact"
```
6. Повторите запуск playbook на окружении `prod.yml`. Убедитесь, что выдаются корректные значения для всех хостов.
```bash
[root@dz 1]# ansible-playbook site.yml -i inventory/prod.yml

PLAY [Print os facts] ***************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *********************************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] *******************************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP **************************************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
7. При помощи `ansible-vault` зашифруйте факты в `group_vars/deb` и `group_vars/el` с паролем `netology`.
```bash
[root@dz 1]# ansible-vault encrypt group_vars/deb/examp.yml group_vars/el/examp.yml
New Vault password:
Confirm New Vault password:
Encryption successful
[root@dz 1]# cat group_vars/deb/examp.yml && cat group_vars/el/examp.yml
$ANSIBLE_VAULT;1.1;AES256
32373736323735303266313430623739303336386661333933363736353835306331373638346339
3231383061326330623036383966306466393435396231370a343134626265373830383933353165
63353235353135363634636538636365383165323461653764356234343037366562636265396136
3465363531633164370a343665356164373262313631653530643438393130306631663238343561
37663436613565623231343834613562666539386633636533363661366331643461323931323762
3439363965333663333336333264383864323264336561326338
$ANSIBLE_VAULT;1.1;AES256
38613030313139333966363662626663326264363063663437346539303234623934313538323431
3336666563316539363034316566313336393139366365380a653939346237663865373433373166
63323366353861363035373131336133636337323763353566656161323661396331356564343366
3564383065363932320a313364356333346263316536386163303661386665323335663830323135
62326463663638626263323331633836643939363235366364396335643434663764356137386231
3666663235313636383465633430613533366636613861613464
```
8. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь в работоспособности.
```bash
[root@dz 1]# ansible-playbook site.yml -i inventory/prod.yml --ask-vault-pass
Vault password:

PLAY [Print os facts] ***************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *********************************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] *******************************************************************************************************************************************************************************************************************
ok: [ubuntu] => {
    "msg": "deb default fact"
}
ok: [centos7] => {
    "msg": "el default fact"
}

PLAY RECAP **************************************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
9. Посмотрите при помощи `ansible-doc` список плагинов для подключения. Выберите подходящий для работы на `control node`.
```bash
[root@dz 1]# ansible-doc --type=connection -l
kubectl      Execute tasks in pods running on Kubernetes
napalm       Provides persistent connection using NAPALM
qubes        Interact with an existing QubesOS AppVM
libvirt_lxc  Run tasks in lxc containers via libvirt
funcd        Use funcd to connect to target
chroot       Interact with local chroot
psrp         Run tasks over Microsoft PowerShell Remoting Protocol
zone         Run tasks in a zone instance
winrm        Run tasks over Microsoft's WinRM
paramiko_ssh Run tasks via python ssh (paramiko)
local        execute on controller
network_cli  Use network_cli to run command on network appliances
netconf      Provides a persistent connection using the netconf protocol
vmware_tools Execute tasks inside a VM via VMware Tools
podman       Interact with an existing podman container
lxd          Run tasks in lxc containers via lxc CLI
lxc          Run tasks in lxc containers via lxc python library
ssh          connect via ssh client binary
httpapi      Use httpapi to run command on network appliances
iocage       Run tasks in iocage jails
oc           Execute tasks in pods running on OpenShift
persistent   Use a persistent unix socket for connection
jail         Run tasks in jails
saltstack    Allow ansible to piggyback on salt minions
docker       Run tasks in docker containers
buildah      Interact with an existing buildah container
```
Нас интересует `local - execute on controller`

10. В `prod.yml` добавьте новую группу хостов с именем `local`, в ней разместите localhost с необходимым типом подключения.
```yaml
[root@dz 1]# cat inventory/prod.yml
---
  el:
    hosts:
      centos7:
        ansible_connection: docker
  deb:
    hosts:
      ubuntu:
        ansible_connection: docker

  local:
    hosts:
      localhost:
        ansible_connection: local
```
11. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь, что факты `some_fact` для каждого из хостов определены из верных `group_vars`.
```bash
[root@dz 1]# ansible-playbook site.yml -i inventory/prod.yml --ask-vault-pass
Vault password:

PLAY [Print os facts] ***************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************************************************
ok: [localhost]
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *********************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "CentOS"
}
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] *******************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP **************************************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```