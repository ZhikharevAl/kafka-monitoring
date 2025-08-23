## Тесты Kafka

### 1. Создание топика

**Описание**: Создаём тестовый топик с 3 партициями

```bash
kafka-topics --create --topic test-topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
```

### 2. Просмотр списка топиков

**Описание**: Проверяем, что топик создался

```bash
kafka-topics --list --bootstrap-server localhost:9092
```

### 3. Детали топика

**Описание**: Смотрим подробную информацию о топике

```bash
kafka-topics --describe --topic test-topic --bootstrap-server localhost:9092
```

### 4. Отправка сообщений (Producer)

**Описание**: Запускаем producer для отправки сообщений

```bash
kafka-console-producer --topic test-topic --bootstrap-server localhost:9092
```

*После запуска введите несколько сообщений и нажмите Ctrl+C*

### 5. Чтение сообщений (Consumer)

**Описание**: Читаем все сообщения с начала топика

```bash
kafka-console-consumer --topic test-topic --bootstrap-server localhost:9092 --from-beginning
```

### 6. Consumer с группой

**Описание**: Создаём потребителя в группе "test-group"

```bash
kafka-console-consumer --topic test-topic --bootstrap-server localhost:9092 --group test-group --from-beginning
```

### 7. Список Consumer Groups

**Описание**: Показывает все активные группы потребителей

```bash
kafka-consumer-groups --list --bootstrap-server localhost:9092
```

### 8. Детали Consumer Group

**Описание**: Информация о смещениях и лагах группы

```bash
kafka-consumer-groups --describe --group test-group --bootstrap-server localhost:9092
```

### 9. Тест производительности Producer

**Описание**: Отправляем 1000 сообщений размером 1KB со скоростью 100 msg/sec

```bash
kafka-producer-perf-test --topic test-topic --num-records 1000 --record-size 1024 --throughput 100 --producer-props bootstrap.servers=localhost:9092
```

### 10. Тест производительности Consumer

**Описание**: Читаем 1000 сообщений и измеряем производительность

```bash
kafka-consumer-perf-test --topic test-topic --bootstrap-server localhost:9092 --messages 1000 --threads 1
```

### 11. Проверка версий API брокера

**Описание**: Проверяем здоровье и доступные API брокера

```bash
kafka-broker-api-versions --bootstrap-server localhost:9092
```

### 12. Информация о директориях логов

**Описание**: Показывает использование дискового пространства

```bash
kafka-log-dirs --bootstrap-server localhost:9092 --describe
```

### 13. Удаление топика

**Описание**: Удаляем тестовый топик

```bash
kafka-topics --delete --topic test-topic --bootstrap-server localhost:9092
```

---

## Чек-лист тестирования Kafka

### Базовая функциональность

- [ ] Kafka контейнер запущен и отвечает
- [ ] Можно создать топик
- [ ] Топик отображается в списке
- [ ] Детали топика корректные (партиции, реплики)
- [ ] Producer может отправить сообщения
- [ ] Consumer может прочитать сообщения
- [ ] Сообщения доставляются в правильном порядке

### Consumer Groups

- [ ] Можно создать consumer в группе
- [ ] Consumer group отображается в списке
- [ ] Смещения (offsets) корректно отслеживаются
- [ ] Лаг (lag) отображается правильно
- [ ] При перезапуске consumer продолжает с последнего offset

### Производительность

- [ ] Producer test показывает разумный throughput
- [ ] Consumer test показывает разумную скорость
- [ ] Нет критических ошибок в логах
- [ ] Использование памяти в норме

### Веб-интерфейс (Kafdrop)

- [ ] Kafdrop доступен на <http://localhost:9001>
- [ ] Топики отображаются в интерфейсе
- [ ] Можно просмотреть сообщения через веб
- [ ] Consumer groups видны в интерфейсе
- [ ] Статистика обновляется в реальном времени

### Управление

- [ ] Можно удалить топик
- [ ] API версии брокера доступны
- [ ] Информация о логах доступна
- [ ] Конфигурация кластера корректная

### Отказоустойчивость

- [ ] При остановке consumer в группе его партиции перераспределяются
- [ ] Сообщения не теряются при перезапуске
- [ ] Healthcheck контейнеров проходит успешно
