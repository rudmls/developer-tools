# Installation sur WSL

## Structure des fichiers

``` sh
.
└─ hadoop-install/
   ├─ start.sh                
   └─ config.sh
```

## Etapes de lancement

- **Lancer le script**

!!! warning
    Le script doit etre lancé en tant que root ou sudoers

```bash
cd hadoop-install && sudo ./start.sh
```

- **Se connecter à l'utilisateur `hadoop`**

```bash
su - hadoop
```

- **Formater le namenode**

```bash
${HADOOP_HOME}/bin/hdfs namenode -format
```

- **Démarrer hadoop**

```bash
hadoop-start.sh
```

## Code

=== "start.sh"

    ```bash
    #!/bin/bash

    hadoop_user="hadoop"
    hadoop_user_home="/home/${hadoop_user}"
    hadoop_password="ipm"
    hadoop_version="3.3.0"
    hadoop_home="/usr/local/hadoop-${hadoop_version}"
    hadoop_url="https://archive.apache.org/dist/hadoop/common/hadoop-${hadoop_version}/hadoop-${hadoop_version}.tar.gz"
    sqoop_version="1.4.7"
    sqoop_hadoop_version="2.6.0"
    sqoop_home="/usr/local/sqoop-${sqoop_version}.bin__hadoop-${sqoop_hadoop_version}"
    sqoop_url="https://archive.apache.org/dist/sqoop/${sqoop_version}/sqoop-${sqoop_version}.bin__hadoop-${sqoop_hadoop_version}.tar.gz"
    mysql_driver_url="https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.48.tar.gz"
    bashrc="${hadoop_user_home}/.bashrc"

    if [ "$(id -u)" -eq 0 ]; then

        ####################
        # INSTALL PACKAGES #
        ####################

        apt remove -y openssh-server && \
        apt-get update && apt-get install -y --no-install-recommends \
        openjdk-8-jdk curl net-tools gnupg openssh-server \
        && rm -rf /var/lib/apt/lists/* \
        && service ssh start || exit 1

        ###################
        # ADD HADOOP USER #
        ###################

        useradd \
        --create-home \
        --password $(openssl passwd -1 ${hadoop_password}) \
        --shell /bin/bash \
        --groups sudo \
        ${hadoop_user} || exit 1

        #################
        # CONFIGURE SSH #
        #################

        mkdir ${hadoop_user_home}/.ssh && \
        ssh-keygen -t rsa -P '' -f ${hadoop_user_home}/.ssh/id_rsa && \
        cat ${hadoop_user_home}/.ssh/id_rsa.pub >> ${hadoop_user_home}/.ssh/authorized_keys && \
        chmod 0700 ${hadoop_user_home}/.ssh && \
        chmod 0600 ${hadoop_user_home}/.ssh/id_rsa && \
        chmod 0644 ${hadoop_user_home}/.ssh/id_rsa.pub && \
        chmod 0644 ${hadoop_user_home}/.ssh/authorized_keys && \
        chown -R ${hadoop_user}:${hadoop_user} ${hadoop_user_home}/.ssh || exit 1

        #########################
        # INSTALL HADOOP BINARY #
        #########################

        curl -O https://dist.apache.org/repos/dist/release/hadoop/common/KEYS && \
        gpg --import KEYS && rm KEYS && \
        curl -fSL "${hadoop_url}" -o /tmp/hadoop.tar.gz && \
        curl -fSL "${hadoop_url}.asc" -o /tmp/hadoop.tar.gz.asc && \
        gpg --verify /tmp/hadoop.tar.gz.asc && \
        tar -xf /tmp/hadoop.tar.gz -C /usr/local/ && \
        rm /tmp/hadoop.tar.gz* && \
        mkdir -p ${hadoop_home}/logs || exit 1

        ########################
        # INSTALL SQOOP BINARY #
        ########################

        curl -O https://archive.apache.org/dist/sqoop/KEYS && \
        gpg --import KEYS && rm KEYS && \
        curl -fSL "${sqoop_url}" -o /tmp/sqoop.tar.gz && \
        curl -fSL "${sqoop_url}.asc" -o /tmp/sqoop.tar.gz.asc && \
        gpg --verify /tmp/sqoop.tar.gz.asc && \
        tar -xf /tmp/sqoop.tar.gz -C /usr/local/ && \
        rm /tmp/sqoop.tar.gz* && \
        wget ${mysql_driver_url} && \
        tar -xzf mysql-connector-java-5.1.48.tar.gz
        cp mysql-connector-java-5.1.48/mysql-connector-java-5.1.48.jar ${sqoop_home}/lib && \
        rm -rf mysql-connector-java-5.1.48* || exit 1

        ############################
        # SET ENVIRONMENT VARIABLE #
        ############################

        echo 'export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64' >> ${bashrc} && \
        echo "export HADOOP_HOME=${hadoop_home}" >> ${bashrc} && \
        echo "export SQOOP_HOME=${sqoop_home}" >> ${bashrc} && \
        echo 'export PATH=$PATH:$HADOOP_HOME/bin:' >> ${bashrc} && \
        echo 'export PATH=$PATH:$HADOOP_HOME/sbin' >> ${bashrc} && \
        echo 'export PATH=$PATH:$SQOOP_HOME/bin' >> ${bashrc} && \
        echo 'export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop' >> ${bashrc} && \
        echo 'export HADOOP_MAPRED_HOME=$HADOOP_HOME' >> ${bashrc} && \
        echo 'export HADOOP_COMMON_HOME=$HADOOP_HOME' >> ${bashrc} && \
        echo 'export HADOOP_HDFS_HOME=$HADOOP_HOME' >> ${bashrc} && \
        echo 'export YARN_HOME=$HADOOP_HOME' >> ${bashrc} && \
        source ${bashrc} || exit 1

        ####################
        # CONFIGURE HADOOP #
        ####################

        sed -i 's/^[ #]*export\s*JAVA_HOME*\s*=\s*.*/export JAVA_HOME=\/usr\/lib\/jvm\/java-8-openjdk-amd64/' \
        /usr/local/hadoop-3.3.0/etc/hadoop/hadoop-env.sh || exit 1

        ./config.sh ${hadoop_home} | exit 1

        ###########################
        # HADOOP USER PERMISSIONS #
        ###########################

        chown -R ${hadoop_user}:${hadoop_user} ${hadoop_home}

        # /usr/local/hadoop-3.3.0/bin/hdfs namenode -format

    else
        echo 'The script must be run as root.' >&2
    fi 

    ```
=== "config.sh"

    ```bash
    #!/bin/bash

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
            <value>hdfs://localhost:9000</value>
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
        <property>
            <name>mapreduce.application.classpath</name>
            <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
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
        <property>
            <name>yarn.nodemanager.env-whitelist</name>
            <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
        </property>
    </configuration>
    EOF

    ##################
    # hadoop-start.sh #
    ##################

    cat << 'EOF' > ${hadoop_home}/sbin/hadoop-start.sh && chmod +x ${hadoop_home}/sbin/hadoop-start.sh
    #!/bin/bash
    ${HADOOP_HOME}/sbin/start-dfs.sh
    ${HADOOP_HOME}/sbin/start-yarn.sh
    EOF

    ##################
    # hadoop-stop.sh #
    ##################

    cat << 'EOF' > ${hadoop_home}/sbin/hadoop-stop.sh && chmod +x ${hadoop_home}/sbin/hadoop-stop.sh
    #!/bin/bash
    ${HADOOP_HOME}/sbin/stop-yarn.sh
    ${HADOOP_HOME}/sbin/stop-dfs.sh
    EOF
    ```