## masuk ke zookeeper-shell

```
zookeeper-shell localhost:2181
zookeeper-shell localhost:2182
zookeeper-shell localhost:2183
```
## cek leader brokers

```
get /controller
```

## cek list brokers

```
get /brokers/ids
```

## Cek detail masing-masing broker

```
get /brokers/ids/0
get /brokers/ids/1
get /brokers/ids/2
```