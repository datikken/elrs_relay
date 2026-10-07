# ELRS Relay v2 — WiFi-CRSF-мост через VPS с маршрутизацией пар

## Архитектура

```
TX12 --3 провода--> ESP32 #1 (пилот)
                     | WiFi STA -> роутер -> интернет
                     |
                     +-- UDP --> VPS (ретранслятор) <-- UDP --+
                          (pairId + role)                     |
                                                 ESP32 #2 (дрон)
                                                 | WiFi STA
                                                 |
                                          4 провода -- JR-модуль Bandit
```

## Структура пакета

```
[PAIR_ID: 4 байта] [ROLE: 1 байт] [CRSF-кадр: N байт]
                    0x00 = пилот
                    0x01 = дрон
```

Ретранслятор разбирает заголовок, находит пару по pairId и пересылает чистый CRSF второй стороне.

## Файлы

| Файл | Назначение |
|---|---|
| `platformio.ini` | PlatformIO: два окружения (pilot / drone) |
| `src/pilot/main.cpp` | ESP32 #1 — в пульте |
| `src/drone/main.cpp` | ESP32 #2 — рядом с Bandit |
| `relay.py` | сервер-ретранслятор для VPS |

## Настройка перед прошивкой

В обоих `main.cpp` замените:

```cpp
#define WIFI_SSID       "ВАШ_WIFI"
#define WIFI_PASS       "ВАШ_ПАРОЛЬ"
#define RELAY_IP        "185.xxx.xxx.xxx"
```

Убедитесь, что PAIR_ID одинаковый для пилота и дрона.

## Прошивка

```bash
pio run -e pilot -t upload
pio run -e drone -t upload
```

## Запуск ретранслятора на VPS

```bash
scp relay.py user@IP_VPS:~/
sudo ufw allow 14550/udp
python3 relay.py
```

## Несколько пар

У каждой пары свой PAIR_ID (4 байта). Измените PAIR_ID_BYTE0..3 в обоих `main.cpp`.

## Порядок включения

1. Включить дрон → подождать 5 сек
2. Включить пульт → подождать 5 сек
3. Проверить стики

## Порядок выключения

1. Выключить пульт
2. Выключить дрон

Some usefull docs:
https://docs.platformio.org/en/latest/frameworks/arduino.html
https://docs.espressif.com/projects/arduino-esp32/en/latest/api/wifi.html
https://github.com/espressif/arduino-esp32/blob/master/libraries/WiFi/src/WiFi.h
