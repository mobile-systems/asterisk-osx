***Собираем Asterisk для OS X Yosemite*** [Статья изначально, 29 Октябрь 2015, мною была размещена на linsovet.org.ua](http://linsovet.org.ua/build-asterisk-for-os-x-yosemite)

Уже давно в голове вынашивалась идея запустить Asterisk под работающим почти круглосуточно Mac-mini с OS X Yosemite, однако готовой сборки на просторах интернет не попадалось под руку.

Если кому интересен это краткий совет, тогда читайте далее...

росто собрать не получилось, по рецепту домашнего сайта https://wiki.asterisk.org/wiki/display/AST/Checking+Asterisk+Requirements
а на http://www.voip-info.org/wiki/view/Building+Asterisk+on+MacOSX приведён рецепт для старых версий OS X, нынче не актуальный, хотя с этой ссылки можно почерпнуть информацию как настроить автоматический старт Asterisk при перезагрузке ОС.

Пришлось гуглить и на просторах интернет попалась инструкция сборки в глубинах ГитХаба https://gist.github.com/morgant/49ca14d05d8b5469e5d1 и по ссылке http://www.remesov.ru/2015/01/25/build-asterisk-13-1-0-in-mac-os-x-10-10/ много полезного, но ставить brew нет никакого желания.

В общем моя сборка свелась к загрузке исходников
[code]
$ [b]wget -c http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-13-current.tar.gz[/b]
[/code]

Установки зависимостей (в данном случае jansson) и естественно должен быть установлен Xcode с утилитами командной строки.

Разворачиваем архив:
[code]
$ [b]tar xvf asterisk-13-current.tar.gz[/b]
[/code]

Конфигурируем Makefile
[code]
$ [b]./configure --without-netsnmp[/b]
[/code]

Добавляем в заготовочные файлы инфо о типах
[code]
$ [b]echo "#define HAVE_HTONLL 1" >> include/asterisk/autoconfig.h[/b]
$ [b]echo "#define HAVE_NTOHLL 1" >> include/asterisk/autoconfig.h[/b]
[/code]

собираем 
[code]
$ [b]make NOISY_BUILD=1[/b]
[/code]

устанавливаем
[code]
$ [b]make install[/b]
[/code]

и создаём примеры конфигов, аккуратнее, если есть старые он их затрёт, тогда можно этот шаг пропустить.
[code]
$ [b]make sample[/b]
[/code]

[code]TODO:[/code] пока что не получилось собрать http://sourceforge.net/projects/chan-sccp-b/ для Cisco-телефонов, приходится пользоваться штатным skinny.
