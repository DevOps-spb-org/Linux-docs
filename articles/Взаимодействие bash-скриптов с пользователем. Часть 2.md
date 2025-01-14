# Взаимодействие bash-скриптов с пользователем. Часть 2

_Наша программа настолько сурова, что даже логин отображается звездочками (bash.org.ru)_

Вашему вниманию представляется новая подборка средств общения скриптов с пользователем. Надеюсь, интересно будет всем, кто не боится работать с консолью.  
Первую часть можно найти  [тут](http://habrahabr.ru/blogs/nix/126701/ "Взаимодействие bash-скриптов с пользователем") .  
  

#### Опции (ключи)

  
Этот способ был достоин первой части статьи, но не попал туда из-за забывчивости автора. Несомненно, этот способ знаком всем пользователям \*nix, хоть раз работавших с консолью. Простой наглядный пример:  
![](/images/1a7f2eed2de5b7e92fcda28d5422e4eb.png)   
Плюс в том, что ключи короткие и их можно комбинировать. Попытаемся сделать что-нибудь подобное, а за одно изучим еще несколько моментов.  

> ```
> #!/bin/bash
> set -e
> 
> ME=`basename $0`
> function print_help() {
>     echo "Работа с файлом test_file"
>     echo
>     echo "Использование: $ME options..."
>     echo "Параметры:"
>     echo "  -c          Создание файла test_file."
>     echo "  -w text     Запись в файл строки text."
>     echo "  -r          Удаление файла test_file."
>     echo "  -h          Справка."
>     echo
> }
> 
> function create_file() {
>     touch test_file
> }
> 
> function write_to_file {
>     echo "$TEXT" >> test_file
> }
> 
> function remove_file {
>     rm test_file
> }
> 
> # Если скрипт запущен без аргументов, открываем справку.
> if [ $# = 0 ]; then
>     print_help
> fi
> 
> while getopts ":cw:r" opt ;
> do
>     case $opt in
>         c) create_file;
>             ;;
>         w) TEXT=$OPTARG;
>             write_to_file
>             ;;
>         r) remove_file
>             ;;
>         *) echo "Неправильный параметр";
>             echo "Для вызова справки запустите $ME -h";
>             exit 1
>             ;;
>         esac
> done
> 
> ```

  
Итак, что мы имеем?  

*   Команда **set -e** остановит скрипт, если при его выполнении возникнет ошибка (подробнее о других опциях [здесь](http://ss64.com/bash/set.html "set man page") ).
*   Основные операции скрипта упакованы в функции. Конечно, глупо помещать в функции по одной команде, но это лишь для примера, в реальности их может быть ну очень много.
*   Функция **getopts** разбирает переданные аргументы. За ней перечисляются допустимые опции. Двоеточие после опции 'w' означает, что c данной опцией идет дополнительный аргумент, который помещается в переменную **$OPTARG** .
*   Опции можно комбинировать, но стоит учитывать то, что они выполняются по порядку. Это значит, что если мы выполним **script -rc** то сначала файл будет удален, а затем создан. При этом, если файла не существовало, то скрипт завершится с ошибкой, не дойдя до создания файла.
*   Также стоит учитывать то, что после ключа 'w' обязательно должен следовать аргумент. Если он будет отсутствовать, то скрипт выполнит опцию '\*' (по умолчанию). Интересно, что если запустить **script -wr Hallo** , то опция 'r' будет воспринята как дополнительный параметр к опции 'w', а 'Hallo' проигнорировано. Правильно будет **script -w Hallo -r** 

  
Подробнее о **getopts** можно узнать [здесь](http://www.opennet.ru/docs/RUS/bash_scripting_guide/c5358.html#GETOPTSX) .  
  

#### Выбор

  
В предыдущей статье я рассматривал выбор варианта выполнения с помощью **case** . А сейчас рассмотрим создание меню с помощью конструкции **select** , которая позволяет создать простые нумерованные меню.  

> ```
> #!/bin/bash
> 
> # Изменение строки приветствия
> PS3='Выберите операционную систему: '
> 
> select OS in "Linux" "Windows" "Mac OS" "BolgenOS"
> do
>   echo
>   echo "Вы выбрали $OS!"
>   echo
>   break
> done
> 
> ```

  
![](/images/f4308ea3d83c91b299607d23a3f1541d.png)   
Подробнее описано [здесь](http://www.opennet.ru/docs/RUS/bash_scripting_guide/x5210.html#select) .  
  

#### Логирование

  
Бывает удобно не выводить сообщения на экран, а записывать их в лог-файл. Особенно если скрипт запускается при старте системы.  
Для этого можно использовать обычную запись в файл.  

> ```
> #!/bin/bash
> 
> NAME=`basename $0`
> TIME=`date +%F\ %H:%M:%S`
> TYPE='<info>'
> 
> echo "$TIME $NAME: $TYPE Operation completed successfully" >> /tmp/log
> 
> ```

  
Но есть и специальный инструмент ведения логов — **logger** .  

> ```
> logger Operation completed successfully
> sudo tail /var/log/syslog
> 
> ```

  
Подробнее [тут](http://www.opennet.ru/man.shtml?topic=logger&category=1 "man logger") .  
  

#### Оповещения на рабочем столе

  
![](/images/47509e9db5adb43000e1b7ece9db81a4.png)   
Немножко развлечемся и поиграемся с нотиферами. Для начала поставим нужный пакет:  

> ```
> sudo apt-get install libnotify-bin
> 
> ```

  
Теперь выполним простейший пример прямо в терминале:  

> ```
> notify-send --expire-time=10000 "Привет" "Я слежу за тобой"
> 
> ```

  
Об этом уже писали на [Хабре](http://habrahabr.ru/blogs/linux/47892/) .  
  

#### Клавиатурные индикаторы

  
Хотите поморгать лампочками на клавиатуре? Да, пожалуйста!  

> ```
> #!/bin/bash
> 
> setleds -D +caps < /dev/tty7
> sleep 1
> setleds -D -caps < /dev/tty7
> 
> ```

  
Скрипт необходимо запускать с правами рута!  
  

#### Звуковые сигналы

  
Звуковые сигналы можно подавать несколькими способами:  

*   С помощью управляющей последовательности на системный динамик  
    
    > ```
    > echo -e "\a"
    > 
    > ```
    
*   С помощью утилиты **beep**   
    
    > ```
    > beep 659 120
    > 
    > ```
    
*   Консольными плеерами, например aplay, mplayer и т.д.
*   Синтезатором речи.

  
Первые два способа у меня не сработали, скорее всего из-за настроек терминала.  
  

#### Открытие/закрытие сидирома

  

> ```
> #!/bin/bash
> # открыть сидиром
> eject
> # закрыть сидиром
> eject -t
> 
> ```

  
«Это не интерфейс!» — скажете вы. Но факты  [доказывают](http://www.youtube.com/watch?v=Ol1kndxV6-E)  [обратное](http://bash.org.ru/quote/323695) .

**********
[bash](/tags/bash.md)
[select](/tags/select.md)
