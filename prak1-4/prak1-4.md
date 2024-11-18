### cran1.js
```js
// Задаем пороги температуры
var temp_on = 30;  // Температура для включения вентилятора
var temp_off = 25; // Температура для выключения вентилятора

// Определение правила
defineRule("fan_control_by_temperature", {
  whenChanged: "wb-msw-v3_21/Temperature",  // Датчик температуры на WB-MSW v3
  then: function (newValue, devName, cellName) {
    var temperature = parseFloat(newValue);  // Получаем текущее значение температуры

    // Включаем вентилятор, если температура выше порога включения
    if (temperature >= temp_on) {
      dev["wb-gpio"]["Relay1"] = true;  // Включаем реле вентилятора
      log("Fan turned ON. Current temperature: " + temperature);
    }
    // Выключаем вентилятор, если температура ниже порога выключения
    else if (temperature <= temp_off) {
      dev["wb-gpio"]["Relay1"] = false;  // Выключаем реле вентилятора
      log("Fan turned OFF. Current temperature: " + temperature);
    }
  }
});
```

### krane.js
```js
defineRule ("crane_control", {
  when: "wb-gpio/EXT1_R3A2" == true && "wb-gpio/EXT1_R3A3" == true,
    then: function (newValue, devName, cellName) {
      dev["wb-mwac_68/K2"] = true;
    }
  
});
```
