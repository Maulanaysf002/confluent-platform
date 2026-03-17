# ZOOKEEPER

## cek service zk
```bash
ss -lntp | grep 218
```

## masuk zksheel dengan ssl
```bash
KAFKA_OPTS="
-Djava.security.auth.login.config=/etc/kafka/zk_client_jaas.conf
-Dzookeeper.client.secure=true
-Dzookeeper.clientCnxnSocket=org.apache.zookeeper.ClientCnxnSocketNetty
-Dzookeeper.ssl.trustStore.location=/etc/kafka/secrets/zk.truststore.jks
-Dzookeeper.ssl.trustStore.password=password
" zookeeper-shell localhost:2281
```

## cara mengecek dan mematikan service zk yang running dengan daemon
```bash
sudo ss -tlnp | grep -E "2181|2182|2183|2281|2282|2283"
sudo kill <pid>
```

## cek port 2888
```bash
ss -tulpn | grep 2888
```

---

# BROKER (confluent-server)

---

# C3 (control-center)

## cek log control-center
`journalctl -u confluent-control-center -n 200 --no-pager`
`journalctl -u confluent-control-center -f`

---

# KSQLDB

## masuk ke ksqldb
```bash
ksql http://localhost:8088
```
---

# KAFKA_CONNECT

## cek journalctl
```bash
sudo journalctl -u confluent-kafka-connect -n 100 --no-pager
sudo journalctl -u confluent-kafka-connect -f | grep -E "started|ready|listening|LEADER|ERROR"
```

## cek port 8083
```bash
sudo ss -tlnp | grep 8083
```

---

# SERVICE

## perintah jalankan semua service
```bash
sudo systemctl start confluent-zookeeper
sudo systemctl start confluent-server
sudo systemctl start confluent-schema-registry
sudo systemctl start confluent-kafka-connect
sudo systemctl start confluent-ksqldb
sudo systemctl start confluent-kafka-rest
sudo systemctl start confluent-control-center
```

## restart services
```bash
sudo systemctl restart confluent-zookeeper
sudo systemctl restart confluent-server
sudo systemctl restart confluent-schema-registry
sudo systemctl restart confluent-kafka-connect
sudo systemctl restart confluent-ksqldb
sudo systemctl restart confluent-kafka-rest
sudo systemctl restart confluent-control-center
```

## perintah cpet stop semua
```bash
sudo systemctl stop confluent-control-center
sudo systemctl stop confluent-kafka-rest
sudo systemctl stop confluent-ksqldb
sudo systemctl stop confluent-kafka-connect
sudo systemctl stop confluent-schema-registry
sudo systemctl stop confluent-server
sudo systemctl stop confluent-zookeeper
```

---

# ANSIBLE

## testing koneksi
```bash
ansible -i inventory/hosts.yml all -m ping
ansible -i inventory/hosts.yml all -m command -a "whoami" --become
```

## verifikasi variable
```bash
ansible -i inventory/hosts.yml confluent-4 -m debug \
  -a "var=confluent_package_version,ssl_enabled,sasl_protocol" \
  -e "@vars/platform.yml"
```

## deploy seluruh service
```bash
ansible-playbook -i inventory/hosts.yml confluent.yml

# -v: task info
# -vv: + variabel
# -vvv: + SSH detail
ansible-playbook -i inventory/hosts.yml confluent.yml -vv 2>&1 | tee ~/deployment.log
```

## deploy service satupersatu
```bash
# Hanya deploy ZooKeeper dan Kafka
ansible-playbook -i inventory/hosts.yml confluent.yml \
  --tags zookeeper,kafka_broker

# Hanya deploy Schema Registry
ansible-playbook -i inventory/hosts.yml confluent.yml \
  --tags schema_registry

# Hanya deploy kafka connect
ansible-playbook -i inventory/hosts.yml confluent.yml \
  --tags kafka_connect

# Hanya deploy ksqldb
ansible-playbook -i inventory/hosts.yml confluent.yml \
  --tags ksql

# Hanya deploy kafka connect
ansible-playbook -i inventory/hosts.yml confluent.yml \
  --tags kafka_rest

# Hanya deploy Control Center
ansible-playbook -i inventory/hosts.yml confluent.yml \
  --tags control_center
```






