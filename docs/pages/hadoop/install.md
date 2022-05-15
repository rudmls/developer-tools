# Installer hadoop


### Installer java

```bash
sudo apt-get update && \
sudo apt-get install -y openjdk-8-jdk && \
java -version
```

### Créer l'utilisateur Hadoop


```bash
sudo useradd \
--create-home \
--password $(openssl passwd -1 ipm) \
--shell /bin/bash \
--groups sudo \
hadoop
```


Se connecter à l'utilisateur

```bash
su - hadoop
```


**Télécharger**

```bash
wget https://archive.apache.org/dist/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz && \
tar -xvzf hadoop-2.7.3.tar.gz && \
sudo mv hadoop-2.7.3 /usr/local/ && \
rm hadoop-2.7.3.tar.gz 
```



```bash
echo '
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=/usr/local/hadoop-2.7.3
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"' >> ~/.bashrc
```

```bash
source ~/.bashrc 
```

### Configurer Hadoop

```bash
sudo mkdir -p /usr/local/hdfs/namenode && \
sudo mkdir -p /usr/local/hdfs/datanode 
```
