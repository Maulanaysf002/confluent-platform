Pastikan semua service berjalan dengan command dibawah ini

## Jalankan zookeeper quorum

```
sudo /usr/bin/zookeeper-server-start /etc/kafka/zookeeper1.properties
sudo /usr/bin/zookeeper-server-start /etc/kafka/zookeeper2.properties
sudo /usr/bin/zookeeper-server-start /etc/kafka/zookeeper3.properties
```

## Jalankan service cp lain

```
sudo systemctl start confluent-server
sudo systemctl start confluent-schema-registry
sudo systemctl start confluent-kafka-connect
sudo systemctl start confluent-ksqldb
sudo systemctl start confluent-kafka-rest
sudo systemctl start confluent-control-center
```

---

## Arsitektur Lab

```
Datagen Source (pegawai)
        ↓
      Kafka Topic
        ↓
 JDBC Sink Connector
        ↓
   PostgreSQL (employee table)

```

## 1. Install Plugin

### Install Datagen Source Connector

```
sudo confluent-hub install confluentinc/kafka-connect-datagen:latest
```

hasilnya:

![install-plugin-datagen-ps2](../img/install-plugin-datagen-ps2.png)


### Install JDBC Sink Connector

```
sudo confluent-hub install confluentinc/kafka-connect-jdbc:latest
```

hasilnya:
![install-jdbc-sink-connector](../img/install-jdbc-sink-connector.png)

### restart kafka-connect

```
sudo systemctl restart confluent-kafka-connect
```

![restart-kafka-connect](../img/restart-kafka-connect.png)


### verifikasi plugin terinstall

```
curl --silent http://localhost:8083/connector-plugins | jq
```

![verifikasi-plugin-kafka-connect](../img/verifikasi-plugin-kafka-connect.png)


## 2. buat kafka topic

untuk lab ini saya ingin menggunakan data pegawai, maka buat topic dengan nama `pegawai-avro`.

```
kafka-topics \
  --bootstrap-server localhost:9092 \
  --create \
  --topic pegawai-avro \
  --partitions 1 \
  --replication-factor 1
```

### cek list topic pegawai-avro di c3

![tp-pegawai-avro-ps2](../img/tp-pegawai-avro-ps2.png)

## 3. deploy datagen source connector

```
nano datagen-pegawai.json
```

```
{
  "name": "datagen-pegawai-avro",
  "config": {
    "connector.class": "io.confluent.kafka.connect.datagen.DatagenConnector",
    "tasks.max": "1",
    "kafka.topic": "pegawai-avro",
    "iterations": "10",
    "max.interval": "1000",

    "schema.string": "{\"type\":\"record\",\"name\":\"pegawai\",\"fields\":[{\"name\":\"id\",\"type\":\"int\"},{\"name\":\"nama\",\"type\":\"string\"},{\"name\":\"jabatan\",\"type\":\"string\"},{\"name\":\"gaji\",\"type\":\"double\"},{\"name\":\"aktif\",\"type\":\"boolean\"}]}",

    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "value.converter": "io.confluent.connect.avro.AvroConverter",
    "value.converter.schema.registry.url": "http://localhost:8081"
  }
}
```

### create datagen soruce

pastikan berada didalam folder datagen-pegawai.json berada. dan jika tidak menggunakan template quick start hapus dari config.

```
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d @datagen-pegawai.json
```

hasilnya:

![create-datagen-avro-ps2](../img/create-datagen-avro-ps2.png)

### update connctor datagen

```
curl -X PUT http://localhost:8083/connectors/datagen-pegawai-avro/config \
  -H "Content-Type: application/json" \
  -d @datagen-pegawai.json
```

### delete connector

```
curl -X DELETE http://localhost:8083/connectors/datagen-pegawai-avro
```

### pause dan rerun datagen connector

```
curl -X PUT http://localhost:8083/connectors/datagen-pegawai-avro/pause

curl -X PUT http://localhost:8083/connectors/datagen-pegawai-avro/resume
```

### cek status source connector

```
curl http://localhost:8083/connectors/datagen-pegawai-avro/status | jq
```

hasilnya:

![running-datagen-pegawai-avro](../img/running-datagen-pegawai-avro.png)

### verifikasi data pegawai

```
kafka-avro-console-consumer \
  --bootstrap-server localhost:9092 \
  --topic pegawai-avro \
  --from-beginning \
  --property schema.registry.url=http://localhost:8081
```

hasil dari datagen:

![hasil-datagen-pegawai-ps2](../img/hasil-datagen-pegawai-ps2.png)

---

## sink connector

## 1. siapkan db postgres

saya menggunakan db postgres di windows, dan untuk menghubungkan ke wsl menggunakan ip `172.20.16.1` bisa di cek dengan command:

```
ip route show | grep -i default | awk '{ print $3}'
nc -zv 172.20.16.1 5432
```

konek menggunakan url:

```
postgres://postgres:password@172.20.16.1:5432/postgres
```

## 2. buat config JDBC Sink Connector

buat config jdbc dengan nama file `jdbc-sink-pegawai.json`

```
{
  "name": "jdbc-sink-postgres-pegawai",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
    "tasks.max": "1",
    "topics": "pegawai-avro",

    "connection.url": "jdbc:postgresql://172.20.16.1:5432/postgres",
    "connection.user": "postgres",
    "connection.password": "password",

    "auto.create": "true",
    "auto.evolve": "true",

    "insert.mode": "insert",
    "pk.mode": "none",

    "table.name.format": "kafka_sink_pegawai",

    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "value.converter": "io.confluent.connect.avro.AvroConverter",
    "value.converter.schema.registry.url": "http://localhost:8081",

    "value.converter.schemas.enable": "true",

    "errors.tolerance": "all",
    "errors.log.enable": "true",
    "errors.log.include.messages": "true"
  }
}
```

### jalankan jdbc sink connector

```
curl -X POST \
  -H "Content-Type: application/json" \
  --data @jdbc-sink-pegawai.json \
  http://localhost:8083/connectors
```

hasilnya:

![runnnin-jdbc-postgres-connector](../img/runnnin-jdbc-postgres-connector.png)


### cek status jdbc connector

```
curl http://localhost:8083/connectors/jdbc-sink-postgres-pegawai/status | jq
```

### hapus sink connector

```
curl -X DELETE http://localhost:8083/connectors/jdbc-sink-postgres-pegawai
```

## 3. setelah menggu lagging data akan masuk ke database

tampilan data di db:

![tampilan-data-pegawai-sink-postgres](../img/tampilan-data-pegawai-sink-postgres.png)

tampilan cunsumer di c3:

![consumer-pegawai-postgres](../img/consumer-pegawai-postgres.png)