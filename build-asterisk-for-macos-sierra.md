Это обновление, точнее продолжение опубликованного [url=http://linsovet.org.ua/build-asterisk-for-os-x-yosemite]ранее[/url], совета, но уже и для macOS Sierra.

Началось с того, что после посещения [url=www.asterisk.org]официального сайта[/url] решил обновить Asterisk (про его установку как раз [url=http://linsovet.org.ua/build-asterisk-for-os-x-yosemite]предыдущая статья[/url]), ибо, на всё том же Apple Mac-mini 2010 с OS X Yosemite, всё так же работал Asterisk PBX версии 13.6.0, однако, если обновить ОС руки пока не доходили, тем более что апдейты на OS X Yosemite всё ещё регулярно выходили, то на обновление Asterisk PBX созрел.

Кому интересно, читайте далее...


Как ни странно, но обновление самого Asterisk с 13.6.0 до 13.16.0 на OS X Yosemite прошли без особых шаманских танцев описанных в предыдущей статье.

То есть мне просто было достаточно загрузить с официального сайта исходники и контрольную сумму архива исходников
[code]
$ [b]wget -c http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-13.16.0.tar.gz[/b]
$ [b]wget -c http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-13.16.0.sha256[/b]
[/code]

Проверяем целостность архива (должна совпадать с рассчитанной в asterisk-13.16.0.sha256)
[code]
$ [b]shasum -a 256 asterisk-13.16.0.tar.gz[/b]
[/code]

Разворачиваем архив:
[code]
$ [b]tar -zxf asterisk-13.16.0.tar.gz[/b]
[/code]

Переходим в исходники. ВНИМАНИЕ далее все команды выполняются от этой локации!
[code]
$ [b]cd /usr/local/src/asterisk-13.16.0[/b]
[/code]

Аналогично предыдущей статье конфигурируем Makefile (правда, каюсь, добавил PostgreSQL в нём варится realtime)
[code]
$ [b]./configure --without-netsnmp --with-postgres=/Library/PostgreSQL/9.4/[/b]
[/code]

Добавлять в заготовочные файлы инфо о типах [b]HAVE_HTONLL[/b] и [b]HAVE_NTOHLL[/b] не пришлось

Подгрузим поддержку mp3 с помощью
[code]
$ [b]./contrib/scripts/get_mp3_source.sh[/b]
[/code]

Сконфигурил в [url=https://wiki.asterisk.org/wiki/display/AST/Using+Menuselect+to+Select+Asterisk+Options]menuselect[/url] добавление дополнительных звуковых файлов и добавил поддержку mp3
[code]
$ [b]make menuselect[/b]
[/code]

собираем нормально, то есть не потребовалось как ранее [i]NOISY_BUILD=1[/i]
[code]
$ [b]make[/b]
[/code]

устанавливаем
[code]
$ [b]make install[/b]
[/code]

Примеры конфигов я не пересоздавал, этот шаг пропустил, так как у меня была настроенная и рабочая PBX,  но кто ставит первый раз, то можно создать
[code]
$ [b]make sample[/b]
[/code]

Всё бы вроде ничего, всё завертелось, закрутилось, но тут через какое-то время всё же решился обновить ОС, то есть перейти на [url=http://apple.com/ru/macos/sierra/]macOS Sierra[/url] (само обновление ОС ничего особенного за собой не несёт, поэтому выношу это за скобки данного описания).

Как ни странно, после обновления macOS, Asterisk PBX продолжал работать без проблем, но тут я решил пересобрать PBX и началось...
В процессе конфигурации сыпались ошибки и строка уже разрослась до умопомрачительной:
[code]
$ [b]LDFLAGS="-L/usr/local/lib -L/usr/lib" CFLAGS="-I/usr/local/include -I/usr/include" CPPFLAGS="-I/usr/local/include -I/usr/include" ./configure --without-netsnmp --with-postgres=/Library/PostgreSQL/9.4/ --prefix=/usr/local/ --includedir=/usr/local/include/ --libdir=/usr/local/lib/ --with-jansson=/usr/local/[/b]
[/code]

Начало конфигуриться нормально, но после menuselect и на запуске make вываливало следующую ошибку:

[code]
...
кусок лога
...
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/ranlib: file: libdb1.a(bt_debug.o) has no symbols
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/ranlib: file: libdb1.a(bt_debug.o) has no symbols
   [LD] astdb2sqlite3.o db1-ast/libdb1.a -> astdb2sqlite3
   [CC] astdb2bdb.c -> astdb2bdb.o
   [LD] astdb2bdb.o db1-ast/libdb1.a -> astdb2bdb
   [CC] chan_bridge_media.c -> chan_bridge_media.o
   [LD] chan_bridge_media.o -> chan_bridge_media.so
clang: [red]error:[/red] no such file or directory: '/usr/lib/bundle1.o'
make[1]: *** [chan_bridge_media.so] Error 1
make: *** [channels] Error 2
[/code]

Тут я упёрся и очень долго искал решение, но спустя пару дней случайно наткнулся на [url=https://github.com/leedm777/homebrew-asterisk]простой совет[/url]! Как я сам-то не додумался, что кроме обновления ОС и Xcode, надо нормально обновить и CLT!!!
[quote]
[b]Common problems[/b]

Compilation fails with error: /usr/lib/bundle1.o: No such file or directory

This happens when you try to build Asterisk without the XCode CLT installed. This also affects how GCC is built, so you will also want to reinstall GCC. The good news is that with the CLT installed, GCC should install via bottle, which is tons faster than building it from source.
[/quote]

[code]
$ [b]xcode-select --install[/b]
xcode-select: note: install requested for command line developer tools
[/code]

В общем после обновления command line tools для Xcode всё начало конфигурироваться и собираться без танцев с бубном

Подготовим к сборке [url=https://github.com/mobile-systems/asterisk-osx/raw/master/macos-sierra-asterisk-13.16.0-configure-with-xcode-clt-new.log]результат[/url]
[code]
$ [b]./configure --without-netsnmp --with-postgres=/Library/PostgreSQL/9.4/[/b]
[/code]

Подгрузим поддержку mp3 с помощью
[code]
$ [b]./contrib/scripts/get_mp3_source.sh[/b]
[/code]

Сконфигурил [url=https://wiki.asterisk.org/wiki/display/AST/Using+Menuselect+to+Select+Asterisk+Options]menuselect[/url]
[code]
$ [b]make menuselect[/b]
[/code]

Наконец таки собираем и [url=https://github.com/mobile-systems/asterisk-osx/raw/master/macos-sierra-asterisk-13.16.0-make-with-xcode-clt-new.log]лог результата сборки[/url]
[code]
$ [b]make[/b]
[/code]

Завершаем процесс установкой
[code]
$ [b]make install[/b]
[/code]

Всем удачи! И сборок без заморочек!
