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
├── notebooks/           # Jupyter ноутбуки
│   ├── nyc_taxi_analysis.ipynb
│   └── spark_and_hdfs_tutorial.ipynb
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
- **Jupyter Notebook**: [http://localhost:8888](http://localhost:8888) (пароль/токен см. в логах `docker-compose logs jupyter`)

## 🧑‍💻 Работа с проектом

### 1. Jupyter Notebook (PySpark)
Самый удобный способ работы с данными.

**Быстрый старт с обучением:**
1. Откройте [http://localhost:8888](http://localhost:8888).
2. Перейдите в папку `work`.
3. Откройте файл **`spark_and_hdfs_tutorial.ipynb`**.
4. Это интерактивный урок: читайте описание и запускайте код ячейка за ячейкой (Shift+Enter).

**Проверка работы вручную:**
1. Создайте новый ноутбук.
2. Вставьте код:

```python
from pyspark.sql import SparkSession

# Создаем SparkSession с подключением к кластеру
spark = SparkSession.builder \
    .appName("JupyterTest") \
    .master("spark://spark-master:7077") \
    .getOrCreate()

# Проверяем работу
data = [("Done", 1), ("Success", 2)]
df = spark.createDataFrame(data, ["Status", "ID"])
df.show()

print(f"Spark Version: {spark.version}")
spark.stop()
```

### 2. Работа с HDFS (CLI)
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

## 🔧 Решение проблем (Troubleshooting)

### 1. Jupyter: `hdfs: command not found`
Если в ноутбуке не работают команды `!hdfs`, убедитесь, что выполнена ячейка настройки окружения в начале ноутбука.
Она добавляет пути к бинарникам Hadoop (которые скачиваются в `notebooks/hadoop`).

### 2. Spark SQL: `TABLE_OR_VIEW_NOT_FOUND`
Если запрос `SELECT ... FROM table` падает с ошибкой, значит вы забыли зарегистрировать DataFrame как таблицу.
Выполните перед SQL запросом:
```python
df.createOrReplaceTempView("table_name")
```

### 3. HDFS: `Permission denied`
Если вы не можете записать файлы в HDFS из Jupyter (ошибка `AccessControlException`), значит у пользователя `jovyan` нет прав на запись в папку.
Решение (разрешить запись всем):
```bash
docker exec namenode hdfs dfs -chmod 777 /user/myself
```
