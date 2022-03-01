# Migrate Spark jobs to Dataproc

[Dataproc](https://cloud.google.com/dataproc/) is a fully managed apache spark on GCP. On advantage of on-
premise spark you don't need to bother with server configuration.  

When using Dataproc is a good practice to separate the compute processing from storage and use ephemeral clusters.  

To separate storage from computing on your existing on-premise spark jobs you can replace the `hdfs:///<INPUT_FILE>` inside your code with `gs://<BUCKET_NAME>/<INPUT_FILE>`.  

To get the bucket name used in the Dataproc instance you can run this command `echo "gs://$(gcloud dataproc clusters describe <YOUR_CLUSTERNAME> --region=<YOUR_REGION> --format=json | jq -r '.config.configBucket')"`

This is a chunk of code from [google cloud platform GitHub](https://github.com/GoogleCloudPlatform/training-data-analyst/blob/master/quests/sparktobq/01_spark.ipynb) that illustrates these changes.

### Using hdfs - Storage inside Dataproc cluster
```python
from pyspark.sqlimport SparkSession, SQLContext, Row

spark = SparkSession.builder.appName("kdd").getOrCreate()
sc = spark.sparkContext
data_file = "hdfs:///kddcup.data_10_percent.gz"
raw_rdd = sc.textFile(data_file).cache()
raw_rdd.take(5)
```

### Using google cloud storage - Storage outside Dataproc cluster
```python
from pyspark.sql import SparkSession, SQLContext, Row

spark = SparkSession.builder.appName("kdd").getOrCreate()
sc = spark.sparkContext
data_file = "gs://{}/kddcup.data_10_percent.gz".format(BUCKET)
raw_rdd = sc.textFile(data_file).cache()
raw_rdd.take(5)

.
.
.

connections_by_protocol.
  write.
  format("csv").
  mode("overwrite").
  save("gs://{}/sparktobq/connections_by_protocol".format(BUCKET))
```
 
