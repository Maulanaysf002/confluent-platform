# Lokasi konfig
Setiap komponen mempunyai topic sendiri dengan nama (.properties)

## zookeeper

lokasi utama config zookepeer adalah:
```
/etc/kafka/zookeeper.properties
```

## Kafka

lokasi file properties kafka 
```
/etc/kafka/server.properties
```

## Control center

file properties default yang digunakan:
```
/etc/confluent-control-center/control-center-production.properties
```

jika menggunakan **single node di wsl**, gunakan env dev:
```
ls /etc/confluent-control-center/
sudo systemctl cat confluent-control-center | grep ExecStart
sudo nano /usr/lib/systemd/system/confluent-control-center.service

sudo systemctl daemon-reload
sudo systemctl restart confluent-control-center
```