[![Foo](https://img.shields.io/badge/Version-3.3-brightgreen.svg?style=flat-square)](#versions)
[![Foo](https://img.shields.io/badge/Website-AlexGyver.ru-blue.svg?style=flat-square)](https://alexgyver.ru/)
[![Foo](https://img.shields.io/badge/%E2%82%BD$%E2%82%AC%20%D0%9D%D0%B0%20%D0%BF%D0%B8%D0%B2%D0%BE-%D1%81%20%D1%80%D1%8B%D0%B1%D0%BA%D0%BE%D0%B9-orange.svg?style=flat-square)](https://alexgyver.ru/support_alex/)

[![Foo](https://img.shields.io/badge/README-ENGLISH-brightgreen.svg?style=for-the-badge)](https://github-com.translate.goog/GyverLibs/GyverPID?_x_tr_sl=ru&_x_tr_tl=en)

# GyverPID
GyverPID - библиотека PID регулятора для Arduino
- Время одного расчёта около 70 мкс
- Режим работы по величине или по её изменению (для интегрирующих процессов)
- Возвращает результат по встроенному таймеру или в ручном режиме
- Встроенные калибровщики коэффициентов
- Режим работы по ошибке и по ошибке измерения
- Встроенные оптимизаторы интегральной суммы

### Совместимость
Совместима со всеми Arduino платформами (используются Arduino-функции)

### Документация
К библиотеке есть [расширенная документация](https://alexgyver.ru/GyverPID/)

## Содержание
- [Установка](#install)
- [Инициализация](#init)
- [Использование](#usage)
- [Пример](#example)
- [Версии](#versions)
- [Баги и обратная связь](#feedback)

<a id="install"></a>
## Установка
- Библиотеку можно найти по названию **GyverPID** и установить через менеджер библиотек в:
    - Arduino IDE
    - Arduino IDE v2
    - PlatformIO
- [Скачать библиотеку](https://github.com/GyverLibs/GyverPID/archive/refs/heads/main.zip) .zip архивом для ручной установки:
    - Распаковать и положить в *C:\Program Files (x86)\Arduino\libraries* (Windows x64)
    - Распаковать и положить в *C:\Program Files\Arduino\libraries* (Windows x32)
    - Распаковать и положить в *Документы/Arduino/libraries/*
    - (Arduino IDE) автоматическая установка из .zip: *Скетч/Подключить библиотеку/Добавить .ZIP библиотеку…* и указать скачанный архив
- Читай более подробную инструкцию по установке библиотек [здесь](https://alexgyver.ru/arduino-first/#%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0_%D0%B1%D0%B8%D0%B1%D0%BB%D0%B8%D0%BE%D1%82%D0%B5%D0%BA)

<a id="init"></a>
## Инициализация
```cpp
// Можно инициализировать объект тремя способами:
GyverPID regulator;                 // инициализировать без настроек (всё по нулям, dt 100 мс)
GyverPID regulator(kp, ki, kd);     // инициализировать с коэффициентами. dt будет стандартно 100 мс
GyverPID regulator(kp, ki, kd, dt); // инициализировать с коэффициентами и dt (в миллисекундах)
```

<a id="usage"></a>
## Использование
Смотри [документацию](https://alexgyver.ru/GyverPID/)
```cpp
datatype setpoint = 0;     // заданная величина, которую должен поддерживать регулятор
datatype input = 0;        // сигнал с датчика (например температура, которую мы регулируем)
datatype output = 0;       // выход с регулятора на управляющее устройство (например величина ШИМ или угол поворота серво)
  
datatype getResult();      // возвращает новое значение при вызове (если используем свой таймер с периодом dt!)
datatype getResultTimer(); // возвращает новое значение не ранее, чем через dt миллисекунд (встроенный таймер с периодом dt)
void setDirection(boolean direction);    // направление регулирования: NORMAL (0) или REVERSE (1)
void setMode(boolean mode);              // режим: работа по входной ошибке ON_ERROR (0) или по изменению ON_RATE (1)
void setLimits(int min_output, int max_output);    // лимит выходной величины (например для ШИМ ставим 0-255)
void setDt(int16_t new_dt);              // установка времени дискретизации (для getResultTimer)
float Kp = 0.0;
float Ki = 0.0;
float Kd = 0.0;
```

<a id="example"></a>
## Пример
Остальные примеры смотри в **examples**!
```cpp
/*
   Пример работы ПИД регулятора в автоматическом режиме по встроенному таймеру
   Давайте представим, что на 3 пине у нас спираль нагрева, подключенная через мосфет,
   управляем ШИМ сигналом
   И есть какой то абстрактный датчик температуры, на который влияет спираль
*/

// перед подключением библиотеки можно добавить настройки:
// сделает часть вычислений целочисленными, что чуть (совсем чуть!) ускорит код
// #define PID_INTEGER

// режим, при котором интегральная составляющая суммируется только в пределах указанного количества значений
// #define PID_INTEGRAL_WINDOW 50
#include "GyverPID.h"

GyverPID regulator(0.1, 0.05, 0.01, 10);  // коэф. П, коэф. И, коэф. Д, период дискретизации dt (мс)
// или так:
// GyverPID regulator(0.1, 0.05, 0.01);	// можно П, И, Д, без dt, dt будет по умолч. 100 мс

void setup() {
  regulator.setDirection(NORMAL); // направление регулирования (NORMAL/REVERSE). ПО УМОЛЧАНИЮ СТОИТ NORMAL
  regulator.setLimits(0, 255);    // пределы (ставим для 8 битного ШИМ). ПО УМОЛЧАНИЮ СТОЯТ 0 И 255
  regulator.setpoint = 50;        // сообщаем регулятору температуру, которую он должен поддерживать

  // в процессе работы можно менять коэффициенты
  regulator.Kp = 5.2;
  regulator.Ki += 0.5;
  regulator.Kd = 0;
}

void loop() {
  int temp;                 // читаем с датчика температуру
  regulator.input = temp;   // сообщаем регулятору текущую температуру

  // getResultTimer возвращает значение для управляющего устройства
  // (после вызова можно получать это значение как regulator.output)
  // обновление происходит по встроенному таймеру на millis()
  analogWrite(3, regulator.getResultTimer());  // отправляем на мосфет

  // .getResultTimer() по сути возвращает regulator.output
}
```

<a id="versions"></a>
## Версии
- v1.1 - убраны дефайны
- v1.2 - возвращены дефайны
- v1.3 - вычисления ускорены, библиотека облегчена
- v2.0 - логика работы чуть переосмыслена, код улучшен, упрощён и облегчён
- v2.1 - integral вынесен в public
- v2.2 - оптимизация вычислений
- v2.3 - добавлен режим PID_INTEGRAL_WINDOW
- v2.4 - реализация внесена в класс
- v3.0
    - Добавлен режим оптимизации интегральной составляющей (см. доку)
    - Добавлены автоматические калибровщики коэффициентов (см. примеры и доку)
- v3.1 - исправлен режиме ON_RATE, добавлено автоограничение инт. суммы
- v3.2 - чуть оптимизации, добавлена getResultNow
- v3.3 - в тюнерах можно передать другой обработчик класса Stream для отладки

<a id="feedback"></a>
## Баги и обратная связь
При нахождении багов создавайте **Issue**, а лучше сразу пишите на почту [alex@alexgyver.ru](mailto:alex@alexgyver.ru)  
Библиотека открыта для доработки и ваших **Pull Request**'ов!
