---
type: "practice"
difficulty: "Basic"
---

# What are the different file formats you can use in Hadoop such as text CSV Avro ORC and Parquet?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ HDFS Fundamentals](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/HDFS_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Hadoop, data is stored using various file formats to suit specific use cases, performance needs, and storage requirements. The common file formats you might encounter include Text, CSV, Avro, ORC, and Parquet. These formats each have their own strengths and are chosen based on the project's needs.

Firstly, Text files are simple line-separated text records, making them easy to read and write using tools like cat or grep. However, they lack structured metadata and do not support efficient querying or indexing.

Secondly, CSV (Comma-Separated Values) files are a variation of Text files with a defined format where values are separated by commas. While CSV is widely used due to its simplicity, it also lacks schema enforcement and can lead to data inconsistencies if the delimiter is not respected.

Avro, on the other hand, is a columnar storage file format that includes metadata about the data stored in each record. This metadata allows for better serialization and deserialization of complex records, supporting complex datatypes like maps, lists, and structs. Avro files are self-describing, meaning they include schema information within the file itself.

ORC (Optimized Row Columnar) is another columnar storage format that aims to improve query performance by storing data in a way that optimizes access patterns. ORC supports advanced features such as compression, encryption, and predicate pushdown, making it highly efficient for large datasets.

Finally, Parquet files are also columnar but are designed with a focus on analytical workloads. They support complex nested datatypes and can be read by various big data tools like Spark and Impala. Parquet is particularly useful when dealing with large-scale analytics due to its ability to store metadata alongside the data.

Each of these formats has trade-offs in terms of storage efficiency, query performance, and ease of use. For instance, while Text and CSV are simple and widely supported, Avro and ORC/Parquet offer more advanced features at the cost of complexity.

**Code:**

```bash
# Example of creating a file with different formats

# Create a text file
echo "Name,Age,Country" > employee.txt
echo "John,25,USA" >> employee.txt
echo "Alice,30,UK" >> employee.txt

# Create a CSV file
echo "Name,Age,Country" > employees.csv
echo "John,25,USA" >> employees.csv
echo "Alice,30,UK" >> employees.csv

# Avro schema and record (simplified)
{
  "namespace": "example.avro",
  "type": "record",
  "name": "Employee",
  "fields": [
    {"name": "Name", "type": "string"},
    {"name": "Age", "type": "int"},
    {"name": "Country", "type": "string"}
  ]
}
```

**Example input / output:**

```text
# Example of loading and reading data in Hadoop

# Loading CSV into HDFS
hadoop fs -put employees.csv /user/hadoop/

# Reading CSV using Hive
CREATE TABLE employee (name STRING, age INT, country STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE;
LOAD DATA INPATH '/user/hadoop/employees.csv' INTO TABLE employee;

# Loading Avro into HDFS
hadoop fs -put employees.avro /user/hadoop/

# Reading Avro using Hive
CREATE EXTERNAL TABLE employee (name STRING, age INT, country STRING) STORED AS AVRO LOCATION '/user/hadoop/';

# Reading ORC into HDFS
hadoop fs -put employees.orc /user/hadoop/

# Reading ORC using Hive
CREATE EXTERNAL TABLE employee (name STRING, age INT, country STRING) STORED AS ORC LOCATION '/user/hadoop/';

# Reading Parquet into HDFS
hadoop fs -put employees.parquet /user/hadoop/

# Reading Parquet using Hive
CREATE EXTERNAL TABLE employee (name STRING, age INT, country STRING) STORED AS PARQUET LOCATION '/user/hadoop/';
```

**Data:**

```json
{
  "employees_avro": {
    "Name": "John",
    "Age": 25,
    "Country": "USA"
  }
}
Record	Data Type
Name	String
Age	Integer
Country	String
JSON
Copy
{
  "employees_orc": {
    "Name": "John",
    "Age": 25,
    "Country": "USA"
  }
}
Record	Data Type
Name	String
Age	Integer
Country	String
JSON
Copy
{
  "employees_parquet": {
    "Name": "John",
    "Age": 25,
    "Country": "USA"
  }
}
Record	Data Type
Name	String
Age	Integer
Country	String
```

