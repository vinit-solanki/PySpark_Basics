# 🚀 **PySpark End-to-End Interview Cheatsheet**

---

## 🧠 1. What is PySpark?

PySpark = Python API for **Apache Spark** — a distributed computing engine used for big data processing.
It allows **parallel processing** on large datasets across clusters.

🔹 Built on top of **JVM**
🔹 Uses **RDD**, **DataFrame**, **Dataset**, and **SQL** APIs
🔹 Great for: ETL, MLlib, Streaming, and Graph Processing

---

## ⚙️ 2. Spark Architecture (must-know diagram conceptually)

```
Driver Program
 ├── SparkContext → coordinates everything
 │     ├── Cluster Manager (YARN / Standalone / Mesos)
 │     └── Executors (on Worker Nodes)
 │           └── Tasks
```

📌 **Driver** → coordinates
📌 **Executor** → runs the tasks
📌 **Cluster Manager** → allocates resources

---

## 🧱 3. Core Concepts

| Concept                                 | Description                                  | Example                    |
| --------------------------------------- | -------------------------------------------- | -------------------------- |
| **RDD** (Resilient Distributed Dataset) | Low-level distributed data collection        | `sc.parallelize([1,2,3])`  |
| **Transformation**                      | Lazy operation returning new RDD             | `map`, `filter`, `flatMap` |
| **Action**                              | Triggers computation                         | `count()`, `collect()`     |
| **DataFrame**                           | Structured RDD with schema                   | `spark.read.csv()`         |
| **Dataset**                             | Typed version of DataFrame (Scala/Java only) | —                          |

---

## 🧮 4. Creating a SparkSession

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("Demo").master("local[*]").getOrCreate()
```

---

## 🧾 5. Reading & Writing Data

### ✅ Read Data

```python
df = spark.read.csv("data.csv", header=True, inferSchema=True)
df = spark.read.json("data.json")
```

### ✅ Write Data

```python
df.write.csv("output.csv", header=True)
df.write.parquet("output.parquet")
```

---

## 📊 6. DataFrame Basics

```python
df.show(5)           # Display top rows
df.printSchema()     # Schema
df.columns           # Columns
df.describe().show() # Summary stats
```

---

## 🧩 7. DataFrame Transformations

| Operation          | Example                                 | Description       |
| ------------------ | --------------------------------------- | ----------------- |
| `select`           | `df.select("col1", "col2")`             | Choose columns    |
| `filter` / `where` | `df.filter(col("age") > 30)`            | Row filter        |
| `withColumn`       | `df.withColumn("new", col("a")+1)`      | Add column        |
| `drop`             | `df.drop("col1")`                       | Drop column       |
| `groupBy`          | `df.groupBy("city").agg(avg("salary"))` | Aggregation       |
| `orderBy`          | `df.orderBy(col("age").desc())`         | Sorting           |
| `distinct`         | `df.distinct()`                         | Remove duplicates |

---

## 🧠 8. SQL with Spark

### Register & Query

```python
df.createOrReplaceTempView("people")
spark.sql("SELECT name, age FROM people WHERE age > 30").show()
```

---

## 🧹 9. Handling Missing Data

```python
df.na.drop()                     # Drop nulls
df.na.fill({"age": 0})           # Fill specific columns
df.na.replace("?", None)         # Replace values
```

### Using Imputer

```python
from pyspark.ml.feature import Imputer
imputer = Imputer(
    inputCols=['age'], outputCols=['age_imputed']
).setStrategy("mean")
df = imputer.fit(df).transform(df)
```

---

## 🧮 10. Feature Engineering (MLlib)

### StringIndexer (categorical → numeric)

```python
from pyspark.ml.feature import StringIndexer
indexer = StringIndexer(inputCol="gender", outputCol="gender_indexed")
df = indexer.fit(df).transform(df)
```

### OneHotEncoder

```python
from pyspark.ml.feature import OneHotEncoder
encoder = OneHotEncoder(inputCols=['gender_indexed'], outputCols=['gender_vec'])
df = encoder.fit(df).transform(df)
```

### VectorAssembler (combine features)

```python
from pyspark.ml.feature import VectorAssembler
assembler = VectorAssembler(
    inputCols=['age','salary','gender_vec'],
    outputCol='features'
)
df = assembler.transform(df)
```

---

## 🤖 11. Machine Learning Pipeline

### Example: Linear Regression

```python
from pyspark.ml.regression import LinearRegression

train, test = df.randomSplit([0.8, 0.2], seed=42)
lr = LinearRegression(featuresCol='features', labelCol='salary')
model = lr.fit(train)

predictions = model.transform(test)
predictions.select("salary","prediction").show()
```

### Evaluate

```python
from pyspark.ml.evaluation import RegressionEvaluator
evaluator = RegressionEvaluator(labelCol="salary", predictionCol="prediction", metricName="rmse")
rmse = evaluator.evaluate(predictions)
print("RMSE:", rmse)
```

---

## 📦 12. Pipelines (best practice)

```python
from pyspark.ml import Pipeline

stages = [indexer, encoder, assembler, lr]
pipeline = Pipeline(stages=stages)
model = pipeline.fit(train)
pred = model.transform(test)
```

---

## 📈 13. Common Transformations (interview quickfire)

| Transformation  | Description         |
| --------------- | ------------------- |
| `map()`         | Apply a function    |
| `filter()`      | Keep matching rows  |
| `flatMap()`     | Expand each element |
| `reduceByKey()` | Aggregate by key    |
| `join()`        | Merge datasets      |
| `union()`       | Combine DataFrames  |
| `crossJoin()`   | Cartesian product   |

---

## ⚡ 14. Performance Optimization Tips

✅ Cache data if reused

```python
df.cache()
```

✅ Use **Parquet** (columnar, compressed) instead of CSV

✅ Avoid using `collect()` on large data
✅ Use `repartition()` for load balancing
✅ Prefer **DataFrames** over RDDs

---

## 🧩 15. Window Functions

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import rank, col

win = Window.partitionBy("dept").orderBy(col("salary").desc())
df.withColumn("rank", rank().over(win)).show()
```

---

## 🧰 16. Useful Functions

```python
from pyspark.sql.functions import col, when, countDistinct, avg, sum, lit

df.withColumn("is_adult", when(col("age") >= 18, 1).otherwise(0))
df.groupBy("dept").agg(avg("salary"), countDistinct("name"))
df.withColumn("constant", lit("Fixed"))
```

---

## 🧱 17. Writing to Databases

```python
df.write \
    .format("jdbc") \
    .option("url", "jdbc:mysql://localhost:3306/db") \
    .option("dbtable", "table_name") \
    .option("user", "root") \
    .option("password", "1234") \
    .save()
```

---

## 🧠 18. Common Interview Questions

| Question                                           | Key Idea                                                    |
| -------------------------------------------------- | ----------------------------------------------------------- |
| What is lazy evaluation?                           | Spark builds DAG, executes only on action                   |
| Difference between narrow vs wide transformations? | Narrow: one-to-one (map), Wide: shuffle (groupBy)           |
| Why use DataFrame over RDD?                        | Optimized by Catalyst, less code                            |
| What is Catalyst Optimizer?                        | Spark SQL query planner                                     |
| What is Tungsten?                                  | Spark’s physical execution engine for memory mgmt.          |
| What are stages and tasks?                         | Stages = units of parallel work, Tasks = per partition work |
| How to persist data?                               | `.cache()`, `.persist()`                                    |
| What causes shuffle?                               | GroupBy, Join, Repartition                                  |
| What’s a broadcast join?                           | Small dataset copied to all executors                       |

---

## 🧾 19. Streaming (bonus)

```python
stream_df = spark.readStream.format("socket").option("host","localhost").option("port",9999).load()
query = stream_df.writeStream.outputMode("append").format("console").start()
query.awaitTermination()
```

---

## 🚀 20. Quick Debug Checklist

✅ Check column names → `df.columns`
✅ Print schema → `df.printSchema()`
✅ Check for extra spaces → `df.toDF(*[c.strip() for c in df.columns])`
✅ Handle missing values → `.na.fill()`
✅ Avoid `FileNotFoundError` → use correct path (e.g., `"/content/data.csv"` in Colab)

---

# 🎯 Summary Mind Map

```
PySpark
 ├── Core: RDD / DF / SQL
 ├── MLlib: StringIndexer → OneHotEncoder → VectorAssembler → Model
 ├── Optimizers: Catalyst + Tungsten
 ├── Deployment: Cluster / Local
 ├── Best Practices: cache, parquet, avoid collect()
 └── Common Errors: missing columns, spaces, data type mismatch
```
