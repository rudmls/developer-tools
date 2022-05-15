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
    java_home="/usr/lib/jvm/java-8-openjdk-amd64"

    if [ "$(id -u)" -eq 0 ]; then

        #install packages
        apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        openjdk-8-jdk \
        curl \
        gnupg \
        && rm -rf /var/lib/apt/lists/*

        # add hadoop user
        useradd \
        --create-home \
        --password $(openssl passwd -1 ${hadoop_password}) \
        --shell /bin/bash \
        --groups sudo \
        ${hadoop_user}

        # download hadoop
        curl -O https://dist.apache.org/repos/dist/release/hadoop/common/KEYS \
        && gpg --import KEYS && rm KEYS

        curl -fSL "${hadoop_url}" -o /tmp/hadoop.tar.gz \
        && curl -fSL "${hadoop_url}.asc" -o /tmp/hadoop.tar.gz.asc \
        && gpg --verify /tmp/hadoop.tar.gz.asc \
        && tar -xvf /tmp/hadoop.tar.gz -C /usr/local/ \
        && rm /tmp/hadoop.tar.gz*

        # hadoop environment variable
        {
            echo "export JAVA_HOME=${java_home}"
            echo "export HADOOP_HOME=${hadoop_home}"
            echo 'export HADOOP_INSTALL=$HADOOP_HOM'
            echo 'export HADOOP_MAPRED_HOME=$HADOOP_HOME'
            echo 'export HADOOP_COMMON_HOME=$HADOOP_HOME'
            echo 'export HADOOP_HDFS_HOME=$HADOOP_HOME'
            echo 'export HADOOP_YARN_HOME=$HADOOP_HOME'
            echo 'export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native'
            echo 'export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin'
            echo 'export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"'
        } >> /home/${hadoop_user}/.bashrc

        source /home/${hadoop_user}/.bashrc

        mkdir -p /usr/local/hdfs/namenode \
        /usr/local/hdfs/datanode 

        # add java path
        sed -i -e '/JAVA_HOME=/ s/=.*/=\/usr\/lib\/jvm\/java-8-openjdk-amd64/' \
        /usr/local/hadoop-2.7.3/etc/hadoop/hadoop-env.sh

        # add hadoop configuration
        ./config.sh ${hadoop_home}

    else
        echo 'The script must be run as root.' >&2
    fi 

    ```
=== "config.sh"

    ```bash
    #!/usr/bin/env bash

    hadoop_home=$1

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
    ```