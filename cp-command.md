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









