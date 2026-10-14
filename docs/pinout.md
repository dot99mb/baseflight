# Подробная распиновка

## 1. Базовая таблица GPIO

Обе цели используют STM32F103 в 48-выводном корпусе. Номер корпуса приведён
для диагностики и разработки собственной платы; на готовой плате следует
ориентироваться по подписи площадки.

| GPIO | Вывод LQFP48 | Логическая функция Baseflight | Альтернативы и конфликты |
|---|---:|---|---|
| PA0 | 10 | `PWM1`, RC1, PPM | TIM2_CH1; вход калибровки газа |
| PA1 | 11 | `PWM2`, RC2 | TIM2_CH2, ADC1_IN1 для тока/RSSI |
| PA2 | 12 | `PWM3`, RC3 | TIM2_CH3, USART2_TX (GPS) |
| PA3 | 13 | `PWM4`, RC4 | TIM2_CH4, USART2_RX (GPS/SerialRX) |
| PA4 | 14 | VBAT | ADC1_IN4, делитель 10 кОм / 1 кОм указан в коде |
| PA5 | 15 | внешний ADC-пад rev5+ | ADC1_IN5; на старых платах ACC_INT2 |
| PA6 | 16 | `PWM5`, RC5 | TIM3_CH1, SoftSerial 1 RX, ADC1_IN6 |
| PA7 | 17 | `PWM6`, RC6 | TIM3_CH2, SoftSerial 1 TX, ADC1_IN7 |
| PB0 | 18 | `PWM7`, RC7 | TIM3_CH3, SoftSerial 2 RX, ADC1_IN8, sonar TRIG |
| PB1 | 19 | `PWM8`, RC8 | TIM3_CH4, SoftSerial 2 TX, ADC1_IN9, sonar ECHO, ток/RSSI |
| PB2 | 20 | SBUS inverter select | BOOT1 используется как GPIO |
| PB10 | 21 | I²C2 SCL | USART3_TX на NAZE-SP/FlexPort |
| PB11 | 22 | I²C2 SDA | USART3_RX на NAZE-SP/FlexPort/Spektrum |
| PB12 | 25 | SPI2 NSS | MAG_DRDY на старой NAZE rev4 |
| PB13 | 26 | SPI2 SCK | старый MPU interrupt конфликтует со SPI |
| PB14 | 27 | SPI2 MISO | — |
| PB15 | 28 | SPI2 MOSI | — |
| PA8 | 29 | `PWM9`, основной OUT1 | TIM1_CH1 |
| PA9 | 30 | USART1_TX | основной MSP/CLI и UART-телеметрия |
| PA10 | 31 | USART1_RX | основной MSP/CLI |
| PA11 | 32 | `PWM10`, основной OUT2 | TIM1_CH4; USB D− аппаратно, USB стек не используется |
| PA12 | 33 | буззер NAZE | USB D+ аппаратно, USB стек не используется |
| PA13 | 34 | SWDIO | SWD остаётся включённым |
| PA14 | 37 | SWCLK | SWD остаётся включённым |
| PA15 | 38 | свободен в прикладном коде | JTAG отключён |
| PB3 | 39 | LED0 NAZE | JTAG отключён для освобождения GPIO |
| PB4 | 40 | LED1 NAZE | JTAG отключён для освобождения GPIO |
| PB5 | 41 | опциональный аппаратный bind plug | только при сборке с `HARDWARE_BIND_PLUG` |
| PB6 | 42 | `PWM11` | TIM4_CH1; I²C1 SCL на CJMCU |
| PB7 | 43 | `PWM12` | TIM4_CH2; I²C1 SDA на CJMCU |
| PB8 | 45 | `PWM13` | TIM4_CH3; альтернативный sonar TRIG |
| PB9 | 46 | `PWM14` | TIM4_CH4; альтернативный sonar ECHO |
| PC13 | 2 | NAZE BARO/MPU interrupt; CJMCU LED0 | функция зависит от ревизии и датчика |
| PC14 | 3 | NAZE BARO EOC/MAG_DRDY; CJMCU LED1 | функция зависит от ревизии и датчика |
| PC15 | 4 | CJMCU LED2 | на NAZE прикладной функции нет |

Основание таблицы: [`drv_timer.c`](../src/drv_timer.c),
[`drv_uart.c`](../src/drv_uart.c), [`drv_system.c`](../src/drv_system.c) и
[`board.h`](../src/board.h).

## 2. Логические таймерные порты

Эта таблица является первичной: именно по ней `drv_pwm.c`, SoftSerial и сонар
находят GPIO.

| Порт Baseflight | GPIO | Таймер | Типичная подпись NAZE |
|---|---|---|---|
| PWM1 | PA0 | TIM2_CH1 | RC1 / PPM |
| PWM2 | PA1 | TIM2_CH2 | RC2 |
| PWM3 | PA2 | TIM2_CH3 | RC3 / UART2 TX |
| PWM4 | PA3 | TIM2_CH4 | RC4 / UART2 RX |
| PWM5 | PA6 | TIM3_CH1 | RC5 |
| PWM6 | PA7 | TIM3_CH2 | RC6 |
| PWM7 | PB0 | TIM3_CH3 | RC7 |
| PWM8 | PB1 | TIM3_CH4 | RC8 |
| PWM9 | PA8 | TIM1_CH1 | PWM/OUT1 |
| PWM10 | PA11 | TIM1_CH4 | PWM/OUT2 |
| PWM11 | PB6 | TIM4_CH1 | PWM/OUT3 |
| PWM12 | PB7 | TIM4_CH2 | PWM/OUT4 |
| PWM13 | PB8 | TIM4_CH3 | PWM/OUT5 |
| PWM14 | PB9 | TIM4_CH4 | PWM/OUT6 |

Каналы одного таймера имеют общую базовую частоту. Это особенно важно при
смешивании моторных выходов и сервоприводов: PA8/PA11 образуют группу TIM1,
PB6..PB9 — группу TIM4, PA6/PA7/PB0/PB1 — TIM3.

## 3. NAZE: назначения входов и выходов

### 3.1 Мультикоптер, параллельный PWM-приёмник

| Назначение | Логический порт | GPIO |
|---|---|---|
| RC1..RC8 | PWM1..PWM8 | PA0, PA1, PA2, PA3, PA6, PA7, PB0, PB1 |
| Motor 1 | PWM9 | PA8 |
| Motor 2 | PWM10 | PA11 |
| Motor 3 | PWM11 | PB6 |
| Motor 4 | PWM12 | PB7 |
| Motor 5 | PWM13 | PB8 |
| Motor 6 | PWM14 | PB9 |

Если микшер требует серво, `PWM9` и `PWM10` становятся Servo 1/2. Тогда
нумерация моторов в программном массиве начинается с `PWM11`.

### 3.2 Мультикоптер, PPM-приёмник

- PPM подключается к `PWM1` / PA0.
- В порядке выделения код использует выходы: PA8, PA11, PB6, PB7, PB8, PB9,
  затем PA6, PA7, PB0, PB1.
- При наличии сервоприводов PA8 и PA11 становятся Servo 1/2.
- При `GIMBAL_FORWARDAUX` PA6, PA7, PB0, PB1 также переводятся из моторов в
  Servo 3..6.

### 3.3 Самолёт/летающее крыло

При параллельном PWM:

- RC1..RC8 занимают PA0, PA1, PA2, PA3, PA6, PA7, PB0, PB1;
- моторы: PA8 и PA11;
- Servo 1..4: PB6, PB7, PB8, PB9.

При PPM:

- PPM: PA0;
- моторы: PA8 и PA11;
- Servo 1..4: PB6, PB7, PB8, PB9;
- Servo 5..8: PA6, PA7, PB0, PB1.

### 3.4 Ревизии NAZE

| Ревизия в коде | Как определяется | Основные аппаратные особенности |
|---|---|---|
| `NAZE32` | кварц 8 МГц | I²C2 PB10/PB11; старые линии interrupt/reset датчиков |
| `NAZE32_REV5` | кварц 12 МГц | внешний ADC PA5; MPU interrupt PC13; MAG_DRDY PC14 |
| `NAZE32_SP` | 12 МГц и MPU найден на SPI2 | MPU на PB12..PB15; I²C не инициализируется; доступен FlexPort USART3 PB10/PB11 |
| `NAZE32_REV6` | 12 МГц и MPU65xx найден на I²C | I²C2 PB10/PB11; внешний ADC PA5 |

Замечание: при запуске NAZE код пробует SPI2 для определения устройства даже
на не-SP ревизиях. Успешно найденный SPI MPU переводит плату в режим
`NAZE32_SP`; в этом режиме основной вызов `i2cInit()` пропускается.

## 4. CJMCU

### 4.1 Светодиоды и шины

| Функция | GPIO |
|---|---|
| LED0 | PC13 |
| LED1 | PC14 |
| LED2 | PC15 |
| I²C1 SCL | PB6 |
| I²C1 SDA | PB7 |
| Основной UART TX/RX | PA9 / PA10 |

В цель CJMCU не входят драйверы GPS, SoftSerial, телеметрии, сонара, SPI и
буззера. Наличие общих объявлений в заголовках не означает, что эти функции
собраны в прошивку.

### 4.2 Мультикоптер, PPM

| Назначение | Логический порт | GPIO |
|---|---|---|
| PPM | PWM1 | PA0 |
| Motor 1 | PWM7 | PB0 |
| Motor 2 | PWM14 | PB9 |
| Motor 3 | PWM8 | PB1 |
| Motor 4 | PWM13 | PB8 |

### 4.3 Мультикоптер, параллельный PWM

| Назначение | Логический порт | GPIO |
|---|---|---|
| RC1 | PWM1 | PA0 |
| RC2 | PWM2 | PA1 |
| RC3 | PWM3 | PA2 |
| RC4 | PWM4 | PA3 |
| RC5 | PWM9 | PA8 |
| RC6 | PWM10 | PA11 |
| Motor 1 | PWM7 | PB0 |
| Motor 2 | PWM14 | PB9 |
| Motor 3 | PWM8 | PB1 |
| Motor 4 | PWM13 | PB8 |

Порядок моторов CJMCU необычен, но именно так он задан массивами `multiPPM`
и `multiPWM` в [`drv_pwm.c`](../src/drv_pwm.c). Частота по умолчанию для этой
цели — 16 кГц, что соответствует коллекторным силовым ключам, а не обычным ESC.

Общие самолётные карты также компилируются для CJMCU, но исходники не содержат
описания разводки платы для такого применения. Использовать их без проверки
схемы CJMCU не рекомендуется.

## 5. Последовательные порты

| Порт | GPIO контроллера | Применение |
|---|---|---|
| USART1 | TX PA9, RX PA10 | всегда основной MSP/CLI; FrSky-телеметрия может временно занять его |
| USART2 | TX PA2, RX PA3 | GPS либо SerialRX (SBUS/SUMD/IBUS/Spektrum) |
| USART3 | TX PB10, RX PB11 | только FlexPort на NAZE-SP; также Spektrum при `spektrum_sat_on_flexport=1` |
| SoftSerial 1 | RX PA6, TX PA7 | NAZE; общий baud rate, максимум драйвера 38400 бод |
| SoftSerial 2 | RX PB0, TX PB1 | NAZE; общий baud rate, максимум драйвера 38400 бод |

Фактическое ограничение CLI для `softserial_baudrate` в этой версии — до
19200 бод, хотя драйвер объявляет абсолютный максимум 38400.

USART2 и `PWM3/PWM4` физически совпадают. При включении GPS или SerialRX код
исключает эти два таймерных порта из PWM-карты. При SerialRX обычные PWM-входы
приёмника отключаются полностью.

## 6. I²C и SPI

| Цель/режим | Шина | SCL/SCK | SDA/MISO | Дополнительно |
|---|---|---|---|---|
| NAZE rev4/rev5/rev6 | I²C2 | PB10 | PB11 | open-drain, внешние pull-up должны быть на плате/модуле |
| CJMCU | I²C1 | PB6 | PB7 | конфликтует с PWM11/PWM12 |
| NAZE-SP | SPI2 | PB13 | PB14 | MOSI PB15, CS/NSS PB12, mode 3 |

SPI2 настроен master, 8 бит, MSB first, CPOL=1, CPHA=2 edge, делитель 8. Один
GPIO CS PB12 используется либо для MPU6500, либо для обнаруживаемой флеш-памяти
M25P16; отдельных CS для нескольких внешних устройств код не предоставляет.

## 7. Аналоговые входы

| Измерение | GPIO / ADC | Условие |
|---|---|---|
| VBAT | PA4 / ADC1_IN4 | присутствует всегда в ADC; используется при `FEATURE_VBAT` |
| внешний ADC pad | PA5 / ADC1_IN5 | автоматически сканируется на NAZE rev5+ |
| ток или RSSI | PA1 / ADC1_IN1 | `power_adc_channel=1` или `rssi_adc_channel=1`; отнимает RC2 |
| ток или RSSI | PB1 / ADC1_IN9 | значение настройки `9`; отнимает RC8 |
| ток или RSSI | PA5 / ADC1_IN5 | значение настройки `5`; рассчитано на pad rev5+ |

PA4 ожидает уже масштабированное напряжение: комментарий драйвера фиксирует
делитель 10 кОм : 1 кОм. Для PA1/PB1/PA5 код не описывает внешний делитель или
датчик тока — сигнал обязан находиться в безопасном диапазоне ADC STM32.

## 8. Системные и внутренние сигналы NAZE

| Функция | GPIO | Примечание |
|---|---|---|
| LED0 | PB3 | активный низкий уровень |
| LED1 | PB4 | активный низкий уровень |
| Buzzer | PA12 | rev4: open-drain/PNP; rev5+: push-pull/NPN, логика инвертируется в драйвере |
| SBUS inverter select | PB2 | `INV_ON` при запуске SBUS |
| SWD | PA13, PA14 | доступен; отключается только полный JTAG |
| аппаратный bind plug | PB5 | активный низкий, только compile option `HARDWARE_BIND_PLUG` |

Линии внутренних датчиков зависят от ревизии: PC13 используется как MPU INT
на rev5+ или XCLR старого BMP085/MS5611-переключения на rev4; PC14 — BARO EOC
либо MAG_DRDY. PB12 был MAG_DRDY на rev4, а PB13 — старой линией MPU INT, но
оба входят в SPI2. Эти контакты нельзя считать свободными внешними GPIO.

## 9. Матрица основных конфликтов

| Если включено | Занятые контакты | Что становится недоступно/ограничено |
|---|---|---|
| PPM | PA0 | RC1 как отдельный PWM-вход |
| GPS на USART2 | PA2, PA3 | RC3, RC4; SerialRX |
| SerialRX на USART2 | PA3 (и периферия USART2) | все параллельные RC-входы; GPS без SoftSerial/FlexPort |
| SoftSerial | PA6, PA7, PB0, PB1 | RC5..RC8, поздние мотор/серво-выходы PPM-карты |
| Sonar (реально вызываемый режим) | PB0, PB1 | RC7, RC8 и SoftSerial 2 |
| ток/RSSI на ADC1_IN1 | PA1 | RC2 |
| ток/RSSI на ADC1_IN9 | PB1 | RC8, SoftSerial 2 TX, sonar ECHO |
| NAZE I²C2 | PB10, PB11 | USART3/FlexPort |
| CJMCU I²C1 | PB6, PB7 | PWM11/PWM12 |
| NAZE-SP SPI2 | PB12..PB15 | старые interrupt-линии; нет основной I²C и внешних I²C-датчиков |
| серво на PWM9/PWM10 | PA8, PA11 | первые два моторных выхода перенумеровываются на следующие порты |

