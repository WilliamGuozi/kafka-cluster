## Configuration
The configuration can modify by update `conf/*` file and env vars.

```bash
docker-compose build
```

## Usage
update file in each machine
```
file docker-compose.yml
ZOO_MY_ID
ZOO_SERVERS

file config/server.properties 
advertised.listeners
zookeeper.connect
listener.name.sasl_plaintext.plain.sasl.jaas.config

file config/consumer.properties
sasl.jaas.config

file config/producer.properties
sasl.jaas.config 
 ```

Start a cluster:

- ```docker-compose up -d ```

Destroy a cluster:

- ```docker-compose stop```

create topic 

```
#KAFKA
# 创建topic, 授权生产，消费
topic_list=(topic1 topic2 topic3)
for topic in "${topic_list[@]}"; do
  kafka-topics.sh --bootstrap-server localhost:9092 --create --topic "${topic}" --partitions 24 --replication-factor 2  --command-config config/producer.properties
done
```

authorize producer
```
# producer
user_list=(test)
topic_list=(topic1 topic2 topic3)
for topic in "${topic_list[@]}"; do
  for user in "${user_list[@]}"; do
    kafka-acls.sh --bootstrap-server localhost:9092  --add --allow-principal User:"${user}" --producer --topic "${topic}" --command-config config/producer.properties
  done
done
```

authorize consumer
```
user_list=(test)
group_list=(test)
topic_list=(topic1 topic2 topic3)
for topic in "${topic_list[@]}"; do
    for user in "${user_list[@]}"; do
      group=${user}
      kafka-acls.sh --bootstrap-server localhost:9092 --add --allow-principal User:"${user}" --consumer --topic "${topic}" --group "${group}" --command-config config/producer.properties
    done
done
```

revoke authorize
```
# 删除
topic_list=(Hello-Kafka)
user_list=(test)
for topic in "${topic_list[@]}"; do
  kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic "${topic}" --command-config config/producer.properties
  for user in "${user_list[@]}"; do
    kafka-acls.sh --bootstrap-server localhost:9092  --remove --allow-principal User:"${user}" --producer --topic "${topic}" --command-config config/producer.properties
    # for group in "${group_list[@]}"; do
    kafka-acls.sh --bootstrap-server localhost:9092 --remove --allow-principal User:"${user}" --consumer --topic "${topic}" --group "${group}" --command-config config/producer.properties
    # done
  done
done
```

query & reset

```
kafka-topics.sh --bootstrap-server localhost:9092 --list --command-config config/producer.properties
kafka-acls.sh --bootstrap-server localhost:9092 --list --command-config config/producer.properties
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group test --reset-offsets --to-earliest --command-config config/consumer.properties --all-topics --execute
```
