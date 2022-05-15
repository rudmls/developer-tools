# Script d'installation

**Fichiers**

``` sh
.
└─ hadoop-install/
   ├─ start.sh                
   └─ config.sh
```

Lancer le script

!!! warning
    Le script doit etre lancé en tant que root ou sudoers

```bash
cd hadoop-install && \
sudo ./start.sh
```

Contenu

=== "start.sh"

    ```bash
    #!/usr/bin/env bash

    hadoop_user="hadoop"
    hadoop_password="ipm"
    hadoop_version="2.7.3"
    hadoop_url="https://archive.apache.org/dist/hadoop/common/hadoop-${hadoop_version}/hadoop-${hadoop_version}.tar.gz"
    hadoop_home="/usr/local/hadoop-${hadoop_version}"
    sqoop_version="1.4.7"
    sqoop_hadoop_version="2.6.0"
    sqoop_home="/usr/local/sqoop-${sqoop_version}.bin__hadoop-${sqoop_version}"
    sqoop_url="https://archive.apache.org/dist/sqoop/${sqoop_version}/sqoop-${sqoop_version}.bin__hadoop-${sqoop_hadoop_version}.tar.gz"
    mysql_driver_url="https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.48.tar.gz"
    bashrc="/home/${hadoop_user}/.bashrc"
    java_home="/usr/lib/jvm/java-8-openjdk-amd64"

    if [ "$(id -u)" -eq 0 ]; then

        #install packages
        apt-get update && apt-get install -y --no-install-recommends \
        openjdk-8-jdk curl gnupg \
        && rm -rf /var/lib/apt/lists/*

        # add hadoop user
        useradd \
        --create-home \
        --password $(openssl passwd -1 ${hadoop_password}) \
        --shell /bin/bash \
        --groups sudo \
        ${hadoop_user}

        # download hadoop
        curl -O https://dist.apache.org/repos/dist/release/hadoop/common/KEYS && \
        gpg --import KEYS && rm KEYS && \
        curl -fSL "${hadoop_url}" -o /tmp/hadoop.tar.gz && \
        curl -fSL "${hadoop_url}.asc" -o /tmp/hadoop.tar.gz.asc && \
        gpg --verify /tmp/hadoop.tar.gz.asc && \
        tar -xvf /tmp/hadoop.tar.gz -C /usr/local/ && \
        rm /tmp/hadoop.tar.gz*

        # download sqoop
        curl -O https://archive.apache.org/dist/sqoop/KEYS && \
        gpg --import KEYS && rm KEYS && \
        curl -fSL "${sqoop_url}" -o /tmp/sqoop.tar.gz && \
        curl -fSL "${sqoop_url}.asc" -o /tmp/sqoop.tar.gz.asc && \
        gpg --verify /tmp/sqoop.tar.gz.asc && \
        tar -xf /tmp/sqoop.tar.gz -C /usr/local/ && \
        rm /tmp/sqoop.tar.gz* && \
        wget ${mysql_driver_url} && \
        tar -xzf mysql-connector-java-5.1.48.tar.gz && \
        cp mysql-connector-java-5.1.48/mysql-connector-java-5.1.48.jar $SQOOP_HOME/lib && \
        rm -rf mysql-connector-java-5.1.48*

        # hadoop environment variable
        echo "export JAVA_HOME=${java_home}" >> ${bashrc} && \
        echo "export SQOOP_HOME=${sqoop_home}" >> ${bashrc} && \
        echo "export HADOOP_HOME=${hadoop_home}" >> ${bashrc} && \
        echo 'export HADOOP_INSTALL=$HADOOP_HOM' >> ${bashrc} && \
        echo 'export HADOOP_MAPRED_HOME=$HADOOP_HOME' >> ${bashrc} && \
        echo 'export HADOOP_COMMON_HOME=$HADOOP_HOME' >> ${bashrc} && \
        echo 'export HADOOP_HDFS_HOME=$HADOOP_HOME' >> ${bashrc} && \
        echo 'export HADOOP_YARN_HOME=$HADOOP_HOME' >> ${bashrc} && \
        echo 'export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native' >> ${bashrc} && \
        echo 'export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin' >> ${bashrc} && \
        echo 'export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"' >> ${bashrc} && \

        source ${bashrc}

        mkdir -p /usr/local/hdfs/namenode \
        /usr/local/hdfs/datanode 

        # add java path
        sed -i -e '/JAVA_HOME=/ s/=.*/=\/usr\/lib\/jvm\/java-8-openjdk-amd64/' \
        /usr/local/hadoop-2.7.3/etc/hadoop/hadoop-env.sh

        # add hadoop configuration
        ./config.sh ${hadoop_home}

        ${hadoop_home}/bin/hdfs namenode -format

    else
        echo 'The script must be run as root.' >&2
    fi 

    ```
=== "config.sh"

    ```bash
    #!/usr/bin/env bash

    hadoop_home=$1

    #################
    # core-site.xml #
    #################
    
    cat << 'EOF' > ${hadoop_home}/etc/hadoop/core-site.xml 
    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://hadoop-master:9000</value>
            <description>Serveur de données</description>
        </property>
    </configuration>
    EOF

    #################
    # hdfs-site.xml #
    #################

    cat << 'EOF' > ${hadoop_home}/etc/hadoop/hdfs-site.xml
    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <property>
            <name>dfs.replication</name>
            <value>1</value>
            <description>Nombre de copies pour chaque bloc de données (par défaut : 3)</description>
        </property>
        <property>
            <name>dfs.namenode.name.dir</name>
            <value>file:/usr/local/hdfs/namenode</value>
        </property>
        <property>
            <name>dfs.datanode.data.dir</name>
            <value>file:/usr/local/hdfs/datanode</value>
        </property>
        <property>
            <name>dfs.blocksize</name>
            <value>4m</value>
            <description>Taille du bloc de données (par défaut : 128m). Bloc de taille petite (4m) pour éviter une surcharge de l'espace de la VM.</description>
        </property>
    </configuration>
    EOF

    ###################
    # mapred-site.xml #
    ###################

    cat << 'EOF' > ${hadoop_home}/etc/hadoop/mapred-site.xml
    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
    </configuration>
    EOF

    #################
    # yarn-site.xml #
    #################

    cat << 'EOF' > ${hadoop_home}/etc/hadoop/yarn-site.xml
    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
    </configuration>
    EOF

    ###################
    # hadoop-start.sh #
    ###################

    cat << 'EOF' > ${hadoop_home}/sbin/hadoop-start.sh
    #!/usr/bin/env bash
    echo -e "\n"
    $HADOOP_HOME/sbin/start-dfs.sh
    echo -e "\n"
    $HADOOP_HOME/sbin/start-yarn.sh
    echo -e "\n"
    $HADOOP_HOME/sbin/mr-jobhistory-daemon.sh start historyserver
    EOF

    ##################
    # hadoop-stop.sh #
    ##################

    cat << 'EOF' > ${hadoop_home}/sbin/hadoop-stop.sh
    #!/usr/bin/env bash
    mr-jobhistory-daemon.sh stop historyserver
    stop-yarn.sh
    stop-dfs.sh
    EOF
    ```