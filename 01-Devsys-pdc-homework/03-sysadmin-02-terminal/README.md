1. cd - Это встроенная команда в shell. Внешняя команда cd все-таки существует и может быть реализована как сценарий shell, который в основном бесполезен , потому что как внешняя утилита не может повлиять на состояние shell, поэтому его попытки изменить каталог просто игнорируются.
2. Альтернатива команде `grep <some_string> <some_file> | wc -l`, команда `grep <some_string> <some_file> -c`.
3. Родителем для всех процессов в моей виртуальной машине с PID 1 является `systemd(1)`
4. Команда для перенаправления вывода stderr `ls -l \root 2>/dev/pts/1`. \
Вывод в другой сессии терминала `ls: cannot open directory 'root': Permission denied`
5. Считываем наш файл командой `cat test`:
```
Hello world
2021
2020
End
```
Дальше используем команду `cat <test >test_out` и считываем новый файл **test_out**:
```
Hello world
2021
2020
End
```
6. Используя команду `echo Hello world from pts0 to tty3 >/dev/tty3`\
Да сможем, но только надо переключится в tty (Ctrl+Alt+F3)
7. Команда `bash 5>&1` - Создаст дескриптор 5 и перенаправит его в stdout.\
Команда `echo netology > /proc/$$/fd/5` - выведет в дескриптор "5", который был перенаправлен в stdout.
8. Команда `ls -l /root 7>&2 2>&1 1>&7 | grep 'Permission denied' -c`\
7>&2 - Создали дескриптор 7 и перенаправили его в stderr\
2>&1 - stderr перенаправили в stdout\
1>&7 - stdout перенаправили в дескриптор 7.
9. Командой `cat /proc/$$/environ` будет выведено переменные окружения.
Аналогично можно использовать команду `env`
10. `/proc/<PID>/cmdline` - полный путь до исполняемого файла процесса [PID].  **(строка 172)**\
`/proc/<PID>/exe` - является символьной ссылкой, содержащей фактическое полное имя выполняемого файла. Если запустить exe будет открыт исполняемый файл.
Если ввести команду `/proc/<PID>/exe` чтобы запустить другую копию процесса такого же как и процесс с номером [PID]. **(Строка 212)**
11. Командой `cat /proc/cpuinfo` выведем информацию о процессоре.
Старшая версия набора инструкции SSE 4.2, а так же имеется набор инструкция SSE 4.1
12. ssh не имитирует TTY, если только ему не будет принудительно сказано сделать это с флагом `-t`. Тогда после исполнения команды ответ будет: 
```
/dev/pts/2
Connection to localhost closed.
```
13. При выполнении команды repty, были ошибки с доступом/правами в файле `10-patrace.conf`.
После изменения значения в строчки с `kernel.yama.ptrace_scope = 1` на `kernel.yama.ptrace_scope = 0`, получилось перевести сессию в screen.
14. Команда `tee` делает вывод одновременно и в файл, указанный в качестве параметра.
В данном примере команда читает из stdin и перенаправляет через pipe в stdout, а так как команда запущена под sudo , соответственно имеет права на запись в файл.
