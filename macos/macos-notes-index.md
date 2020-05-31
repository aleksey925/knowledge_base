Заметки о работе с MacOS
=============================

# Оглавление

1. <h3>[Настройка системы](#Настройка-системы)</h3>
    
    - [Горячие клавиши](#Горячие-клавиши)
    - [Отключение Dashboard](#Отключение-Dashboard)
    - [Отключение автоматического изменения порядка рабочих столов](#Отключение-автоматического-изменения-порядка-рабочих-столов)
    - [Расширение возможностей finder](#Расширение-возможностей-finder)
    - [Отображение в finder папок перед файлами](#Отображение-в-finder-папок-перед-файлами)
    - [Изменение значения TTL](#Изменение-значения-TTL)
    - [Отключение SIP](#Отключение-SIP)
    - [Отключение Spotlight](#Отключение-Spotlight)
    - [Отключение уведомления "default interactive shell is now zsh"](#Отключение-уведомления-default-interactive-shell-is-now-zsh)
    - [Создание MacOS приложения из простого скрипта](#Создание-MacOS-приложения-из-простого-скрипта)
    
2. <h3>[Установка/удаление программ](#Установка/удаление-программ)</h3>

    - <h4>[Brew](#Brew)</h4>
    
        - [Установка brew](#Установка-brew)
        - [Установка пакетов brew](#Установка-пакетов-brew)
        - [Удаление пакетов brew](#Удаление-пакетов-brew)
        - [Репозитори brew](#Репозитори-brew)
        - [Кэш brew](#Кэш-brew)
        - [Смена используемой версии пакета](#Смена-используемой-версии-пакета)
        - [Запуск и остановка фоновых служб brew](#Запуск-и-остановка-фоновых-служб-brew)
        - [Установка приложений](#Установка-приложений)
        
    - <h4>[PGK](#PGK)</h4>
    
        - [Удаление pkg](#Удаление-pkg)
        
    - <h4>[Python](#Python)</h4>
    
        - [Установка PyQt4](#Установка-PyQt4)
        - [Установка lxml](#Установка-lxml)
        - [Установка mpi4py](#Установка-mpi4py)
        - [Установка numpy](#Установка-numpy)

3. <h3>[Устранение проблем](#Устранение-проблем)</h3>
    
    - [Не удаляется VPN соединение](#Не-удаляется-VPN-соединение)

4. <h3>[Разное](#Разное)</h3>
    
    - [Создание автозагрузочной флешки с windows](#Создание-автозагрузочной-флешки-с-windows)


<a name='Настройка-системы'></a>
## Настройка системы


<a name='Горячие-клавиши'></a>
### Горячие клавиши

**Finder**
 
Комбинация              |   Описание 
------------------------|-------------------------------
shift + command + .     | Показать/скрыть скрытые файлы


<a name='Отключение-Dashboard'></a>
### Отключение Dashboard

Для отключение Dashboard необходимо в консоли выполнить следующие команды

```bash
defaults write com.apple.dashboard mcx-disabled -boolean YES
killall Dock
```

Если вы передумали, включить Dashboard можно следующей командой

```bash
defaults write com.apple.dashboard mcx-disabled -boolean NO
```


<a name='Отключение-автоматического-изменения-порядка-рабочих-столов'></a>
### Отключение автоматического изменения порядка рабочих столов

В macos есть фукнция, которая автоматически меняет порядок созданных рабочих 
столов. Упорядочивает она их по частоте использования. Такое поведение не 
всегда удобно, а иногда вообще доставляет сильный дискомфорт.

К счастью эту "супер крутую" фичу можно отключить. Для этого:

- откройте `Системные настройки`
- выберите пункт `Mission Control`
- снимите галочку с `Автоматически использовать последние расположение пространств`


<a name='Расширение-возможностей-finder'></a>
### Расширение возможностей finder

Для того, чтобы расширить возможности finder можно воспользоваться утилитой
[XtraFinder](https://www.trankynam.com/xtrafinder/). 

Некоторые из ее возможностей:

- добавление в контекстное меню кнопки "вырезать"
- делает возможным отобразить скрытые файлы и папки
- и т.д. и т.п.

К минусам можно отнести то, что приложение по сути пачит оригинальный finder и
из-за этого устанавливать его проблематично.


<a name='Отображение-в-finder-папок-перед-файлами'></a>
### Отображение в finder папок перед файлами

Полезные ссылки:

- [Finder – как показывать папки перед файлами](https://severe-programmer.com/manual/finder-kak-pokazyvat-papki-pered-faĭlami/) 

<a name='Изменение-значения-TTL'></a>
### Изменение значения TTL

Time to live (TTL) — число итераций или переходов, за который набор данных 
(пакет) может существовать до своего исчезновения.

Есть 2 способа изменить значение TTL:

1. Значение TTL меняется на ограниченое время. После перезагрузки
   системы будет востановлено значение по умолчанию.

    ```bash
    sudo sysctl -w net.inet.ip.ttl=<нужное значение ttl>
    ```
   
2. Установка постоянного значения TTL делается следующим образом:

    - создаем файл `/etc/sysctl.conf`
    - добавляем в него строку `net.inet.ip.ttl=<нужное значение ttl>`


<a name='Создание-MacOS-приложения-из-простого-скрипта'></a>
### Создание MacOS приложения из простого скрипта

В MacOS программы устроенны очень интересным образом, все программы это 
своебразные пакеты с набором файлов в том, числе и с самой программой. Вы 
можете сами убедиться в этом, нажав на программу правой кнопкой мыши выбрав 
-> `Показать содержимое пакета`.

Внутри вы скорей всего увидите примерно такую структуру:

```
.
└── Contents
    ├── MacOS
    └── Resources
```

`Resources` - содержит ресурсы программы (иконка программы и т.д.), в `MacOS` 
находиться сама программа, точнее исполнительный файл, который запускает 
программу. 

Кроме описанных выше каталогов, в корне папаки `Contents` могут находится еще 
различные текстовые файлы, в них хранятся метаданные пакета.

Теперь, когда мы уже имеем некоторое представление о том, что из себя 
представляют приложения в MacOS, давайте превратим обычный python скрипт в 
полноценное приложение.

Код скрипта:

```python
import sys
from PyQt5.QtWidgets import QApplication, QWidget


app = QApplication(sys.argv)

w = QWidget()
w.resize(250, 150)
w.move(300, 300)
w.setWindowTitle('Simple')
w.show()

sys.exit(app.exec_())
```

Для того, чтобы реализовать задуманное:

- Необходимо создать папку, которая будет иметь такое-же имя как наша программа.
- Потом, создаем внутри этой папки следующую структуру:
    ```
    .
    └── Contents
        ├── MacOS
        ├── Resources
        └── Info.plist
    ```
- Теперь можно разместить внутри папки MacOS наш скрипт, содержание которого 
  мы описали выше.
- Кроме самого python скрипта, потребуется еще bash скрипт, который
  будет указывать, как именно запускать python скрипт. Расположить его 
  необходимо так же в папке MacOS. В нашем случае скрипт будет называться 
  simpleapp и будет иметь следующие содержание:
  ```bash
  #!/usr/bin/env bash
  BASE_DIR=$( cd "$(dirname "${BASH_SOURCE[0]}")" ; pwd -P )
  cd "$BASE_DIR"
  python3 ./main.py
  ``` 
- Для того, чтобы скрипт успешно выполнился, он должен иметь права на 
  исполнение. Дать ему эти права можно следующей командой: `sudo chmod +x simpleapp`
- После создания скриптов, необходимо скопировать иконку изображения в папку 
  Resources. Иконка должна быть в формате `*.icns`.
- Теперь, необходимо создать в корне папки `Contents` файл `Info.plist` со 
  следующим содержанием:
    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
        <dict>
            <key>CFBundleDevelopmentRegion</key>
            <string>English</string>
            <key>CFBundleExecutable</key>
            <string>run.sh</string>
            <key>CFBundleIconFile</key>
            <string>app.icns</string>
            <key>CFBundleInfoDictionaryVersion</key>
            <string>6.0</string>
            <key>CFBundleName</key>
            <string>Simple app</string>
            <key>CFBundlePackageType</key>
            <string>APPL</string>
            <key>CFBundleIdentifier</key>
            <string>org.python.python</string>
            <key>CFBundleSignature</key>
            <string>????</string>
            <key>CFBundleGetInfoString</key>
            <string>Simple app 1.0.0</string>
            <key>CFBundleShortVersionString</key>
            <string>1.0.0</string>
            <key>CFBundleVersion</key>
            <string>1.0.0</string>
            <key>LSApplicationCategoryType</key>
            <string>public.app-category.entertainment</string>
    
            <key>NSRequiresAquaSystemAppearance</key>
            <string>false</string>
    
            <key>LSRequiresNativeExecution</key>
            <string>YES</string>
            <key>LSMinimumSystemVersion</key>
            <string>10.8</string>
        </dict>
    </plist>
    ```
- И в качестве завершающего этапа, необходимо к имени папки, которую мы 
  создали выполняя первый этап инструкции, добавить расширение `.app`.
  
После того, как были проделаны все описанные выше пункты, должнна получиться 
папка со следующим содержанием:

```
.
└── Simple\ app.app
    └── Contents
        ├── Info.plist
        ├── MacOS
        │   ├── main.py
        │   └── run.sh
        └── Resources
            └── app.icns

```  
 
Теперь, вы можете 2 раза щелкнуть по папке `Simple app` и будет запущен 
созданный нами скрипт. 

**P.S.**

Для того, чтобы убедиться в том, что у нашего приложения отображается 
правильная картинка, необходимо либо перезагрузить комьютер, либо просто 
выполнить `touch ./Simple app.app`.

Полезные ссылки:

- [Превращаем Bash скрипт в macOS *.app программу](http://mynotelist.blogspot.com/2016/06/macos-bash-macos-app.html)
- [Changing Mac OS X Application Icons Programmatically](https://www.sethvargo.com/replace-icons-osx)
- [Пример скрипта, который автоматически собирает python приложение в macos пакет](https://bitbucket.org/alex925/serial-notifier/src/master/tools/create_macos_package.py)
- [building-osx-app-bundle - stackoverflow](https://stackoverflow.com/questions/1596945/building-osx-app-bundle)


<a name='Отключение-SIP'></a>
### Отключение SIP

System Integrity Protection – это технология, направленная на защиту от 
вредоносного программного обеспечения. SIP серьезно ограничивает возможность 
любого изменения системных файлов сторонним программами или пользователем. 
Доступ к редактированию обладают только особые процессы, подписанные Apple. 
Например, один из таких процессов — обновление ПО.

Данный способ защиты может доставлять достаточно большое количество проблем, 
если компьюетр используется для ведения разработки, по этому лучше его 
отключить.

Для того, чтобы отключить SIP, вам необходимо выполнить следующие действия:

- выключаем mac
- при включении удерживаем комбинацию клавиш Command + R, чтобы загрузился 
  режим Recovery
- после загрузки выбираем в меню «Утилиты» приложение «Терминал»
- в терминале выполняем команду
  ```bash
  csrutil disable
  ```
  Если в будующем вы решите обратно включить данную защиту, то вам необходимо
  выполнить
  ```bash
  csrutil enable
  ```

<a name='Отключение-Spotlight'></a>
### Отключение Spotlight

Spotlight - это система «глобального» поиска по компьютеру. Она переодически 
запускает индексацию файлов, и тратит на это ресурсы компьютера (аккумулятор, 
процессор и т д). Если вы не пользутесь данной службой, то лучше будет ее 
отключить.

Сделать это можно командой:

```bash
sudo mdutil -a -i off
``` 

Включить обратно службу можно будет так:

```bash
sudo mdutil -a -i on
``` 

Полезные ссылки:

- [Как ускорить работу Mac и сократить расход батареи с помощью Spotlight](https://iphonegeek.me/instructions/dlya-chajnikov/255-kak-uskorit-rabotu-mac-i-sokratit-rashod-batarei-s-pomoschyu-spotlight.html)


<a name='Отключение-уведомления-default-interactive-shell-is-now-zsh'></a>
### Отключение уведомления "default interactive shell is now zsh"

После обновления на macos calatina при открытии терминала всегда появляется 
надоедливое сообщение:

>The default interactive shell is now zsh. To update your account to use zsh, 
>please run chsh -s /bin/zsh.

К счатью его можно легко отключить. Для этого необходимо добавить в файл
`~/.bash_profile` строку `export BASH_SILENCE_DEPRECATION_WARNING=1`.


<a name='Установка/удаление-программ'></a>
## Установка/удаление программ


<a name='Brew'></a>
### Brew

<a name='Установка-brew'></a>
#### Установка brew

Для того, чтобы установить brew вам необходимо сначала установить 
`Инструменты разработчика xcode`. После того как это сделано, можно приступить 
к установке самого brew, он ставится 1 просто командой:

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Полезные ссылки:
- http://dghubble.com/blog/posts/homebrew-os-x-package-management/

<a name='Установка-пакетов-brew'></a>
#### Установка пакетов brew

Для того, чтобы установить пакет вам нужно выполнить 1 простую команду:

```bash
brew install <FORMULA>
```

FORMULA - это название пакета, который будет установлен.

**Установка конкретной вресии**

При помощи команды выше будет установлена самая последняя версия приложения.
Если вам необходимо установить опредленную версию, то необходимо будет сделать 
немного больше телодвижений:

- заходим в репозиторий и ищем в Pull requests коммит с нужной нам версией
программы и переходим к нему
- открываем вкладку commits
- переходим к коду этого коммита
- нажимаем view, чтобы посмотреть код этого коммита
- теперь переходим в режим просмотра кода raw
- копируем ссылку из адресной строки и подставляем в команду
    ```bash
    brew install https://raw.github.com/dsr/homebrew/9b22d42f50fcbc5e52c764448b3ac002bc153bd7/Library/Formula/python3.rb
    ``` 
Таким способом можно поставить любую версию интресующей вас программы.

Ссылки на дополнительную информацию:

- https://github.com/Homebrew/homebrew/blob/master/share/doc/homebrew/Tips-N'-Tricks.md
- http://effectif.com/mac-os-x/installing-specific-version-of-homebrew-formula (инф. старая)


<a name='Удаление-пакетов-brew'></a>
#### Удаление пакетов brew

Удаление пакетов установленных при помощи brew производится командой:

```bash
brew rm <FORMULA>
```

Неприятным моментом является то, что эта команда удаляет только указанный пакет,
а все зависимости данного пакета остаются. Частичным решением данной проблемы 
является команда (выполнять ее нужно после удаления самого пакета):

```bash
brew rm $(join <(brew leaves) <(brew deps FORMULA))
```

- `FORMULA` - название пакета у которого необходимо найти неиспользуемые 
   зависимости
- `brew leaves` - выводит все пакеты, которые не зависят от других пакетов
- `brew deps` - выводит пакеты, от которых зависит данный пакет

Она удаляет пакеты, которые зависят от удаляемого пакета и, которые не 
зависят больше ни от каких других пакетов. К сожалению данный способ ни как не
решает проблему циклических зависимостей.

**rmtree**

rmtree - это расширение для brew, которое в теории должно позволять 1 командой
удалять пакет со всеми его завимостями. К сожлению, работает оно не всегда 
хорошо.

Для того, чтобы установить rmtree выполните:

```bash
brew tap beeftornado/rmtree
```

Теперь удалять пакеты можно следующей командой:

```bash
brew rmtree <package>
```


Если подитожить, то пока нет идеального способа удалить пакет со всеми его 
зависимостями. Лучший способ - это комбинация 2 описанных ранее способов.

Ссылки по теме:

- http://oryband.com/removing-homebrew-packages-with-dependencies/
- https://stackoverflow.com/questions/7323261/uninstall-remove-a-homebrew-package-including-all-its-dependencies


<a name='Репозитори-brew'></a>
#### Репозитори brew

Реппозитории в brew называеются tap.

Для того, чтобы добавить новый репозиторий можно пользоваться командой:

```bash
brew tap <REPO>
```

Список полезных реппозиториев:

- caskroom/cask


<a name='Кэш-brew'></a>
#### Кэш brew

Для того, чтобы получить путь к папке, где brew хранит исходные коды 
используемые для сборки приложения нужно выполнить в консоли:

```bash
brew --cache
```

<a name='Смена-используемой-версии-пакета'></a>
#### Смена используемой версии пакета

Brew позволяет вам установить одновременно несколько версий одного приложения, 
но по умолчанию у вас будет вызываться последняя установленная версия.
Для того, чтобы переключиться на предыдущую версию необходимо использовать 
команду switch.

Общий синтаксис команды

```bash
brew switch <FORMULA> <VERSION>
```

Пример установки python 3.5.1 как версии используемой по умолчанию 

```bash
brew switch python3 3.5.1
```


<a name='Запуск-и-остановка-фоновых-служб-brew'></a>
#### Запуск и остановка фоновых служб brew

https://robots.thoughtbot.com/starting-and-stopping-background-services-with-homebrew


<a name='Установка-приложений'></a>
#### Установка приложений

brew — «менеджер недостающих пакетов для macOS», как его называет создатель. 
Изначально он предназначается для компиляции приложений из исходных кодов. Но 
brew умеет не только это. Вместе с дополнением homebrew-cask он способен 
устанавливать приложения из «Терминала».

Для этого необходимо добавить в Homebrew реппозиторий cask:

```
brew tap caskroom/cask
```

После этого, можно устанавливать приложения просто 1 командой. Например команда
ниже производит установку браузера google chrome:

```
brew cask install google-chrome
```

Если вам требуется установить несколько приложений, то вы можете сделать это 1 
командой. Данная команда установит docker и google-chrome:

```
brew cask install google-chrome docker
```

Удаляются установленные приложения также просто как и устанавливаются:

```
brew cask uninstall google-chrome
```

<a name='PGK'></a>
### PGK

<a name='Удаление-pkg'></a>
#### Удаление pkg

1. Выведем список всех pkg в системе

    ```bash
    pkgutil --pkgs
    ```

2. Выбираем pkg, который нужно удалить и копируем его имя и вставляем его в команду

    ```bash
    pkgutil --forget <скопированное имя>
    ```
    
3. Перетащить app файл в корзину (если он остался)


<a name='Python'></a>
### Python

<a name='Установка-PyQt4'></a>
#### Установка PyQt4

```bash
brew install pyqt --with-python3
```

<a name='Установка-lxml'></a>
#### Установка lxml

Для того, чтобы установить библиотеку lxml нужно сначала установить сишную 
реализацию этой либы

```bash
brew install libxml2 
brew install libxslt
 
brew link libxml2 --force 
brew link libxslt --force
```

Теперь можно устанавливать обертку для python

```bash
pip3 install lxml
```

Теперь можно выполнить

```bash
brew unlink libxml2 
brew unlink libxslt
```

<a name='Установка-mpi4py'></a>
#### Установка mpi4py

```bash
brew install mpich
pip3 install mpi4py
```

<a name='Установка-numpy'></a>
#### Установка numpy

```bash
brew install homebrew/python/numpy --with-python3
```



<a name='Устранение-проблем'></a>
## Устранение проблем

<a name='Не-удаляется-VPN-соединение'></a>
### Не удаляется VPN соединение

Иногда может случиться так, что у вас не получается удалить старое VPN 
соединение (кнопка удаления не в меню "Сеть" не активна и в меню "Профили"
профиль для вашего VPN уже удален). 
В этом случае для того, чтобы избавиться от этого содинения вам придется 
удалить информацию о всех VPN содинениях.

```bash
sudo cp /Library/Preferences/com.apple.networkextension.plist /Library/Preferences/com.apple.networkextension.plist.old
sudo rm /Library/Preferences/com.apple.networkextension.plist 
```

P.S. Если у вас не получается удадить файл, то проверьте отключен ли у вас SIP. 

Полезные ссылки:

- [Where macOS keeps VPN Network configuration?](https://apple.stackexchange.com/questions/315797/where-macos-keeps-vpn-network-configuration)



<a name='Разное'></a>
## Разное

<a name='Создание-автозагрузочной-флешки-с-windows'></a>
### Создание автозагрузочной флешки с windows

Для создания автозагрузочной флешки сделайте следующее:

1. Подключите флешку к ПК

2. Откройте терминал и введите там команду

    ```
    diskutil list
    ``` 
    
    ```
    MBP-Aleksej:~ aleksey$ diskutil list
    /dev/disk0 (internal, physical):
       #:                       TYPE NAME                    SIZE       IDENTIFIER
       0:      GUID_partition_scheme                        *1.0 TB     disk0
       1:                        EFI EFI                     314.6 MB   disk0s1
       2:                 Apple_APFS Container disk1         1.0 TB     disk0s2
    
    /dev/disk1 (synthesized):
       #:                       TYPE NAME                    SIZE       IDENTIFIER
       0:      APFS Container Scheme -                      +1.0 TB     disk1
                                     Physical Store disk0s2
       1:                APFS Volume Macintosh HD — данные   185.7 GB   disk1s1
       2:                APFS Volume Preboot                 84.1 MB    disk1s2
       3:                APFS Volume Recovery                528.1 MB   disk1s3
       4:                APFS Volume VM                      1.1 GB     disk1s4
       5:                APFS Volume Macintosh HD            11.1 GB    disk1s5
    
    /dev/disk2 (external, physical):
       #:                       TYPE NAME                    SIZE       IDENTIFIER
       0:      GUID_partition_scheme                        *7.8 GB     disk2
       1:                        EFI EFI                     209.7 MB   disk2s1
       2:       Microsoft Basic Data WIN10                   7.6 GB     disk2s2
    ```
    
    Данная команда позволила нам узнать идентификатор, который система присвоила
    нашей флешке. В моем случае флешка называется `/dev/disk2`.

3. Теперь необходимо отформатировать флешку для работы в windows.

    ```
    diskutil eraseDisk MS-DOS "LIVEUSB" GPT /dev/disk2
    ```
    
    ```
    Started erase on disk2
    Unmounting disk
    Creating the partition map
    Waiting for partitions to activate
    Formatting disk2s2 as MS-DOS (FAT) with name LIVEUSB
    512 bytes per physical sector
    /dev/rdisk2s2: 14814936 sectors in 1851867 FAT32 clusters (4096 bytes/cluster)
    bps=512 spc=8 res=32 nft=2 mid=0xf8 spt=32 hds=255 hid=411648 drv=0x80 bsec=14843904 bspf=14468 rdcl=2 infs=1 bkbs=6
    Mounting disk
    Finished erase on disk2
    ```
    
    Если при форматировании фозникли ошибки, то необходимо выполнить немного другую
    команду:
    
    ```
    diskutil eraseDisk MS-DOS "LIVEUSB" MBR /dev/disk2
    ```

4. Примонтируем iso образ, который будем устанавливать на флешку

    ```
    hdiutil mount ~/Downloads/Win10.iso
    ```

5. Скопируем все файлы из образа на нашу флешку

    ```
    cp -rp /Volumes/Win10/* /Volumes/LIVEUSB/
    ```

Отлично, теперь все готово, можно извлечь флешку и использовать.

Полезные ссылки:

- [How Make a Windows 10 USB Using Your Mac - Build a Bootable ISO From Your Mac's Terminal](https://www.freecodecamp.org/news/how-make-a-windows-10-usb-using-your-mac-build-a-bootable-iso-from-your-macs-terminal/)