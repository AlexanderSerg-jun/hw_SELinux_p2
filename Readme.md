Обеспечение работоспособности приложения при включенном SELinux

Выполним клонирование репозитория: git clone https://github.com/mbfx/otus-linux-adm.git

![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/74b296bb-2722-4210-9fc9-72c032a1c612)


Перейдём в каталог со стендом: cd otus-linux-adm/selinux_dns_problems

![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/2534db6a-525c-43cd-b3be-63fcf4259afb)
Развернём 2 ВМ с помощью vagrant: vagrant up

![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/65b546bd-2135-4368-9bab-f21a97b56ccf)

![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/00bf707c-88c5-4f6e-80ce-bc7a5a415d1e)

![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/b0e17b95-4733-44cc-bee5-cc38ba4600c6)

После того, как стенд развернется, проверим ВМ с помощью команды: vagrant status

![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/8940c15d-a4c5-4528-8ed3-81c3dd8d68d4)

Подключимся к клиенту: vagrant ssh client
![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/e3d4c52e-3053-4e7a-ac7e-4505d59e7e3d)

Попробуем внести изменения в зону: nsupdate -k /etc/named.zonetransfer.key
![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/3763ab31-7117-4c37-9f83-d21e2f9cd35a)
Как мы видим, внести изменения не получилось. 
Для этого воспользуемся утилитой audit2why: cat /var/log/audit/audit.log | audit2why
![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/ad4e9435-8658-4ca6-9242-cff03809ba0a)
Как мы видим, на клиенте отсутствуют ошибки. 

*Не закрывая сессию на клиенте, подключимся к серверу ns01 и проверим логи SELinux:
![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/654e9130-e3cb-4c1d-b402-5b670bc90754)

В логах мы видим, что ошибка в контексте безопасности. Вместо типа named_t используется тип etc_t.
Проверим данную проблему в каталоге /etc/named:
![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/58e2d573-1856-4c1b-a60f-030f3ce7770c)
Тут мы также видим, что контекст безопасности неправильный. Проблема заключается в том, что конфигурационные файлы лежат в другом каталоге. Посмотреть в каком каталоги должны лежать, файлы, чтобы на них распространялись правильные политики SELinux можно с помощью команды: sudo semanage fcontext -l 
![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/6c66e16d-a230-4f69-a04a-143914b52c00)

Изменим тип контекста безопасности для каталога /etc/named: sudo chcon -R -t named_zone_t /etc/named
![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/1c0667eb-9c44-46cc-adef-8ccdebc2075e)

Попробуем снова внести изменения с клиента: 
![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/b8616966-313b-4352-8b0e-dbdafad8e15b)
как видно из скриншота ошибки не возникло
Проверим применились ли изменения  используя dig (dig — утилита, предоставляющая пользователю интерфейс командной строки для обращения к системе DNS. Позволяет задавать различные типы запросов и запрашивать произвольно указываемые сервера. Является аналогом утилиты nslookup.)

![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/29b72726-1941-46a3-bc68-857f1706e299)

Перезагрузим хосты и проверим еще раз
![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/a2ce7cd8-b93e-4a8a-9577-d853dbb01315)
Как мы видим изменения сохранились.

![изображение](https://github.com/AlexanderSerg-jun/hw_SELinux_p2/assets/85576634/7a4eff6b-c683-41ba-8cbf-798fe2ff499e)




