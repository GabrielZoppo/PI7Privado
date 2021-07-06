# PI7Privado

~~~python
# Importando bibliotecas:
import time
import psutil
import paho.mqtt.client as mqtt
import json
from datetime import datetime, timezone
import psycopg2

# fazer a conexão com o broker:
client = mqtt.Client()
client.username_pw_set("proj7", "integrador@")
client.connect("142.47.103.158", 1883)

# fazer conexão como o postgres
con = psycopg2.connect(host='localhost', database='postgres', user='projeto', password='!int7@')
cur = con.cursor()
dt = datetime.now(timezone.utc)
insertline = "INSERT INTO gabriel (id, nome_sensor, valor, data_pub)"

# looping para postar
while True:

    # tratamento da hora:
    dataHoraAtual = datetime.now()
    dataHoraTexto = dataHoraAtual.strftime("%d/%m/%Y %H:%M%S")

    # leitura e tratamento de dados da CPU:
    cpu_percent = psutil.cpu_percent(interval=1)
    cpu_p = float(cpu_percent)

    # declaracao de dados para postar no postgres
    metrica = "CPU"
    sensor = cpu_p
    message = metrica + "" + str(int(time.time())) + "\n"
    print('Enviando ao Banco de dados Grafite: %s' % message)

    try:
        # preparando e enviando dados para o postgres
        sensor_id = "1"
        nome_Sensor = metrica
        data_pub = dataHoraTexto
        valor_Sensor = sensor

        values = (sensor_id, nome_Sensor, valor_Sensor, data_pub)
        cur.execute(insertline, values)

        con.commit()

        # Enviando informações da CPU
        client.publish("PI7", json.dumps({"id": "Gabriel_cpu", "data": "%f" % cpu_p}))

        time.sleep(60)  # mandar as informações a cada minuto/evitar spam de dados

    except:
        print("erro na conexão e impressão de dados")  # Em caso de erro de conexão
        time.sleep(5)
~~~
