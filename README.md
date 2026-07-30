# MIDIShip Bridge

MIDIShip Bridge 0.5.0 —
VST3-инструмент-обёртка для Cubase, который загружает
другой VST3-инструмент внутрь себя и связывает его MIDI CC с четырьмя
моторизированными touch-фейдерами MIDIShip M в режиме HUI.

Плагин рассчитан на `Instrument Track`. Ноты, pitch bend и остальные MIDI-данные
проходят во вложенный Kontakt, Omnisphere или другой VST3i, а четыре выбранных
CC одновременно публикуются как параметры Bridge. Cubase MIDI Remote преобразует
их в HUI для моторов. В обратную сторону движение фейдера через отдельный
loopMIDI-порт снова становится обычным MIDI CC, который Cubase записывает в
MIDI-part. MIDI Send на Instrument Track для этого не требуется.

Перед первым тестом сохраните копию проекта Cubase.

## Текущая версия

- Имя `MIDIShip Bridge`, vendor `Andrey Zubets`, manufacturer code `AnZu`,
  plug-in code `MSB1`, bundle id `ru.andreyzubets.midishipbridge`.
- Компактный интерфейс с четырьмя реальными полосами MIDIShip M и заводским
  набором `CC1 / CC2 / CC11 / CC21`.
- Браузер установленных VST3 со строкой поиска, обновлением списка и ручным
  выбором файла или bundle.
- До 12 избранных инструментов, управление порядком списка и общий
  импорт/экспорт Favorites и пользовательских CC-пресетов.
- Выбранный в браузере или избранном инструмент загружается сразу.
- Опция `Auto-open instrument` по умолчанию открывает нативный редактор
  дочернего VST3 параллельно с компактным окном Bridge. Окно Bridge остаётся
  доступным для смены инструмента, избранного и CC; закрытый редактор можно
  вернуть кнопкой `Open instrument`.
- Для Kontakt 8.11 добавлен общий для процесса кэш описаний VST3. Первый Bridge
  выполняет медленное сканирование flat-модуля один раз, а последующие экземпляры
  с тем же путём создаются по сохранённому `PluginDescription` без повторного
  `findAllTypesForFile` при уже работающем Kontakt.
- Проект установщика Inno Setup 6 резервирует обновляемый MIDIShip Bridge и его
  актуальный профиль MIDI Remote. Устаревшие пользовательские MIDI Remote-
  профили установщик не удаляет.

Поскольку VST3 ID изменился в ветке 0.2, Cubase считает MIDIShip Bridge новым инструментом. Экземпляры
старого `HUI CC Bridge` в существующих проектах не заменяются автоматически.
Перед миграцией сохраните старый проект и перенесите дорожки осознанно.

## Схема работы

```mermaid
flowchart LR
    MIDI["MIDI notes + CC<br/>Instrument Track"] --> Bridge["MIDIShip Bridge VST3"]
    Bridge --> Child["Вложенный VST3i<br/>Kontakt / Omnisphere"]
    Child --> Audio["Stereo audio в Cubase"]
    Bridge -->|"4 CC mirror values<br/>4 назначения"| Remote["Cubase MIDI Remote"]
    Remote -->|"HUI Output"| Surface["MIDIShip M<br/>4 motorized touch faders"]
    Surface -->|"HUI Input + touch"| Remote
    Remote -->|"CC Loopback Output"| Loop["loopMIDI"]
    Loop --> MIDI
```

Bridge работает в процессе Cubase: хост видит его как инструмент, а Bridge
создаёт экземпляр дочернего VST3i. Это не отдельный процесс и не crash isolation.

Физический HUI-тракт и виртуальный CC loopback должны оставаться раздельными.
Для MIDIShip M Windows показывает одно и то же имя `TinyUSB Device` в списках
входов и выходов. Это нормально: выберите его и для `HUI Input`, и для
`HUI Output`; направление задаётся самим логическим портом. `CC Loopback Output`
назначается только на отдельный loopMIDI-порт.

Физический `TinyUSB Device` необходимо исключить из `All MIDI Inputs`. В режиме
HUI устройство передаёт служебные Control Change и heartbeat; без исключения они
могут попасть непосредственно на Instrument Track и смешаться с CC loopback.

## Как выбирается активный экземпляр

HUI обслуживает только один Bridge. Передача активна, когда профиль MIDI Remote
загружен и одновременно:

1. выбрана Instrument Track с `MIDIShip Bridge`;
2. на выбранной дорожке включён `Record Enable`.

При десяти экземплярах моторы следуют выбранной и вооружённой дорожке. Другие
Bridge продолжают воспроизводить свои инструменты, но не отправляют значения на
физическую поверхность. Желательно оставлять вооружённой только выбранную
дорожку, слушающую общий loopMIDI-порт.

## Быстрый старт

Полная актуальная настройка находится в
[docs/MANUAL_RU.md](docs/MANUAL_RU.md).

Кратко:

1. Установите `MIDIShip Bridge.vst3` и профиль Cubase MIDI Remote.
2. Создайте в loopMIDI порт, например `MIDIShip Bridge CC`.
3. В MIDI Remote назначьте `TinyUSB Device` на `HUI Input` и `HUI Output`, а
   `MIDIShip Bridge CC` — на `CC Loopback Output`.
4. Исключите `TinyUSB Device` из `All MIDI Inputs`.
5. Создайте Instrument Track с `MIDIShip Bridge`; его окно появится
   автоматически. Нажмите зелёную плитку `+`, найдите и выберите VST3i.
6. Выберите дорожку, включите `Record Enable` и направьте её MIDI-вход на
   loopMIDI либо на корректно настроенный `All MIDI Inputs`.

## Сборка в Windows

Требуются Windows x64, CMake 3.22+, Visual Studio с компонентом
`Desktop development with C++` и исходники JUCE 8. Node.js нужен для
JavaScript-проверок MIDI Remote.

```powershell
.\scripts\build.ps1 -JuceDir C:\SDK\JUCE -Configuration Release
```

Типичный результат:

```text
work\build-release\HuiCcBridge_artefacts\Release\VST3\MIDIShip Bridge.vst3
```

## Установка

Для установки из рабочей копии без готового `.exe` закройте Cubase и выполните:

```powershell
.\scripts\install.ps1
```

По умолчанию используются пользовательские каталоги:

- `%LOCALAPPDATA%\Programs\Common\VST3\MIDIShip Bridge.vst3`;
- `%USERPROFILE%\Documents\Steinberg\Cubase\MIDI Remote\Driver Scripts\Local\andrey_zubets\midiship_bridge`.

Для системного VST3-каталога запустите PowerShell с правами администратора:

```powershell
.\scripts\install.ps1 -Vst3Dir "$env:CommonProgramFiles\VST3"
```

Также подготовлен [installer/HuiCcBridge.iss](installer/HuiCcBridge.iss). Он
ставит VST3 в системный каталог, предлагает путь к пользовательскому профилю
Cubase, создаёт только собственную папку профиля и сохраняет резервные копии
обновляемых файлов вне сканируемых каталогов. Legacy-профиль
`openai\hui_cc_bridge` установщик не изменяет. Для
сборки нужен Inno Setup 6:

```powershell
.\scripts\build-installer.ps1 -Version 0.5.0
```

Установить компилятор можно с
[официальной страницы Inno Setup](https://jrsoftware.org/isdl.php).

## Ограничения

- Основная конфигурация — Windows x64, Cubase 15 и MIDIShip M в режиме HUI.
- Интерфейс и физический обмен рассчитаны на четыре моторизированных
  touch-фейдера. Полная восьмиканальная HUI-консоль не реализована.
- HUI-позиция имеет 14 бит, но записываемый MIDI CC — 7 бит, диапазон 0–127.
- Поддерживаются только дочерние VST3-инструменты, не эффекты и не VST2.
- Наружу выведен только основной стереовыход. Дополнительные выходы
  Kontakt/Omnisphere остаются внутри Bridge.
- Omnisphere 2.x в старом однофайловом формате блокируется; используйте bundle
  Omnisphere 3.x из каталога `Spectrasonics`.
- Дочерний VST3 работает в процессе Cubase и может завершить работу хоста при
  собственном сбое.
- Состояние очень большого инструмента может кратко приостановить обработку при
  сохранении проекта.
- MIDI Remote связывается с параметрами Bridge по фиксированному порядку;
  изменение этого контракта требует синхронного обновления VST3 и скрипта.

## Лицензирование и товарные знаки

Исходный код доступен по MIT License; подробности находятся в
[LICENSE.md](LICENSE.md). Проект использует JUCE 8. Перед сборкой и
распространением необходимо соблюдать AGPLv3 либо условия подходящей
коммерческой лицензии JUCE.

Cubase и VST являются товарными знаками Steinberg Media Technologies GmbH;
HUI связан с Mackie; Kontakt и Omnisphere принадлежат их правообладателям.
Проект не является официальным продуктом Steinberg, Mackie, Native Instruments
или Spectrasonics.

## Структура проекта

```text
Source/                                      VST3-обёртка, UI и CC mirror
cubase-midi-remote/andrey_zubets/midiship_bridge/ Cubase MIDI Remote-профиль
tests/                                       C++ и JavaScript проверки
installer/HuiCcBridge.iss                    проект Inno Setup 6
docs/MANUAL_RU.md                            актуальное руководство пользователя
scripts/build.ps1                            сборка и тесты
scripts/build-installer.ps1                  сборка setup.exe и SHA-256
scripts/install.ps1                          установка из рабочей копии
```
