```python
pip install paho-mqtt
```

```python
import xml.etree.ElementTree as ET
import paho.mqtt.client as mqtt
import json
from datetime import datetime, time
from threading import Timer

HOST = "192.168.2.24"  # IP адрес вашего брокера MQTT
PORT = 1883           # Порт для подключения к брокеру MQTT
KEEPALIVE = 60        # Интервал keepalive
SUB_TOPICS = {
    '/devices/wb-msw-v3_21/controls/Max Motion': 'MaxMotion',
    '/devices/wb-msw-v3_21/controls/Current Motion': 'CurrentMotion',
    '/devices/wb-msw-v3_21/controls/Sound Level': 'SoundLevel',
    '/devices/wb-ms_11/controls/Illuminance': 'Illuminance',
    '/devices/wb-ms_11/controls/Temperature': 'Temperature'
}

JSON_LIST = []
JSON_DICT = {value: 0 for value in SUB_TOPICS.values()}
JSON_DICT['time'] = str(datetime.now())

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

        # Запись данных в файл JSON (буферизация)
        with open('data.json', 'w') as file:
            json_string = json.dumps(JSON_LIST, indent=4)
            file.write(json_string)

def save_to_xml():
    """Сохранение данных в XML-файл."""
    root = ET.Element('records')

    for record in JSON_LIST:
        record_elem = ET.SubElement(root, 'record')
        for key, value in record.items():
            child = ET.SubElement(record_elem, key)
            child.text = str(value)

    tree = ET.ElementTree(root)
    tree.write('data.xml', encoding='utf-8', xml_declaration=True)

def periodic_xml_save(interval=60):
    """Периодическая функция для сохранения данных в XML."""
    save_to_xml()
    Timer(interval, periodic_xml_save, [interval]).start()

def parse_json(file_path):
    """Парсинг JSON-файла и вывод данных в консоль."""
    with open(file_path, 'r') as json_file:
        data = json.load(json_file)
        print(data)

def parse_xml(file_path):
    """Парсинг XML-файла и вывод данных в консоль."""
    tree = ET.parse(file_path)
    root = tree.getroot()
    for record in root.findall('record'):
        data = {child.tag: child.text for child in record}
        print(data)

def main():
    """Главная функция для подключения к брокеру и запуска обработчиков."""
    client = mqtt.Client()
    client.on_connect = on_connect
    client.on_message = on_message
    client.connect(HOST, PORT, KEEPALIVE)
    client.loop_start()

    # Запускаем периодическое сохранение в XML с интервалом 60 секунд
    periodic_xml_save(60)

    try:
        while True:
            pass  # Основной цикл, можно добавить другие задачи
    except KeyboardInterrupt:
        print("Завершение работы...")
    finally:
        client.loop_stop()
        client.disconnect()

if __name__ == "__main__":
    main()
```
