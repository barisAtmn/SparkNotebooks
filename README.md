DELTA LAKE

*  "CREATE TABLE {table name} USING DELTA LOCATION '{path}'"
* describe detail {table name}
* describe history {table name}
* DF.write.format("delta").mode("append").partitionBy("date").save("{path}")
* VACUUM doesn’t clean up log files; log files are automatically cleaned up after checkpoints are written.
* By default, Delta tables retain the commit history for 30 days. This means that you can specify a version from 30 days ago.
   * You did not run VACUUM on your Delta table. If you run VACUUM, you lose the ability to go back to a version older than the default 7 day data retention period.
   * delta.logRetentionDuration = "interval <interval>": controls how long the history for a table is kept. Each time a a checkpoint is written, Databricks automatically cleans up log entries older than the retention interval. If you set this config to a large enough value, many log entries are retained. This          should not impact performance as operations against the log are constant time. Operations on history are parallel (but will become more expensive as the log size increases). The default is interval 30 days.
   * delta.deletedFileRetentionDuration = "interval <interval>": controls how long ago a file must have been deleted before being a candidate for VACUUM. The default is interval 7 days. For access to 30 days of historical data, set delta.deletedFileRetentionDuration = "interval 30 days". This setting may cause your      storage costs to go up.

* TIMESTAMP AS OF {} // VERSION AS OF 0
* .option("mergeSchema", true) // Schema evolution
   * ALTER TABLE table_name ADD COLUMNS (col_name.nested_col_name data_type [COMMENT col_comment] [FIRST|AFTER colA_name], ...)
   * ALTER TABLE table_name CHANGE [COLUMN] col_name col_name data_type [COMMENT col_comment] [FIRST|AFTER colA_name]
   * ALTER TABLE table_name REPLACE COLUMNS (col_name1 col_type1 [COMMENT col_comment1], ...)
* .option("overwriteSchema", "true") // Replace table schema
*  val deltaTable = DeltaTable.forPath(spark, "{path}")
   * deltaTable.{delete|update|merge|save}
   * After delete, we have to run vacuum to remove unusued files.
   * deltaTable.vacuum({day})
* val deltaTable = DeltaTable.convertToDelta(spark, "parquet.`/path/to/table`") // // Convert unpartitioned parquet table at path '/path/to/table'
* val partitionedDeltaTable = DeltaTable.convertToDelta(spark, "parquet.`/path/to/table`", "part int, part2 int") // // Convert partitioned Parquet table at path '/path/to/table' and partitioned by integer columns named 'part' and 'part2'
* Convert Delta table to a Parquet table
   * If you have performed Delta Lake operations that can change the data files (for example, delete or merge), run vacuum with retention of 0 hours to delete all      data files that do not belong to the latest version of the table.
   * Delete the _delta_log directory in the table directory.
* symlink_format_manifest for other engine to use manifests.
   * val deltaTable = DeltaTable.forPath(pathToDeltaTable)
   * deltaTable.generate("symlink_format_manifest")
* Optimize a table
   * Once you have performed multiple changes to a table, you might have a lot of small files. To improve the speed of read queries, you can use OPTIMIZE to           collapse small files into larger ones:
   * OPTIMIZE delta.`/mnt/delta/events`  OR  OPTIMIZE {table name}
* Z-order by columns
   * To improve read performance further, you can co-locate related information in the same set of files by Z-Ordering. This co-locality is automatically used by        Delta Lake data-skipping algorithms to dramatically reduce the amount of data that needs to be read. To Z-Order data, you specify the columns to order on in        the ZORDER BY clause. For example, to co-locate by eventType, run:
   * OPTIMIZE events ZORDER BY (eventType)
   * Dont forget to clean : VACUUM {table name}
* History
   * val deltaTable = DeltaTable.forPath(spark, pathToTable)
   * val fullHistoryDF = deltaTable.history()    // get the full history of the table
   * val lastOperationDF = deltaTable.history(1) // get the last operation
