1. Подбираем конфигурацию для проекта:
* Тип инфраструктуры будем использовать изменяемы, так как было заявлено "техническое задание еще не четкое, что приведет к большому количеству небольших релизов, тестирований интеграций, откатов, доработок".
* Будем использовать центральный сервер для управления инфраструктуры. У нас имеется достаточно много инструментов, которыми удобно будет управлять из одного места и конечно же логировать действия(Так как возможно мы будем не одни, кто будет вносить изменения в инфраструктуру), плюс к этому централизованное управление инфраструктурой - это правильный подход.
* Агенты на серверах не нужны, так как мы пользуемся `Ansible`, а в нем агенты не нужны.
* Тут точного ТЗ у нас не имеется, так что можно предположить, что в начале нашего пути мы будем использовать гибридный режим, а уже в последствии перейдем к средствам для управления конфигурации.
	
Для нового проекта мы будем использовать:
* `Packer` - для сбора образов
* `Terraform` - для создания нашей инфраструктуры
* `Docker` - для создания контейнеризированных приложений
* `Kebernetes` - для автоматизации развёртывания, масштабирования и управления контейнеризированными приложениями.
* `Ansible` - для управления конфигурациями
* `Teamcity` - автоматизация процессов CI/CD

Внедрение новых инструментов для проекта:
* Все `bash скрипты` которые у нас имеются, постараться перевести под управление `Ansible`
* Заменить `Teamcity` на `Gitlab CI`
* Конечно добавить мониторинг(Спасибо прошлым заданиям), а именно стек из `Prometheus/Grafana/Node Exporter`. Вариация данных инструментов возможна разная, зависит от ТЗ.
* И главное, ничего не было сказано, мы остаёмся в облаках или создаем свою инфраструктуру. Если свою, то тут предложил бы внедрить `VMware`, а именно `VMware Tanzu`.
2. Установили `terraform`:
```
[root@dz ~]# yum install terraform
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.corbina.net
 * epel: mirror.yandex.ru
 * extras: mirror.corbina.net
 * updates: mirror.corbina.net
Resolving Dependencies
--> Running transaction check
---> Package terraform.x86_64 0:1.1.7-1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==================================================================================================================================================
 Package                             Arch                             Version                           Repository                           Size
==================================================================================================================================================
Installing:
 terraform                           x86_64                           1.1.7-1                           hashicorp                            12 M

Transaction Summary
==================================================================================================================================================
Install  1 Package

Total download size: 12 M
Installed size: 60 M
Is this ok [y/d/N]: y
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
terraform-1.1.7-1.x86_64.rpm                                                                                               |  12 MB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : terraform-1.1.7-1.x86_64                                                                                                       1/1
  Verifying  : terraform-1.1.7-1.x86_64                                                                                                       1/1

Installed:
  terraform.x86_64 0:1.1.7-1

Complete!
[root@dz ~]# terraform --version
Terraform v1.1.7
on linux_amd64
```
3. Поддержали легаси код:
```
[root@dz /]# terraform --version
Terraform v1.1.7
on linux_amd64
[root@dz /]# terraform12 --version
Terraform v0.12.31

Your version of Terraform is out of date! The latest version
is 1.1.7. You can update by downloading from https://www.terraform.io/downloads.html
```
