### TsFile-Spark Connector

This library lets you expose a TsFile as Spark RDDs and execute arbitrary queries in your SparkSQL.

#### Requirements

The versions required for Spark and Java are as follow:

| Spark Version | Scala Version | Java Version |
| ------------- | ------------- | ------------ |
| `2.0+`        | `2.11`        | `1.8`        |

#### Building From Source

To build the TsFile-Spark connector, run the following commands at the root folder of TsFile:
```
mvn install -Dmaven.test.skip=true

cd integration-parent/tsfile-spark
mvn package -Dmaven.test.skip=true
```
The `tsfile-spark-0.1.0-jar-with-dependencies.jar` can be get in folder `target`.

#### Data Type Reflection from TsFile to SparkSQL

This library uses the following mapping the data type from TsFile to SparkSQL:

| TsFile 		   | SparkSQL|
| --------------| -------------- |
| BOOLEAN     		   | BooleanType    |
| INT32       		   | IntegerType    |
| INT64       		   | LongType       |
| FLOAT       		   | FloatType      |
| DOUBLE      		   | DoubleType     |
| ENUMS                | StringType     |
| BYTE_ARRAY           | BinaryType     |


#### TsFile Schema -> SparkSQL Table Structure

The set of time-series data in section "Time-series Data" is used here to illustrate the mapping from TsFile Schema to SparkSQL Table Stucture.

<center>
<table style="text-align:center">
	<tr><th colspan="6">device_1</th></tr>
	<tr><th colspan="2">sensor_1</th><th colspan="2">sensor_2</th><th colspan="2">sensor_3</th></tr>
	<tr><th>time</th><th>value</td><th>time</th><th>value</td><th>time</th><th>value</td>
	<tr><td>1</td><td>1.2</td><td>1</td><td>20</td><td>2</td><td>50</td></tr>
	<tr><td>3</td><td>1.4</td><td>2</td><td>20</td><td>4</td><td>51</td></tr>
	<tr><td>5</td><td>1.1</td><td>3</td><td>21</td><td>6</td><td>52</td></tr>
	<tr><td>7</td><td>1.8</td><td>4</td><td>20</td><td>8</td><td>53</td></tr>
</table>
<span>A set of time-series data</span>
</center>

There are two reserved columns in Spark SQL Table:

- `time` : Timestamp, LongType
- `delta_object` : Delta_object ID, StringType

The SparkSQL Table Structure is as follow:

<center>
	<table style="text-align:center">
	<tr><th>time(LongType)</th><th>delta_object(StringType)</th><th>sensor_1(FloatType)</th><th>sensor_2(IntType)</th><th>sensor_3(IntType)</th></tr>
	<tr><td>1</td><td>device_1</td><td>1.2</td><td>20</td><td>null</td></tr>
	<tr><td>2</td><td>device_1</td><td>null</td><td>20</td><td>50</td></tr>
	<tr><td>3</td><td>device_1</td><td>1.4</td><td>21</td><td>null</td></tr>
	<tr><td>4</td><td>device_1</td><td>null</td><td>20</td><td>51</td></tr>
	<tr><td>5</td><td>device_1</td><td>1.1</td><td>null</td><td>null</td></tr>
	<tr><td>6</td><td>device_1</td><td>null</td><td>null</td><td>52</td></tr>
	<tr><td>7</td><td>device_1</td><td>1.8</td><td>null</td><td>null</td></tr>
	<tr><td>8</td><td>device_1</td><td>null</td><td>null</td><td>53</td></tr>
	</table>

</center>

#### Examples

##### Scala API

* **Example 1**

	```scala
	// import this library and Spark
	import com.corp.delta.tsfile.spark._
	import org.apache.spark.sql.SparkSession

	val spark = SparkSession.builder().master("local").getOrCreate()

	//read data in TsFile and create a table
	val df = spark.read.tsfile("test.ts")
	df.createOrReplaceTempView("TsFile_table")

	//query with filter
	val newDf = spark.sql("select * from TsFile_table where sensor_1 > 1.2").cache()

	newDf.show()

	```

* **Example 2**

	```scala
	val spark = SparkSession.builder().master("local").getOrCreate()
	val df = spark.read
	      .format("com.corp.delta.tsfile")
	      .load("test.ts")


	df.filter("sensor_1 > 1.2").show()

	```

* **Example 3**

	```scala
	val spark = SparkSession.builder().master("local").getOrCreate()

	//create a table in SparkSQL and build relation with a TsFile
	spark.sql("create temporary view TsFile using com.corp.delta.tsfile options(path = \"test.ts\")")

	spark.sql("select * from TsFile where sensor_1 > 1.2").show()

	```

##### spark-shell

This library can be used in `spark-shell`.

```
$ bin/spark-shell --jars tsfile-spark-0.1.0-jar-with-dependencies.jar

scala> sql("CREATE TEMPORARY TABLE TsFile_table USING com.corp.delta.tsfile OPTIONS (path \"hdfs://localhost:9000/test.ts\")")

scala> sql("select * from TsFile_table where sensor_1 > 1.2").show()
```