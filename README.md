# Hadoop Docker

A simple Hadoop Docker setup for running MapReduce. Contains example program for word count of [NOW corpus](https://www.english-corpora.org/now/) in Java.

## Usage

```bash
# Build the Docker image
docker build -t ynshung/hadoop-wordcount:latest .

# or use the pre-built image (may not be uploaded yet)
docker compose pull

docker compose up -d
docker exec -it hadoop-node-1 bash

# In the container
start-dfs.sh && start-yarn.sh
```

Navigate to localhost:9870 for the Hadoop NameNode UI and localhost:8088 for the ResourceManager UI.

### Word Count (Java)

#### 1. Compile the Java program
```bash
javac -classpath "$HADOOP_HOME/share/hadoop/common/hadoop-common-3.4.0.jar:$HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-core-3.4.0.jar" -d . WordCount.java
jar -cvf wordcount.jar scripts/*.class
```

#### 2. Prepare input and run the job
```bash
hadoop fs -mkdir -p /input-now && hadoop fs -put input-now/* /input-now/
hadoop jar wordcount.jar scripts.WordCount /input-now /output-word-count
```

### Output
```bash
# Show the output
hadoop fs -cat /output-word-count/part-r-00000

# or copy the output to local
hadoop fs -copyToLocal /output-word-count/part-r-00000 /home/hadoop/output/word_count.txt

# then, in local terminal
docker cp hadoop-node-1:/home/hadoop/output .
```

### Modify number of nodes
To increase/decrease the number of nodes launched (default is 3):
* Modify the `docker-compose.yml` file
* Inside the node container, modify the `workers` file by running `sudo nano $HADOOP_HOME/etc/hadoop/workers`
