1. **cd** использует системный вызов `chdir("/tmp")`
2. Файл базы типов - `/usr/share/misc/magic.mgc`
В выполненной команде это выглядит так:\
`openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3`
3. Получаем файловый дескриптор для pid где происходит запись в файл, а потом в дескриптор перенаправим пустую строку:
```
ps aux | grep vi
-vagrant    10889  0.0  0.9  21928  9988 pts/0    S+   15:57   0:00 vi test_log
lsof -p 10889
-vi      10889 vagrant    3u   REG  253,0    12288 1835025 /home/vagrant/.test_log.swp (deleted)
echo '' >/proc/10889/fd/3
```
4. Зомби процессы освобождают свои ресурсы, но не освобождают запись в таблице процессов. 
5. Установка утилиты bpfcc-tools - `sudo apt get bpfcc-tools`\
Файлы за первую секудну работы утилиты:
```
659    vmtoolsd            9   0 /proc/meminfo
659    vmtoolsd            9   0 /proc/vmstat
659    vmtoolsd            9   0 /proc/stat
659    vmtoolsd            9   0 /proc/zoneinfo
659    vmtoolsd            9   0 /proc/uptime
659    vmtoolsd            9   0 /proc/diskstats
422    systemd-udevd      14   0 /sys/fs/cgroup/unified/system.slice/systemd-udevd.service/cgroup.procs
422    systemd-udevd      14   0 /sys/fs/cgroup/unified/system.slice/systemd-udevd.service/cgroup.threads
600    multipathd          8   0 /sys/devices/pci0000:00/0000:00:10.0/host2/target2:0:0/2:0:0:0/state
600    multipathd          8   0 /sys/devices/pci0000:00/0000:00:10.0/host2/target2:0:0/2:0:0:0/block/sda/size
600    multipathd          8   0 /sys/devices/pci0000:00/0000:00:10.0/host2/target2:0:0/2:0:0:0/state
600    multipathd         -1   2 /sys/devices/pci0000:00/0000:00:10.0/host2/target2:0:0/2:0:0:0/vpd_pg80
600    multipathd         -1   2 /sys/devices/pci0000:00/0000:00:10.0/host2/target2:0:0/2:0:0:0/vpd_pg83
394    systemd-journal    35   0 /proc/600/status
394    systemd-journal    35   0 /proc/600/status
394    systemd-journal    35   0 /proc/600/comm
394    systemd-journal    35   0 /proc/600/cmdline
394    systemd-journal    35   0 /proc/600/status
394    systemd-journal    35   0 /proc/600/attr/current
394    systemd-journal    35   0 /proc/600/sessionid
394    systemd-journal    35   0 /proc/600/loginuid
394    systemd-journal    35   0 /proc/600/cgroup
394    systemd-journal    -1   2 /run/systemd/units/log-extra-fields:multipathd.service
394    systemd-journal    -1   2 /run/log/journal/84ccdb80214c4dfab8b76df8e4b5593e/system.journal
394    systemd-journal    -1   2 /run/log/journal/84ccdb80214c4dfab8b76df8e4b5593e/system.journal
394    systemd-journal    -1   2 /run/log/journal/84ccdb80214c4dfab8b76df8e4b5593e/system.journal
394    systemd-journal    -1   2 /run/log/journal/84ccdb80214c4dfab8b76df8e4b5593e/system.journal
```
6. Для этого установим manpages-dev `sudo apt install manpages-dev`\
И ответ будет:\
`Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.`
7. При `;` команды выполняются последовательно, независимо от результата работы предыдущей. Если не поставить этот разделитель команд, то последующая команда может быть воспринята как аргумент предыдущей.\
При `&&` следующая команда выполнится только, если предидущая завершится успешно (То есть равен 0), альтернатива данному оператору `||` - это значит следующая команда выполнится только, если предыдущая завершится статусом отличным от нуля .\
shell при `set -e` не завершается в том случае, если выражение, вернуло ненулевой статус. Нет смысла использовать `&&` после вызова `set -e`, так как при ошибке , выполнение команд прекратиться.
8. `man bash | grep pipefail`\
`-e` - предписывает bash немедленно выйти, если какая-либо команда имеет ненулевой статус выхода.\
`-u` - При установке ссылка на любую переменную, которую вы ранее не определяли - за исключением $* и $@ - является ошибкой и приводит к немедленному завершению программы.\
`-x` - Включает режим командной оболочки, в котором все выполняемые команды выводятся на терминал. Выводит каждую команду по мере ее выполнения.\
`-o` - Этот параметр предотвращает подмену ошибок в сценарии. Если какая-либо команда в сценарии завершится неудачно, этот код возврата будет использоваться в качестве кода возврата всего сценария.\
Опцию pipefail хорошо использовать в сценариях, из-за дополнительного логирования, а также завершения сценария при наличии ошибок.
9. Наиболее часто встречающейся статус у процессов в системе - это S (И его разновидности S+,Ss,Ss+).\
Дополнительные буквы статуса - это приоритет, многопоточность.

