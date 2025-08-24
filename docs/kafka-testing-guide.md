# Kafka Testing Guide

## **Сценарии тестирования с мониторингом**

### Тест 1: **Базовая функциональность**

#### 1.1. Создание и проверка топика

```bash
# Создаем топик для тестирования
kafka-topics --create --topic load-test --bootstrap-server localhost:9092 --partitions 6 --replication-factor 1

# Проверяем создание
kafka-topics --describe --topic load-test --bootstrap-server localhost:9092
```

#### 1.2. Проверка начального состояния в мониторинге

- Открыть Grafana: <http://localhost:3000>
- Проверить, что топик `load-test` появился в метриках
- Убедиться что лаг = 0 для всех партиций

---

### Тест 2: **Сценарий реальной нагрузки**

#### 2.1. Запуск Producer с постоянной нагрузкой

```bash
# В одном терминале - непрерывная отправка с низкой скоростью
kafka-producer-perf-test --topic load-test --num-records 10000 --record-size 512 --throughput 50 --producer-props bootstrap.servers=localhost:9092 acks=1
```

#### 2.2. Запуск Consumer с задержкой (имитация медленной обработки)

```bash
# Во втором терминале - consumer с группой
kafka-console-consumer --topic load-test --bootstrap-server localhost:9092 --group slow-processors --property print.timestamp=true
```

#### 2.3. Наблюдение в Grafana

- Смотреть рост лага в реальном времени
- Наблюдать throughput producer vs consumer
- Записать максимальный лаг

---

### Тест 3: **Scaling Consumer Groups**

#### 3.1. Создание множественных consumer instances

```bash
# Терминал 1 - Consumer 1
kafka-console-consumer --topic load-test --bootstrap-server localhost:9092 --group scaling-test --property print.partition=true

# Терминал 2 - Consumer 2 (в новом терминале)
kafka-console-consumer --topic load-test --bootstrap-server localhost:9092 --group scaling-test --property print.partition=true

# Терминал 3 - Consumer 3
kafka-console-consumer --topic load-test --bootstrap-server localhost:9092 --group scaling-test --property print.partition=true
```

#### 3.2. Producer для scaling теста

```bash
# Отправляем сообщения с партиционированием
kafka-producer-perf-test --topic load-test --num-records 5000 --record-size 256 --throughput 100 --producer-props bootstrap.servers=localhost:9092
```

#### 3.3. Проверка распределения

```bash
# Проверяем как распределились партиции между consumers
kafka-consumer-groups --describe --group scaling-test --bootstrap-server localhost:9092
```

---

### Тест 4: **Failure Recovery**

#### 4.1. Остановка одного consumer

- Остановить Consumer 2 (Ctrl+C)
- Наблюдать в Grafana rebalancing
- Проверить перераспределение партиций

#### 4.2. Проверка состояния группы

```bash
kafka-consumer-groups --describe --group scaling-test --bootstrap-server localhost:9092
```

#### 4.3. Restart consumer

- Перезапустить Consumer 2
- Наблюдать новый rebalancing

---

### Тест 5: **Burst Load Testing**

#### 5.1. Burst Producer

```bash
# Отправка большой порции сообщений быстро
kafka-producer-perf-test --topic load-test --num-records 50000 --record-size 1024 --throughput -1 --producer-props bootstrap.servers=localhost:9092 batch.size=16384
```

#### 5.2. Медленный Consumer

```bash
# Consumer который не успевает
kafka-console-consumer --topic load-test --bootstrap-server localhost:9092 --group burst-test
```

#### 5.3. Анализ в Grafana

- Наблюдать резкий рост лага
- Измерить время recovery
- Проверить throughput метрики

---

### Тест 6: **Multi-Topic Scenario**

#### 6.1. Создание дополнительных топиков

```bash
kafka-topics --create --topic orders --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
kafka-topics --create --topic payments --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
kafka-topics --create --topic notifications --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
```

#### 6.2. Симуляция микросервисов

```bash
# Orders Producer
kafka-producer-perf-test --topic orders --num-records 1000 --record-size 256 --throughput 10 --producer-props bootstrap.servers=localhost:9092 &

# Payments Producer
kafka-producer-perf-test --topic payments --num-records 1000 --record-size 128 --throughput 15 --producer-props bootstrap.servers=localhost:9092 &

# Notifications Producer
kafka-producer-perf-test --topic notifications --num-records 2000 --record-size 64 --throughput 25 --producer-props bootstrap.servers=localhost:9092 &
```

#### 6.3. Multi-topic Consumers

```bash
# Orders Consumer
kafka-console-consumer --topic orders --bootstrap-server localhost:9092 --group orders-service &

# Payments Consumer
kafka-console-consumer --topic payments --bootstrap-server localhost:9092 --group payments-service &

# Notifications Consumer
kafka-console-consumer --topic notifications --bootstrap-server localhost:9092 --group notifications-service &
```

---

### Тест 7: **Offset Management**

#### 7.1. Reset Offsets тест

```bash
# Создать consumer group и обработать сообщения
kafka-console-consumer --topic load-test --bootstrap-server localhost:9092 --group offset-test --max-messages 100

# Проверить текущие offsets
kafka-consumer-groups --describe --group offset-test --bootstrap-server localhost:9092

# Reset к началу
kafka-consumer-groups --reset-offsets --to-earliest --group offset-test --topic load-test --bootstrap-server localhost:9092 --execute

# Reset к концу
kafka-consumer-groups --reset-offsets --to-latest --group offset-test --topic load-test --bootstrap-server localhost:9092 --execute

# Reset к конкретному offset
kafka-consumer-groups --reset-offsets --to-offset 500 --group offset-test --topic load-test --bootstrap-server localhost:9092 --execute
```

---

## 📊 **Мониторинг Checklist**

### Metrics в Grafana для каждого теста

#### Consumer Lag Metrics

- [ ] **kafka_consumer_lag_sum** - общий лаг по группе
- [ ] **kafka_consumer_lag** - лаг по партициям
- [ ] **kafka_consumer_lag_max** - максимальный лаг

#### Throughput Metrics

- [ ] **kafka_topic_partition_current_offset** - текущие offsets
- [ ] **kafka_topic_partition_oldest_offset** - старые offsets
- [ ] **kafka_topic_partition_partition_messages** - сообщения в партиции

#### Consumer Group Metrics

- [ ] **kafka_consumer_group_members** - активные члены группы
- [ ] **kafka_consumer_group_lag** - лаг по группам

### Real-time Observations

- [ ] Лаг растет при превышении producer > consumer throughput
- [ ] Лаг уменьшается при scaling consumers
- [ ] Rebalancing виден при добавлении/удалении consumers
- [ ] Recovery time после burst load измерен



---

## 📈 **Expected Results**

### Performance Benchmarks

- **Producer**: 1000+ msgs/sec на среднем железе
- **Consumer**: 500+ msgs/sec на среднем железе
- **End-to-end latency**: < 100ms для 95% сообщений
- **Recovery time**: < 30 секунд после rebalancing

### Monitoring Alerts (рекомендуемые пороги)

- **Consumer Lag > 1000**: Warning
- **Consumer Lag > 5000**: Critical
- **No active consumers**: Critical
- **Producer errors > 1%**: Warning
