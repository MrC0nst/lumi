# Lumi MQTT

MQTT агент для шлюза Xiaomi DGNWG05LM с прошивкой [OpenWRT 19.07.7](https://github.com/openlumi/openwrt/tags).  
Позволяет взаимодействовать со шлюзом через MQTT.

Взаимодействие | MQTT topic, получение | MQTT topic, управление
--- | --- | ---
Встроенный датчик освещения | lumi/illumination
Подсветка | lumi/lamp | lumi/lamp/set
Уведомление подсветкой |  | lumi/alarm/set
Кнопка | lumi/button/action
Воспроизводимый url, volume | lumi/audio/play | lumi/audio/play/set
Громкость | lumi/audio/volume | lumi/audio/volume/set
Голосовое уведомление |  | lumi/say/set
BLE устройства | lumi/{MAC} |

{MAC} - адрес bluetooth устройства.

[Примеры команд](#примеры-команд)

---
Вопросы и обсуждение - https://t.me/lumi_mqtt

---

### Замеченные проблемы:

На некоторых шлюзах при включении сканирования BLE устройств возникают проблемы с работой WiFi.  
Связь со шлюзом становиться не стабильной.

---
Для скачивания и работы необходимы пакеты node.js, git, mpc

Добавляем репозиторий со свежими версиями Node и устанавливаем необходимые пакеты:

```
wget https://openlumi.github.io/openwrt-packages/public.key -O /tmp/public.key
opkg-key add /tmp/public.key
echo 'src/gz openlumi https://openlumi.github.io/openwrt-packages/packages/19.07/arm_cortex-a9_neon' >> /etc/opkg/customfeeds.conf

opkg update && opkg install node git-http mpc mpd-full
```

Скачиваем:

```
mkdir /opt
cd /opt
git clone https://github.com/Beetle-II/lumi.git
cd lumi
cp config_example.json config.json
```

Изменяем конфигурационный файл config.json Указываем адрес своего сервера, логин и пароль

```json
{
  "sensor_debounce_period": 300,
  "sensor_treshhold": 50,
  "button_click_duration": 300,  
  "homeassistant": true,
  "use_ble": false,
  "delete_tts_file": true,
  "mqtt_url": "mqtt://адрес вашего сервера",
  "mqtt_topic": "lumi",
  "use_mac_in_mqtt_topic": false,
  "mqtt_options": {
    "port": 1883,
    "username": "логин сюда",
    "password": "пароль сюда",
    "keepalive": 60,
    "reconnectPeriod": 1000,
    "clean": true,
    "encoding": "utf8"
  }
}
```

Параметр | Описание
--- | ---
"homeassistant": true | уведомлять MQTT брокер об устройствах шлюза. Помогает добавлять устройства в HomeAssistant
"use_ble": false | включить сканирование и отправку данных c BLE устройств
||
"delete_tts_file": false | кешировать файлы TTS после воспроизведения
"button_click_duration": 300 | максимальное время в миллисекундах между нажатиями на кнопку
||
"sensor_debounce_period": 300 | период отправки данных о состоянии устройств (в секундах)
"sensor_treshhold": 50 | порог изменения состояния датчика, для моментальной отправки данных
||
"use_mac_in_mqtt_topic": true | добавить MAC шлюза в MQTT топики

Запускаем:

```
node /opt/lumi/lumi.js
```

Проверяем что пошли данные от датчиков и добавляем в автозапуск:

```
cd /opt/lumi
chmod +x lumi
cp lumi /etc/init.d/lumi
/etc/init.d/lumi enable
/etc/init.d/lumi start
```

---

### Обновить до актуальной версии:

```
/etc/init.d/lumi stop
cd /opt/lumi
git pull
/etc/init.d/lumi start
```

---

### Примеры команд:

Топик | Значение | Описание
---|---|---
lumi/light/set | {"color":{"r":50,"g":50,"b":50},"state":"ON"} | Включить подсветку
lumi/light/set | {"state": "ON", "flash": 2} | Серия из 2 вспышек. После серии из 2х вспышек alarm автоматически отключится
lumi/light/set | {"state":"OFF"} | Выключить подсветку
||
lumi/audio/play/set | "http://ep128.hostingradio.ru:8030/ep128" | Включить Радио Европа+
lumi/audio/play/set | "/tmp/test.mp3" | Воспроизвести локальный звуковой файл
lumi/audio/play/set | {"url": "https://air.radiorecord.ru:805/rr_320", "volume": 50} | Включить Радио рекорд с громкостью 50
lumi/audio/play/set | "STOP" | Выключить воспроизведение
||
lumi/audio/volume/set | 30 | Именить громкость на 30
||
lumi/say/set | {"text": "Привет", "volume": 80} | Произнести 'Привет' с громкостью 80
lumi/say/set | "Привет" | Произнести 'Привет'
||
lumi/alarm/set | {"color":{"r":50,"g":50,"b":50},"state":"ON"} | Включить мигание лампой
lumi/alarm/set | {"state":"OFF"} | Выключить мигание лампой. Параметры подсветки будут возвращены к предыдущим значениям


История версий
2021.04.05
Новый функционал:
Добавлен мультиклик на встроенной кнопке, добавлен autodiscovery для кнопки, добавлен новый режим alarm с автоматической отменой по счетчику миганий.
Исправления:
Исправлена ошибка превышения ограничений в 200 символов механизма Goggle Text-to-Speech, исправлена работа alarm (после выключения возвращаются предыдущие параметры подсветки), исправлен уровень встроенного датчика освещенности (коэффициент конверсии 0,25)
