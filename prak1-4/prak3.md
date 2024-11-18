
### 1 правило.js
```js
var temperatureSensor = "wb-msw-v3_21/Temperature";

// Задайте здесь имя реле вентилятора
var fanRelay = "wb-mr3_56/K2";

// Установите пороговые значения температуры
var temperatureOn = 10; // Температура, выше которой вентилятор включается
var temperatureOff = 0; // Температура, ниже которой вентилятор выключается

// Функция, которая выполняется при изменении температуры
defineRule("FanControlByTemperature", {
  whenChanged: temperatureSensor,
  then: function (newValue, devName, cellName) {
    log("Текущая температура: " + newValue + "°C");

    if (newValue >= temperatureOn) {
      dev[fanRelay] = true;
      log("Вентилятор включен");
    }
    else if (newValue <= temperatureOff) {
      dev[fanRelay] = false;
      log("Вентилятор выключен");
    }
  }
});
```

