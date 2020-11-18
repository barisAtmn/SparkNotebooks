## Delta Lake Spark Notebook Commands

import org.apache.spark.sql._
import org.apache.spark.sql.functions._

val events = spark.read
  .option("inferSchema", "true")
  .json("/databricks-datasets/structured-streaming/events/")
  .withColumn("date", expr("time"))
  .drop("time")
  .withColumn("date", from_unixtime($"date", "yyyy-MM-dd"))

---------------------------------------------

display(events)

---------------------------------------------

events.write.format("delta").mode("overwrite").partitionBy("date").save("/delta/events/")

---------------------------------------------

val events_delta = spark.read.format("delta").load("/delta/events/")

---------------------------------------------

display(spark.sql("DROP TABLE IF EXISTS events"))

display(spark.sql("CREATE TABLE events USING DELTA LOCATION '/delta/events/'"))

---------------------------------------------

events_delta.count()

---------------------------------------------
Select * from events where date='2016-07-27'

describe detail events

---------------------------------------------

val historical_events = spark.read
  .option("inferSchema", "true")
  .json("/databricks-datasets/structured-streaming/events/")
  .withColumn("date", expr("time-172800"))
  .drop("time")
  .withColumn("date", from_unixtime($"date", "yyyy-MM-dd"))

---------------------------------------------

historical_events.write.format("delta").mode("append").partitionBy("date").save("/delta/events/")

---------------------------------------------
Select count(*) from events

DESCRIBE HISTORY events

---------------------------------------------

spark.read.
 option("versionAsOf", 0).
 format("delta").
 load("/delta/events/").
 count()
