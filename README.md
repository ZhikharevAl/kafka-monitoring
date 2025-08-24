# Мониторинг Kafka

Комплексное решение для мониторинга Apache Kafka с Prometheus, Grafana и Kafdrop для разработки и тестирования.

## Возможности

- **Apache Kafka** с режимом KRaft (без Zookeeper)
- **Kafdrop** - веб-интерфейс для управления кластером Kafka
- **Prometheus** - сбор и хранение метрик
- **Grafana** - визуализация данных и дашборды
- **Kafka Exporter** - экспорт метрик Kafka для Prometheus
- **Автоматизированное тестирование** - комплексные тестовые сценарии
- **CI/CD Pipeline** - GitHub Actions для контроля качества кода

## Требования

- Docker и Docker Compose
- Python 3.13+ (для разработки)
- UV менеджер пакетов (для зависимостей Python)

## Быстрый старт

### 1. Запуск инфраструктуры

```bash
# Клонируем репозиторий
git clone https://github.com/ZhikharevAl/kafka-monitoring.git
cd kafka-monitoring

# Запускаем все сервисы
docker-compose up -d

# Проверяем состояние сервисов
docker-compose ps
```

### 2. Доступ к веб-интерфейсам

| Сервис | URL | Логин/Пароль |
|--------|-----|--------------|
| Kafdrop (Kafka UI) | <http://localhost:9001> | - |
| Grafana | <http://localhost:3000> | admin/admin |
| Prometheus | <http://localhost:9090> | - |

### 3. Проверка установки

```bash
# Проверяем, что Kafka отвечает
kafka-topics --list --bootstrap-server localhost:9092

# Создаем тестовый топик
kafka-topics --create --topic test --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
```

## Архитектура


## 📊 Мониторинг

### Доступные метрики

- **Лаг потребителей**: отслеживание задержек обработки сообщений
- **Пропускная способность**: скорость производителя/потребителя
- **Распределение по партициям**: балансировка нагрузки
- **Состояние групп потребителей**: активные участники и ребалансировка
- **Состояние брокера**: версии API, директории логов, время работы

### Дашборды Grafana

В настройке включены предварительно настроенные дашборды для:

- Обзор Kafka
- Группы потребителей
- Детали топиков
- Метрики брокера

## Тестирование

### Базовые тесты Kafka

Выполните комплексный чек-лист тестирования:

```bash
# Следуйте тестовым сценариям в docs/check-list.md
kafka-topics --create --topic test-topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1

# Тест производителя
kafka-console-producer --topic test-topic --bootstrap-server localhost:9092

# Тест потребителя
kafka-console-consumer --topic test-topic --bootstrap-server localhost:9092 --from-beginning
```

### Продвинутые сценарии тестирования

Подробное руководство по тестированию доступно в [`docs/kafka-testing-guide.md`](docs/kafka-testing-guide.md):

1. **Нагрузочное тестирование** - бенчмарки производительности
2. **Масштабирование потребителей** - множественные экземпляры потребителей
3. **Восстановление после сбоев** - тестирование устойчивости
4. **Пиковая нагрузка** - обработка всплесков
5. **Мульти-топики** - симуляция микросервисов
6. **Управление смещениями** - контроль позиции потребителя

### Тестирование производительности

```bash
# Тест производительности производителя
kafka-producer-perf-test --topic test-topic --num-records 10000 --record-size 1024 --throughput 1000 --producer-props bootstrap.servers=localhost:9092

# Тест производительности потребителя
kafka-consumer-perf-test --topic test-topic --bootstrap-server localhost:9092 --messages 10000 --threads 1
```

## Разработка

### Настройка среды разработки

```bash
# Установка UV (если еще не установлен)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Установка зависимостей
uv sync --dev

# Настройка pre-commit хуков
uv run pre-commit install
```

### Контроль качества кода

В проекте используются автоматизированные проверки качества кода:

- **Ruff** - линтинг и форматирование Python
- **Pyright** - проверка типов
- **Pre-commit hooks** - автоматические проверки перед коммитами

```bash
# Запуск линтинга
uvx ruff check .

# Запуск форматирования
uvx ruff format .

# Проверка типов
uv run pyright .
```

## CI/CD

GitHub Actions workflow (`.github/workflows/code-quality.yaml`) автоматически:

1. Проверяет консистентность файла `uv.lock`
2. Запускает линтинг кода с помощью Ruff
3. Проверяет форматирование кода
4. Выполняет проверку типов с помощью Pyright

## 📁 Структура проекта

```text
.
├── .github/
│   ├── actions/          # Переиспользуемые GitHub Actions
│   └── workflows/        # CI/CD workflow'ы
├── docs/
│   ├── check-list.md     # Базовый чек-лист тестирования Kafka
│   └── kafka-testing-guide.md  # Продвинутые сценарии тестирования
├── prometheus/
│   └── prometheus.yml    # Конфигурация Prometheus
├── docker-compose.yaml   # Основная настройка инфраструктуры
├── pyproject.toml       # Конфигурация Python проекта
└── .pre-commit-config.yaml  # Pre-commit хуки
```

## 📚 Документация

- [`docs/check-list.md`](docs/check-list.md) - Базовые тесты функциональности Kafka
- [`docs/kafka-testing-guide.md`](docs/kafka-testing-guide.md) - Продвинутые сценарии тестирования с мониторингом

## Устранение неполадок

### Частые проблемы

**Kafka не запускается:**

```bash
# Проверяем логи
docker-compose logs kafka

# Перезапускаем сервисы
docker-compose restart kafka
```

**Конфликты портов:**

```bash
# Проверяем, что использует порты
lsof -i :9092
lsof -i :3000
lsof -i :9001
```

**Лаг потребителей не уменьшается:**

- Проверьте, что потребитель работает: `kafka-consumer-groups --list --bootstrap-server localhost:9092`
- Проверьте состояние группы потребителей: `kafka-consumer-groups --describe --group <group-name> --bootstrap-server localhost:9092`


## Благодарности

- [Apache Kafka](https://kafka.apache.org/) - платформа распределенного стриминга
- [Confluent](https://www.confluent.io/) - Docker образы Kafka
- [Prometheus](https://prometheus.io/) - система мониторинга
- [Grafana](https://grafana.com/) - платформа визуализации
- [Kafdrop](https://github.com/obsidiandynamics/kafdrop) - веб-интерфейс для Kafka
