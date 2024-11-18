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

# prak3
https://github.com/wirenboard/wb-rules

## Типы правил defineRule
### whenChanged
Правило данного типа срабатывает при изменениях значений параметров или функций.

В примере ниже ко входу А1 устройства wb-gpio подключена кнопка, при нажатии на которую должно срабатывать реле_1 устройства MRM2-mini, при отжатии кнопки реле возвращается в исходное состояние. 

Пример правила типа whenChenged

```js
defineRule("test_rule", {
  whenChanged: "wb-gpio/A1_IN",  // Когда изменяется значение на входе A1
  then: function (newValue, devName, cellName) {
    dev["wb-mrm2-mini_2"]["Relay 1"] = newValue;  // Передаем это значение на реле 1 устройства wb-mrm2-mini_2
  }
});
```

### asSoonAs
Правила этого типа срабатывают в случае, когда значение, возвращаемое функцией, заданной в asSoonAs, становится истинным при том, что при предыдущем просмотре данного правила оно было ложным.

В пример ниже существуют две кнопки, подключенные к входам А2 и А3. При нажатии на первую кнопку, включаем реле, а при нажатии на вторую – выключаем. 

Пример правила типа asSoonAs

```js
// Правило для включения реле при нажатии на первую кнопку (A2)
defineRule({
  asSoonAs: function() {
    return dev["wb-gpio"]["A2_IN"];  // Срабатывает, когда кнопка на A2 нажата
  },
  then: function (newValue, devName, cellName) {
    dev["wb-mrm2-mini_2"]["Relay 2"] = true;  // Включаем реле 2
  }
});

// Правило для выключения реле при нажатии на вторую кнопку (A3)
defineRule({
  asSoonAs: function() {
    return dev["wb-gpio"]["A3_IN"];  // Срабатывает, когда кнопка на A3 нажата
  },
  then: function (newValue, devName, cellName) {
    dev["wb-mrm2-mini_2"]["Relay 2"] = false;  // Выключаем реле 2
  }
});
```
