# Двенадцать советов по повышению безопасности Linux

## Защита терминала

  
 Для того, чтобы повысить безопасность системы, можно защитить консольный доступ к ней, ограничив root-пользователя в использовании определённых терминалов. Сделать это можно, задав терминалы, которые может использовать суперпользователь, в файле  `/etc/securetty`  .   
  
 Рекомендуется, хотя это и не обязательно, позволить суперпользователю входить в систему только из одного терминала, оставив остальные для других пользователей.   
  

## Напоминания о смене пароля

  
 В наши дни сложный пароль — вещь совершенно необходимая. Однако, ещё лучше, когда пароли регулярно меняют. Об этом легко забыть, поэтому хорошо бы задействовать какой-нибудь системный механизм напоминаний о возрасте пароля, и о том, когда его надо поменять.   
  
 Мы предлагаем вам два способа организации подобных напоминаний. Первый заключается в использовании команды  `chage`  , второй — в установке необходимых значений по умолчанию в  `/etc/login.defs`  .   
  
 Вызов команды  `chage`  выглядит так:   
  

```
$ chage -M 20 likegeeks
```

  
 Тут мы используем ключ  `-M`  для того, чтобы установить срок истечения актуальности пароля в днях.   
  
 Использовать эту команду можно и без ключей, тогда она сама предложит ввести необходимое значение:   
  

```
$ chage likegeeks
```

  
 Второй способ заключается в модификации файла  `/etc/login.defs`  . Вот пример того, как могут выглядеть интересующие нас значения. Вы можете изменить их на те, которые нужны вам:   
  

```
PASS_MAX_DAYS 10
PASS_MIN_DAYS 0
PASS_WARN_AGE 3
```

  
 Помните о том, что вам, если вы играете роль администратора, следует способствовать тому, чтобы пользователи применяли сложные пароли. Сделать это можно с помощью  [pam\_cracklib](https://likegeeks.com/linux-pam-easy-guide/#pam-cracklib-module)  .   
  
 После установки этой программы, вы можете перейти в  `/etc/pam.d/system-auth`  и ввести примерно следующее:   
  

```
password required pam_cracklib.so minlen=12 lcredit=-1 ucredit=-1 dcredit=-2 ocredit=-1
```

  

## Уведомления sudo

  
 Команда  `sudo`  , с одной стороны, упрощает жизнь, а с другой, может стать причиной проблем с безопасностью Linux, которые могут привести к непоправимым последствиям. Настройки  `sudo`  хранятся в файле  `/etc/sudoers`  . С помощью этого файла можно запретить обычным пользователям выполнять некоторые команды от имени суперпользователя. Кроме того, можно сделать так, чтобы команда  `sudo`  отправляла электронное письмо при её использовании, добавив в вышеупомянутый файл следующее:   
  

```
mailto yourname@yourdomain.com
```

  
 Также надо установить свойство  `mail_always`  в значение  `on`  :   
  

```
mail_always on
```

  

## Защита SSH

  
 Если мы говорим о безопасности Linux, то нам стоит вспомнить и о службе SSH. SSH — это важная системная служба, она позволяет удалённо подключаться к системе, и иногда это — единственный способ спасти ситуацию, когда что-то идёт не так, поэтому об отключении SSH мы тут не говорим.   
  
 Тут мы используем CentOS 7, поэтому конфигурационный файл SSH можно найти по адресу  `etc/ssh/sshd_config`  . Сканеры или боты, которых используют атакующие, пытаются подключиться к SSH по используемому по умолчанию порту 22.   
  
 Распространена практика изменения стандартного порта SSH на другой, неиспользуемый порт, например, на  `5555`  . Порт SSH можно изменить, задав нужный номер порта в конфигурационном файле. Например, так:   
  

```
Port 5555
```

  
 Кроме того, можно ограничить вход по SSH для root-пользователя, изменив значение параметра  `PermitRootLogin`  на  `no`  :   
  

```
PermitRootLogin no
```

  
 И, конечно, стоит отключить аутентификацию с применением пароля и использовать вместо этого публичные и приватные ключи:   
  

```
PasswordAuthentication no 
PermitEmptyPasswords no
```

  
 Теперь поговорим о тайм-аутах SSH. Проблему тайм-аутов можно решить, настроив некоторые параметры. Например, следующие установки подразумевают, что пакеты, поддерживающие соединение, будут автоматически отправляться через заданное число секунд:   
  

```
ServerAliveInterval 15
ServerAliveCountMax 3
TCPKeepAlive yes
```

  
 Настроив эти параметры, вы можете увеличить время соединения:   
  

```
ClientAliveInterval 30
ClientAliveCountMax 5
```

  
 Можно указать то, каким пользователям разрешено использовать SSH:   
  

```
AllowUsers user1 user2
```

  
 Разрешения можно назначать и на уровне групп:   
  

```
AllowGroup group1 group2
```

  

## Защита SSH с использованием Google Authenticator

  
 Для ещё более надёжной защиты SSH можно использовать двухфакторную аутентификацию, например, задействовав Google Authenticator. Для этого сначала надо установить соответствующую программу:   
  

```
$ yum install google-authenticator
```

  
 Затем запустить её для проверки установки:   
  

```
$ google-authenticator
```

  
 Так же нужно, чтобы приложение Google Authenticator было установлено на вашем телефоне.   
  
 Отредактируйте файл  `/etc/pam.d/sshd`  , добавив в него следующее:   
  

```
auth required pam_google_authenticator.so
```

  
 Теперь осталось лишь сообщить обо всём этом SSH, добавив следующую строку в файл  `/etc/ssh/sshd_config`  :   
  

```
ChallengeResponseAuthentication yes
```

  
 Теперь перезапустите SSH:   
  

```
$ systemctl restart sshd
```

  
 Когда вы попытаетесь войти в систему с использованием SSH, вам предложат ввести код верификации. Как результат, теперь SSH-доступ к вашей системе защищён гораздо лучше, чем прежде.   
  

## Мониторинг файловой системы с помощью Tripwire

  
 Tripwire — это замечательный инструмент для повышения безопасности Linux. Это — система обнаружения вторжений (HIDS).   
  
 Задача Tripwire заключается в том, чтобы отслеживать действия с файловой системой, следить за тем, кто меняет файлы, и когда происходят эти изменения.   
  
 Для того, чтобы установить Tripwire, нужен доступ к репозиторию EPEL. Это задача несложная, решить её можно следующими командами:   
  

```
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-9.noarch.rpm
$ rpm -ivh epel-release-7-9.noarch.rpm
```

  
 После установки репозитория EPEL, вы сможете установить и Tripwire:   
  

```
$ sudo yum install tripwire
```

  
 Теперь создайте файл ключей:   
  

```
$ tripwire-setup-keyfiles
```

  
 Вам предложат ввести сложный пароль для файла ключей. После этого можно настроить Tripwire, внеся изменения в файл  `/etc/tripwire/twpol.txt`  . Работать с этим файлом несложно, так как каждая строка оснащена содержательным комментарием.   
  
 Когда настройка программы завершена, следует её инициализировать:   
  

```
$ tripwire --init
```

  
 Инициализация, в ходе которой выполняется сканирование системы, займёт некоторое время, зависящее от размеров ваших файлов.   
  
 Любые модификации защищённых файлов расцениваются как вторжение, администратор будет об этом оповещён и ему нужно будет восстановить систему, пользуясь файлами, в происхождении которых он не сомневается.   
  
 По этой причине необходимые изменения системы должны быть подтверждены с помощью Tripwire. Для того, чтобы это сделать, используйте следующую команду:   
  

```
$ tripwire --check
```

  
 И вот ещё одна рекомендация, касающаяся Tripwire. Защитите файлы  `twpol.txt`  и  `twcfg.txt`  . Это повысит безопасность системы.   
  
 У Tripwire есть множество параметров и установок. Посмотреть справку по ней можно так:   
  

```
man tripwire
```

  

## Использование Firewalld

  
 Firewalld — это замена для  `iptables`  , данная программа улучшает сетевую безопасность Linux. Firewalld позволяет вносить изменения в настройки, не останавливая текущие соединения. Файрвол работает как сервис, который позволяет добавлять и менять правила без перезапуска и использует сетевые зоны.   
  
 Для того, чтобы выяснить, работает ли в настоящий момент  `firewalld`  , введите следующую команду:   
  

```
$ firewall-cmd --state
```

  
 ![](/images/359911417659531477a2cba6c68df107.png)   
 Просмотреть предопределённые сетевые зоны можно так:   
  

```
$ firewall-cmd --get-zones
```

  
 ![](/images/1aed7a9d902e11e68095be19e68953dc.png)   
 Каждая из этих зон имеет определённый уровень доверия.   
  
 Это значение можно обновить следующим образом:   
  

```
$ firewall-cmd --set-default-zone=<new-name>
```

  
 Получить подробные сведения о конкретной зоне можно так:   
  

```
$ firewall-cmd --zone=<zone-name> --list-all
```

  
 Просмотреть список всех поддерживаемых служб можно следующей командой:   
  

```
$ firewall-cmd --get-services
```

  
 ![](/images/9ed7325b84ff8375cb9c4cdeca9d41f7.png)   
 Затем можно добавлять в зону новые службы или убирать существующие:   
  

```
$ firewall-cmd --zone=<zone-name> --add-service=<service-name>
$ firewall-cmd --zone=<zone-name> --remove-service=<service-name>
```

  
 Можно вывести сведения обо всех открытых портах в любой зоне:   
  

```
$ firewall-cmd --zone=<zone-name> --list-ports
```

  
 Добавлять порты в зону и удалять их из неё можно так:   
  

```
$ firewall-cmd --zone=<zone-name> --add-port=<port-number/protocol>
$ firewall-cmd --zone=<zone-name> --remove-port=<port-number/protocol>
```

  
 Можно настраивать и перенаправление портов:   
  

```
$ firewall-cmd --zone=<zone-name> --add-forward-port=<port-number>
$ firewall-cmd --zone=<zone-name> --remove-forward-port=<port-number>
```

  
 Firewalld — это весьма продвинутый инструмент. Самое примечательное в нём то, что он может нормально работать, например, при внесении изменений в настройки, без перезапусков или остановок службы. Это отличает его от средства  `iptables`  , при работе с которым службу в похожих ситуациях нужно перезапускать.   
  

## Переход с firewalld на iptables

  
 Некоторые предпочитают файрвол  `iptables`  файрволу  `firewalld`  . Если вы пользуетесь  `firewalld`  , но хотите вернуться к  `iptables`  , сделать это довольно просто.   
  
 Сначала отключите  `firewalld`  :   
  

```
$ systemctl disable firewalld
$ systemctl stop firewalld
```

  
 Затем установите  `iptables`  :   
  

```
$ yum install iptables-services
$ touch /etc/sysconfig/iptables
$ touch /etc/sysconfig/ip6tables
```

  
 Теперь можно запустить службу  `iptables`  :   
  

```
$ systemctl start iptables
$ systemctl start ip6tables
$ systemctl enable iptables
$ systemctl enable ip6tables
```

  
 После всего этого перезагрузите компьютер.   
  

## Ограничение компиляторов

  
 Атакующий может скомпилировать эксплойт на своём компьютере и выгрузить его на интересующий его сервер. Естественно, при таком подходе наличие компиляторов на сервере роли не играет. Однако, лучше ограничить компиляторы, если вы не используете их для работы, как происходит в большинстве современных систем управления серверами.   
  
 Для начала выведите список всех бинарных файлов компиляторов из пакетов, а затем установите для них разрешения:   
  

```
$ rpm -q --filesbypkg gcc | grep 'bin'
```

  
 ![](/images/bc970b17b0714b6fe01bb51f50b51bd0.png)   
 Создайте новую группу:   
  

```
$ groupadd compilerGroup
```

  
 Затем измените группу бинарных файлов компилятора:   
  

```
$ chown root:compilerGroup /usr/bin/gcc
```

  
 И ещё одна важная вещь. Нужно изменить разрешения этих бинарных файлов:   
  

```
$ chmod 0750 /usr/bin/gcc
```

  
 Теперь любой пользователь, который попытается использовать  `gcc`  , получит сообщение об ошибке.   
  

## Предотвращение модификации файлов

  
 Иммутабельные файлы не может перезаписать ни один пользователь, даже обладающий root-правами. Пользователь не может модифицировать или удалить такой файл до тех пор, пока установлен флаг иммутабельности, снять который может лишь root-пользователь.   
  
 Несложно заметить, что эта возможность защищает вас, как суперпользователя, от ошибок, которые могут нарушить работу системы. Используя данный подход, можно защитить конфигурационные файлы или любые другие файлы по вашему желанию.   
  
 Для того, чтобы сделать любой файл иммутабельным, воспользуйтесь командой  `chattr`  :   
  

```
$ chattr +i /myscript
```

  
 ![](/images/8e07fe1db226ae3bfdd022209bf6d259.png)   
 Атрибут иммутабельности можно удалить такой командой:   
  

```
$ chattr -i /myscript
```

  
 ![](/images/eab702b92ebfd1902bed73816a5c7cf0.png)   
 Так можно защищать любые файлы, но помните о том, что если вы обработали таким образом бинарные системные файлы, вы не сможете их обновить до тех пор, пока не снимите флаг иммутабельности.   
  

## Управление SELinux с помощью aureport

  
 Нередко система принудительного контроля доступа SELinux оказывается, по умолчанию, отключённой. Это не влияет на работоспособность системы, да и работать с SELinux довольно сложно. Однако, ради повышения безопасности, SELinux можно включить, а упростить управление этим механизмом можно, используя  `aureport`  .   
  
 Утилита  `aureport`  позволяет создавать отчёты на основе  [лог-файлов](https://likegeeks.com/linux-syslog-server-log-management/)  аудита.   
  

```
$ aureport --avc
```

  
 ![](/images/3a99baa765a014c3da0547afeb01c0e2.png)   
 Список исполняемых файлов можно вывести следующей командой:   
  

```
$ aureport -x
```

  
 ![](/images/54415f63a2bc1b21649be77e481236a0.png)   
 Можно использовать  `aureport`  для создания полного отчёта об аутентификации:   
  

```
$ aureport -au -i
```

  
 ![](/images/626ee689dda852046c39cc70fff58bdc.png)   
 Также можно вывести сведения о неудачных попытках аутентификации:   
  

```
$ aureport -au --summary -i --failed
```

  
 ![](/images/53f2ca2ec4c09c7fdf5acdfb5f1c47f2.png)   
 Или, возможно, сводку по удачным попыткам аутентификации:   
  

```
$ aureport -au --summary -i --success
```

  
 ![](/images/2bf12f52fca32f7fc75678f7bdd3f1e8.png)   
 Утилита  `aureport`  значительно упрощает работу с SELinux.   
  

## Использование sealert

  
 В дополнение к  `aureport`  вы можете использовать хороший инструмент безопасности Linux, который называется  `sealert`  . Установить его можно так:   
  

```
$ yum install setools
```

  
 Теперь у нас есть средство, которое будет выдавать оповещения из файла  `/var/log/audit/audit.log`  и даст нам дополнительные сведения о проблемах, выявленных SELinux.   
  
 Использовать его можно так:   
  

```
$ sealert -a /var/log/audit/audit.log
```

  
 ![](/images/6de5475aa1b3a3f5403af0eaa9f33b57.png)   
 Самое интересное тут то, что в оповещениях можно найти советы о том, как решать соответствующие проблемы.


**********
[aureport](/tags/aureport.md)
[chattr](/tags/chattr.md)
[firewalld](/tags/firewalld.md)
[iptables](/tags/iptables.md)
[sealert](/tags/sealert.md)
[sudo](/tags/sudo.md)
[tripwire](/tags/tripwire.md)
