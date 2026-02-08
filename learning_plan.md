# 🎓 План изучения Big Data (Hadoop & Spark)

Этот план обучения был выполнен вами для освоения основ работы с кластером Hadoop в Docker. Он включает как теорию, так и практические лабораторные работы (Labs), которые вы уже завершили или можете повторить.

## ✅ Модуль 1: Основы HDFS (Hadoop Distributed File System)
**Цель:** Научиться управлять файлами в распределенной системе.

### 📚 Теория
- Архитектура NameNode (Master) и DataNode (Worker).
- Блоки, репликация, отказоустойчивость.

### 🛠 Практика (Lab 1)
- **Создание директории:** `hdfs dfs -mkdir -p /user/myself`
- **Загрузка файла:** `echo "Hello" > /tmp/test.txt && hdfs dfs -put /tmp/test.txt /user/myself/`
- **Просмотр:** `hdfs dfs -ls /user/myself/`
- **Чтение:** `hdfs dfs -cat /user/myself/test.txt`

---

## ✅ Модуль 2: MapReduce и YARN
**Цель:** Понять классическую модель обработки данных и управление ресурсами.

### 📚 Теория
- Map (Разбиение) -> Shuffle (Сортировка) -> Reduce (Агрегация).
- YARN: ResourceManager (Планировщик) и NodeManager (Исполнитель).

### 🛠 Практика (Lab 2: WordCount)
Запуск классического примера на Java:
```bash
# 1. Подготовка данных
docker exec namenode bash -c "echo 'Hello Hadoop Hello World' > /tmp/input.txt"
docker exec namenode hdfs dfs -put /tmp/input.txt /user/input/

# 2. Запуск MapReduce Job
docker exec resourcemanager yarn jar \
  /opt/hadoop-3.2.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar \
  wordcount /user/input /user/output

# 3. Проверка результата
docker exec namenode hdfs dfs -cat /user/output/part-r-00000
```

---

## ✅ Модуль 3: Spark Core и RDD
**Цель:** Основы быстрых распределенных вычислений в памяти.

### 📚 Теория
- RDD (Resilient Distributed Dataset) — неизменяемые коллекции.
- Transformations (Lazy) vs Actions (Eager).
- Spark Shell (REPL).

### 🛠 Практика (Lab 3: RDD API)
Внутри `spark-shell`:
```scala
// Чтение файла
val lines = sc.textFile("hdfs://namenode:9000/user/input/input.txt")
// Подсчет слов (MapReduce в одну строку)
val counts = lines.flatMap(_.split(" ")).map((_, 1)).reduceByKey(_ + _)
// Вывод
counts.collect().foreach(println)
```

---

## ✅ Модуль 4: Spark SQL и DataFrames
**Цель:** Работа со структурированными данными (как SQL/Pandas).

### 📚 Теория
- DataFrame API и схема данных.
- Оптимизатор Catalyst.
- Формат Parquet (колоночное хранение).

### 🛠 Практика (Lab 4: NYC Taxi Analysis)
Работа с реальными данными (Yellow Taxi Trip Data):
```scala
// 1. Чтение Parquet
val df = spark.read.parquet("hdfs://namenode:9000/user/myself/yellow_tripdata_2023-01.parquet")

// 2. Изучение схемы
df.printSchema()

// 3. SQL Аналитика (Средний чек по кол-ву пассажиров)
df.createOrReplaceTempView("trips")
spark.sql("""
  SELECT passenger_count, AVG(total_amount) as avg_fare, COUNT(*) as trips
  FROM trips
  GROUP BY passenger_count
  ORDER BY avg_fare DESC
""").show()
```

---

## 🚀 Следующие шаги (Продвинутый уровень)

### 🔜 Модуль 5: Spark Streaming
Обработка данных в реальном времени.
- Чтение потока из Kafka (можно добавить Kafka контейнер).
- Structured Streaming API.

### 🔜 Модуль 6: Оптимизация и Тюнинг
- Partitioning и Bucketing.
- Кэширование (`.cache()` / `.persist()`).
- Понимание Spark UI (DAG Visualization).

### 🔜 Модуль 7: Подключение BI (Business Intelligence)
- Запуск Spark Thrift Server.
- Подключение Tableau, PowerBI или DBeaver по JDBC.
