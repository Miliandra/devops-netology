# Домашнее задание "8.4 Работа с Roles"

1. Репозиторий с ролью от Clickhouse [Ссылка на репозиторий](https://github.com/Miliandra/clickhouse-role)
2. Репозиторий с ролью от Lighthouse [Ссылка на репозиторий](https://github.com/Miliandra/lighthouse-role)
3. Репозиторий с ролью от Vector [Ссылка на репозиторий](https://github.com/Miliandra/vector-role)

Playbook состоит из запуска только ролей.\
Все переменные, которые можно изменить описаны непосредственно в ролях.\
Скачивание ролей с помощью `ansible-galaxy` проходит успешно.
```
root@ansible-cm:/opt/test_ansible/1# ansible-galaxy install -r requirements.yml -p roles
Starting galaxy role install process
- extracting clickhouse to /opt/test_ansible/1/roles/clickhouse
- clickhouse (main) was installed successfully
- extracting lighthouse to /opt/test_ansible/1/roles/lighthouse
- lighthouse (main) was installed successfully
- extracting vector to /opt/test_ansible/1/roles/vector
- vector (main) was installed successfully
```