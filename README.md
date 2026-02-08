# Hadoop & Spark Docker Sandbox 🐘⚡️

Проект для изучения и экспериментов с экосистемой Hadoop (HDFS, YARN, MapReduce) и Apache Spark в Docker-контейнерах.

## 💻 Что в проекте
В данном проекте реализована полноценная среда Big Data. Основные компоненты:
- ✔️ **Docker Compose** - для оркестрации всех сервисов.
- ✔️ **Hadoop HDFS** - распределенная файловая система (NameNode + DataNode).
- ✔️ **Hadoop YARN** - менеджер ресурсов (ResourceManager + NodeManager).
- ✔️ **Hadoop MapReduce** - классический фреймворк вычислений (HistoryServer).
- ✔️ **Apache Spark** - современный движок для быстрой обработки данных (Master + Worker).
- ✔️ **Jupyter / Shell** - возможность запускать задачи интерактивно.

## 📁 Структура проекта
```
Hadoop/
├── docker-compose.yml   # Конфигурация всех сервисов кластера
├── learning_plan.md     # Пошаговый план обучения (HDFS -> MapReduce -> Spark)
└── README.md            # Документация проекта (этот файл)
```

## 🚀 Установка и Запуск

### Предварительные требования
- Docker Desktop (с поддержкой Linux контейнеров)
- 4GB+ RAM выделено под Docker

### Запуск проекта
Перейдите в папку проекта и выполните:
```bash
docker-compose up -d
```

Проверьте статус контейнеров:
```bash
docker-compose ps
```

### Доступ к интерфейсам
После запуска доступны следующие Web UI:
- **HDFS NameNode**: [http://localhost:9870](http://localhost:9870)
- **YARN ResourceManager**: [http://localhost:8088](http://localhost:8088)
- **Spark Master**: [http://localhost:8090](http://localhost:8090)
- **MapReduce History Server**: [http://localhost:8188](http://localhost:8188)

## 🧑‍💻 Работа с проектом

### 1. Работа с HDFS (CLI)
Создание папки и загрузка файла:
```bash
# Создать директорию пользователя
docker exec namenode hdfs dfs -mkdir -p /user/myself

# Создать локальный файл в контейнере и загрузить его в HDFS
docker exec namenode bash -c "echo 'Hello Big Data' > /tmp/test.txt"
docker exec namenode hdfs dfs -put /tmp/test.txt /user/myself/

# Проверить файл
docker exec namenode hdfs dfs -ls /user/myself/
```

### 2. Запуск MapReduce (WordCount)
Классический пример подсчета слов на Hadoop:
```bash
# Подготовим данные
docker exec namenode bash -c "echo 'Hadoop Spark Hadoop YARN' > /tmp/input.txt"
docker exec namenode hdfs dfs -mkdir -p /user/wordcount/input
docker exec namenode hdfs dfs -put /tmp/input.txt /user/wordcount/input/

# Запустим задачу (используя встроенный JAR)
docker exec resourcemanager yarn jar \
  /opt/hadoop-3.2.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar \
  wordcount /user/wordcount/input /user/wordcount/output

# Посмотрим результат
docker exec namenode hdfs dfs -cat /user/wordcount/output/part-r-00000
```

### 3. Работа со Spark (Scala Shell)
Интерактивная консоль для Data Science задач:
```bash
docker exec -it spark-master /opt/spark/bin/spark-shell --master spark://spark-master:7077
```

Внутри консоли (расчет числа Пи):
```scala
val count = sc.parallelize(1 to 100000).filter { _ =>
  val x = math.random
  val y = math.random
  x*x + y*y < 1
}.count()
println(s"Pi is roughly ${4.0 * count / 100000}")
```

### 4. Работа с данными (Пример NYC Taxi)
Загрузка реального датасета (Parquet) и анализ через Spark SQL:

**Загрузка данных:**
```bash
# Скачиваем файл в контейнер
docker exec namenode curl -o /tmp/taxi.parquet https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2023-01.parquet

# Кладем в HDFS
docker exec namenode hdfs dfs -put /tmp/taxi.parquet /user/myself/
```

**Анализ (в Spark Shell):**
```scala
val df = spark.read.parquet("hdfs://namenode:9000/user/myself/taxi.parquet")
df.createOrReplaceTempView("taxi")
spark.sql("SELECT passenger_count, count(*) FROM taxi GROUP BY 1").show()
```

## 📖 План обучения
В файле `learning_plan.md` находится детальный роадмап для изучения технологий в этом окружении.
