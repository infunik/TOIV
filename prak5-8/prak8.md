
```python
import xml.etree.ElementTree as ET
import paho.mqtt.client as mqtt
import json
from datetime import datetime
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np

HOST = "192.168.2.24"
PORT = 1883
KEEPALIVE = 60

# Топики для подписки
SUB_TOPICS = {
    '/devices/wb-msw-v3_21/controls/Max Motion': 'MaxMotion',
    '/devices/wb-msw-v3_21/controls/Current Motion': 'CurrentMotion',
    '/devices/wb-ms_11/controls/Temperature': 'Temperature'
}

# Список для хранения JSON данных
JSON_LIST = []

JSON_DICT = {value: 0 for value in SUB_TOPICS.values()}
JSON_DICT['time'] = str(datetime.now())

# Буфер для графиков
n = {
    "MaxMotion": [0] * 10,
    "CurrentMotion": [0] * 10,
    "Temperature": [0] * 10
}

# Инициализация графиков
x = np.arange(10)
fig, ax = plt.subplots()
lines = {}
for key in n.keys():
    lines[key], = ax.plot(x, n[key], label=key)
ax.legend(loc="upper left")
ax.set_ylim([0, 1500])
ax.set_xlim([0, 10])


def update_plot():
    """Обновление графиков в реальном времени."""
    for key, line in lines.items():
        line.set_ydata(n[key])
    fig.canvas.draw()
    fig.canvas.flush_events()


plt.ion()
plt.show()


def on_connect(client, userdata, flags, rc):
    """Обработчик события подключения к брокеру MQTT."""
    print("Connected with result code " + str(rc))
    for topic in SUB_TOPICS.keys():
        client.subscribe(topic)


def on_message(client, userdata, msg):
    """Обработчик получения сообщений из топиков MQTT."""
    payload = msg.payload.decode()
    topic = msg.topic

    if topic in SUB_TOPICS:
        param_name = SUB_TOPICS[topic]
        JSON_DICT[param_name] = payload
        JSON_DICT['time'] = str(datetime.now())

        JSON_LIST.append(JSON_DICT.copy())
        print(topic + " " + payload)

        with open('data.json', 'w') as file:
            json_string = json.dumps(JSON_LIST, indent=4)
            file.write(json_string)

        parse_json('data.json')


def parse_json(file_path):
    """Парсинг JSON-файла и обновление графиков."""
    with open(file_path, 'r') as json_file:
        data = json.load(json_file)

    for key in n.keys():
        n[key] = n[key][1:] + [int(float(data[-1][key]))]

    update_plot()


def main():
    """Главная функция для подключения к брокеру и запуска обработчиков."""
    client = mqtt.Client()
    client.on_connect = on_connect
    client.on_message = on_message
    client.connect(HOST, PORT, KEEPALIVE)
    client.loop_forever()


if __name__ == "__main__":
    main()
```