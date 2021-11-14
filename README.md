# Course DevOps-Engineer from Netology
## Student Karev Ilya

.gitignore terraform:
1. `**/.terraform/*` - Исключение директории .terraform/* и все что имеется в ней, но игнорировать любые директории и фалы до нее
2. Исключение всех файлов с расширением `*.tfstate` и исключение всех файлов с маской `*.tfstate.*`
3. Исключение файла Crash.log
4. Исключение всех файлов с раширением `*.tfvars`
5. Исключение файла `override.tf` и `override.tf.json`, а также исключение любых файлов с маской `*_override.tf` и `*_override.tf.json`
6. `!example_override.tf` - Можно добавить опредленные фалы по данной маске, минуя исключение №5
7. Исключение файлов по маске `*tfplan*`
8. Исключение файла `.terraformrc` и `terraform.rc`

### ДЗ 2.4. Инструменты Git
1. Поиск хеша был выполнен командой `git log | grep aefea` (Так же можно использовать комманду  `git rev-parse aefea` или `git show aefea`), таким образом был найден полный хеш **aefead2207ef7e2aa5dc81a34aedf0cad4c32545**.\
Комментарий к коммиту был выполнен с помощью команды `git show aefea --name-status` : **Update CHANGELOG.md**
2. Коммит 85024d3 соответсвует тегу tag: v0.12.23. Результат был получен с помощью команды `git show 85024d3`.
3. Командой `git show b8d720` был выполнен поиск родителей данного коммита.\
Был найден родитель с коммитом **9ea88f22f** и **56cd7859e**. Проверкой так же послужила команда `git log b8d720f83 --graph --oneline -10`, где мы наглядно увидели родителей данного коммита.
4. Командой `git log v0.12.23..v0.12.24 --oneline` мы вывели все коммиты и комментарии между тегами v0.12.23 и v0.12.24:
* 33ff1c03b (tag: v0.12.24) v0.12.24
* b14b74c49 [Website] vmc provider links
* 3f235065b Update CHANGELOG.md
* 6ae64e247 registry: Fix panic when server is unreachable
* 5c619ca1b website: Remove links to the getting started guide's old location
* 06275647e Update CHANGELOG.md
* d5f9411f5 command: Fix bug when using terraform login on Windows
* 4b6d06cc5 Update CHANGELOG.md
* dd01a3507 Update CHANGELOG.md
* 225466bc3 Cleanup after v0.12.23 release
5. Для поиска коммита в котором была создана функция `func providerSource`, была использована команда  `git log -S 'func providerSource' --oneline`\
Было найдено 2 коммита:
* 5af1e6234 main: Honor explicit provider_installation CLI config when present\
* 8c928e835 main: Consult local directories as potential mirrors of providers
6. Функция `globalPluginDirs` была изменена в следующих коммитах:
* 78b122055
* 52dbf9483
* 41ab0aef7
* 66ebff90c
* 8364383c3\
Комманда для поиска: `git log -L :'func globalPluginDirs':plugins.go --oneline`
7. Автор функции `synchronizedWriters` - Martin Atkins\
Команда для поиска: `git log -S'func synchronizedWriters' --pretty=format:'%h - %aN - %as - %ar'`
