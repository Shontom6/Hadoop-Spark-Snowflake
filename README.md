# OneDrive Video link
  https://setuo365-my.sharepoint.com/:v:/g/personal/c00313480_setu_ie/EXy6I8_dDYlGmZmQDYHTGMwBF-XyzpkpfSnnxuUw_YFtxw?nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJPbmVEcml2ZUZvckJ1c2luZXNzIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXciLCJyZWZlcnJhbFZpZXciOiJNeUZpbGVzTGlua0NvcHkifX0&e=hA4C4m

# Hadoop and Spark Setup with Snowflake Integration

## Prerequisites
1. Install Vagrant and VirtualBox.
2. Set up two virtual machines (`vm1` and `vm2`).

## Steps to Set Up the Environment

### 1. Initialize Vagrant
```bash
vagrant up
vagrant ssh vm1
```

### 2. Configure SSH Access
```bash
ssh-keygen -t rsa
ssh-copy-id vagrant@node01
ssh-copy-id vagrant@node02
```

### 3. Install Hadoop
```bash
wget https://downloads.apache.org/hadoop/common/hadoop-3.4.1/hadoop-3.4.1.tar.gz
tar -xvzf hadoop-3.4.1.tar.gz
sudo mv hadoop-3.4.1 /usr/local/hadoop
```

### 4. Update Environment Variables
Edit `~/.bashrc`:
```bash
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:/bin/java::")
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```
Apply the changes:
```bash
source ~/.bashrc
```

### 5. Configure Hadoop
Navigate to the Hadoop configuration directory:
```bash
cd /usr/local/hadoop/etc/hadoop
```
Edit the following files:

#### `core-site.xml`
```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://192.168.50.4:9000</value>
  </property>
  <property>
    <name>dfs.webhdfs.address</name>
    <value>hdfs://192.168.50.4:9870</value>
  </property>
</configuration>
```

#### `hdfs-site.xml`
```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>/usr/local/hadoop/hdfs/namenode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>/usr/local/hadoop/hdfs/datanode</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address</name>
    <value>192.168.50.4:9000</value>
  </property>
</configuration>
```

#### `mapred-site.xml`
```xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
```

#### `yarn-site.xml`
```xml
<configuration>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>node01</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
</configuration>
```

#### `workers`
```text
node01
node02
```

### 6. Format and Start Hadoop
```bash
hdfs namenode -format
start-dfs.sh
start-yarn.sh
```

### 7. Install Spark
```bash
wget https://downloads.apache.org/spark/spark-3.5.3/spark-3.5.3-bin-hadoop3.tgz
tar -xvzf spark-3.5.3-bin-hadoop3.tgz
sudo mv spark-3.5.3-bin-hadoop3 /usr/local/spark
```

### 8. Update Environment Variables for Spark
Edit `~/.bashrc`:
```bash
export SPARK_HOME=/usr/local/spark
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
```
Apply the changes:
```bash
source ~/.bashrc
```

### 9. Start Spark
```bash
$SPARK_HOME/sbin/start-master.sh
$SPARK_HOME/sbin/start-slave.sh spark://node01:7077
```

### 10. Verify the Setup
```bash
jps
hdfs dfsadmin -report
$HADOOP_HOME/sbin/hadoop-daemon.sh start datanode
```

### 11. Create HDFS Directory
Create a directory in HDFS for storing data:
```bash
hdfs dfs -mkdir -p /path/in/hdfs
```

### 12. Integrate Snowflake
Download necessary JAR files:
```bash
cd /opt/spark/jars
wget https://repo1.maven.org/maven2/net/snowflake/snowflake-jdbc/3.17.0/snowflake-jdbc-3.17.0.jar
wget https://repo1.maven.org/maven2/net/snowflake/spark-snowflake_2.12/3.0.0/spark-snowflake_2.12-3.0.0.jar
wget https://search.maven.org/remotecontent?filepath=org/apache/spark/spark-sql_2.12/3.5.3/spark-sql_2.12-3.5.3.jar -O spark-sql_2.12-3.5.3.jar
```

#### `SnowflakeIntegration.py`
```python
from pyspark.sql import SparkSession

# Create a Spark session
spark = SparkSession.builder \
    .appName("SnowflakeIntegration") \
    .config("spark.jars", "/opt/spark/jars/snowflake-jdbc-3.17.0.jar,/opt/spark/jars/spark-snowflake_2.12-3.0.0.jar") \
    .getOrCreate()

# Set up Snowflake options
snowflake_options = {
    "sfURL": "xnsgsxr-av28936.snowflakecomputing.com",
    "sfDatabase": "BIGDATA",
    "sfSchema": "SPARK_SNOWFLAKE",
    "sfWarehouse": "WAREHOUSE",
    "sfRole": "ACCOUNTADMIN",
    "sfUser": "C00313480",
    "sfPassword": "Password@123",
    "sfSSL": "true"
}
try:
    df = spark.read \
        .format("snowflake") \
        .options(**snowflake_options) \
        .option("dbtable", "BIGDATA.SPARK_SNOWFLAKE.SOCIAL_MEDIA_DATA") \
        .load()
    df.show()

    print("Data retrieved successfully from Snowflake!")
    hdfs_path = "hdfs://192.168.50.4:9000/path/in/hdfs/"
    df.write \
        .format("csv") \
        .mode("overwrite") \
        .save(hdfs_path)

    print("Data successfully written to HDFS.")
except Exception as e:
    print(f"An error occurred: {e}")
# Stop the Spark session
spark.stop()
```
Run the script:
```bash
spark-submit --master spark://node01:7077 \
    --conf spark.sql.shuffle.partitions=4 \
    --jars /opt/spark/jars/snowflake-jdbc-3.17.0.jar,/opt/spark/jars/spark-snowflake_2.12-3.0.0.jar,/usr/local/spark/jars/spark-sql_2.12-3.5.3.jar \
    /opt/spark/jars/SnowflakeIntegration.py
```

### 13. Read Data from HDFS
#### `readfromhdfs.py`
```python
from pyspark.sql import SparkSession

# Initialize Spark Session
spark = SparkSession.builder \
    .appName("Read CSV from HDFS") \
    .getOrCreate()

# Read the CSV file from HDFS
hdfs_path = "hdfs://node01:9000/path/in/hdfs/part-00000-1111cef1-3c6d-4cc8-9710-827396165aa1-c000.csv"
csv_df = spark.read.csv(hdfs_path, header=True, inferSchema=True)

# Show the DataFrame (print the content)
csv_df.show()
```
Run the script:
```bash
spark-submit /opt/spark/jars/readfromhdfs.py
```

### 14. HDFS Commands
List files:
```bash
hdfs dfs -ls /path/in/hdfs
```
Remove a file:
```bash
hdfs dfs -rm /path/in/hdfs/filename
```

