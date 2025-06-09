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

> Navigate to localhost:9870 for the Hadoop NameNode UI and localhost:8088 for the ResourceManager UI.

### Java Program Compilation
```bash
javac -classpath "$HADOOP_HOME/share/hadoop/common/hadoop-common-3.4.0.jar:$HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-core-3.4.0.jar" -d . WordCount.java
jar -cvf wordcount.jar scripts/*.class
```

### Running Word Count Example using NOW Corpus
```bash
# Copy the NOW corpus text files into the container
hadoop fs -mkdir -p /input-now && hadoop fs -put input-now/* /input-now/

# Run the word count job on the NOW corpus
hadoop jar wordcount.jar scripts.WordCount /input-now /output-word-count

# View the output
hadoop fs -cat /output-word-count/part-r-00000

# or copy the output to container's local filesystem
hadoop fs -copyToLocal /output-word-count/part-r-00000 /home/hadoop/output/word_count.txt
```

### Using Your Own Text File

In local machine terminal:
```bash
# Copy the text file from local machine into the container
docker cp <path-to-your-text-file> hadoop-node-1:/home/hadoop/dataset.txt
```

In the container terminal:
```bash
# In the container, upload your file to HDFS
hadoop fs -mkdir -p /input-custom && hadoop fs -put dataset.txt /input-custom/

# Run the word count job on your file
hadoop jar wordcount.jar scripts.WordCount /input-custom /output-custom-word-count

# View the output
hadoop fs -cat /output-custom-word-count/part-r-00000

# or copy the output to container's local filesystem
hadoop fs -copyToLocal /output-custom-word-count/part-r-00000 /home/hadoop/output/my_word_count.txt
```

### Copy Output to Local Machine
In the local machine terminal:
```bash
# Copy the output from the container to your local machine
docker cp hadoop-node-1:/home/hadoop/output .
```

### Modify number of nodes
To increase/decrease the number of nodes launched (default is 3):
* Modify the `docker-compose.yml` file
* Inside the node container, modify the `workers` file by running `sudo nano $HADOOP_HOME/etc/hadoop/workers`
