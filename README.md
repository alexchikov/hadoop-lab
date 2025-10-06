# Лабораторная работа №5: Знакомство с Apache Hadoop

---

- [Лабораторная работа №5: Знакомство с Apache Hadoop](#лабораторная-работа-5-знакомство-с-apache-hadoop)
  - [Разворачиваем Hadoop кластер на двух нодах](#разворачиваем-hadoop-кластер-на-двух-нодах)
  - [Задания](#задания)


---

## Разворачиваем Hadoop кластер на двух нодах

Итак, первым этапом сей лабораторной работы нам предстоит развернуть кластер Hadoop на двух нодах. Сделать это, при должной внимательности, будет проще, чем вы, возможно, думаете :) 
Для начала на понадобятся, собственно, сами ноды, наши вычислительные ресурсы. Получить мы их можем развернув две виртуальные машины, *как локально так и в облаке.* Примерные параметры для наших виртуальных машин будут такими:
* *OS* &mdash; любой дистрибутив Linux, который вам удобен (личная рекомендация Ubuntu 22.04)
* *Диск* &mdash; HDD, не более 35 гб
* *Выч. ресурсы* &mdash; 2 vCPU, 4 гб RAM
* Создаем ssh ключ для подключения к машинам по ssh, [инструкция](https://yandex.cloud/ru/docs/compute/operations/vm-connect/ssh)
* Для машин задаем имена `hadoopnn` и `hadoopdn`

Далее начинаем работу с `hadoopnn`:
1. Обновляем список пакетов командой `sudo apt update` и устанавливаем Java 8, с помощью команды `sudo apt install openjdk-8-jdk -y`.
2. Редактируем файл `~/.bashrc`, длбавляем в конец строку:
   ```bash
   export JAVA_HOME= # здесь указываем домашнюю директорию Java, например /usr/lib/jvm/java-8-openjdk-amd64/
   export PATH=$PATH:$JAVA_HOME:$JAVA_HOME/bin
   ```
3. Генерируем новый ssh-ключ командой `ssh-keygen`.
4. Добавляем хост datanode в `/etc/hosts`. Добавляем `.pub` ключ в `~/.ssh/authorized_keys` на datanode.
5. Проверяем подключение по `ssh` к другой ноде.
6. Скачиваем архив с Hadoop командой `wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz` и распаковываем его.
7. Переносим папку с hadoop в `/opt/hadoop` командой `mv`.
8. Снова редактируем файл `~/.bashrc` &mdash; добавляем туда следующие записи:
   ```bash
   export HADOOP_HOME=/opt/hadoop
   export PATH=$PATH:$HADOOP_HOME/bin
   export PATH=$PATH:$HADOOP_HOME/sbin
   export HADOOP_MAPRED_HOME=$HADOOP_HOME
   export HADOOP_COMMON_HOME=$HADOOP_HOME
   export HADOOP_HDFS_HOME=$HADOOP_HOME
   export YARN_HOME=$HADOOP_HOME
   ```

Теперь переключаемся на `hadoopdn` и проделываем те же шаги.

Далее на двух нодах:
1. Обновляем `$HADOOP_HOME/etc/hadoop/hadoop-env.sh`:
   ```bash
   export JAVA_HOME=<путь до жавы>
   ```
2. Обновляем `$HADOOP_HOME/etc/hadoop/core-site.xml`:
   ```xml
   <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://hadoop_private_namenode_ip:9000</value>
        </property>
    </configuration>
   ```
3. Обновляем `hdfs-site.xml` в той же папке:
   ```xml
   <configuration>
        <property>
            <name>dfs.replication</name>
            <value>1</value>
        </property>
        <property>
            <name>dfs.namenode.name.dir</name>
            <value>file:///usr/local/hadoop/hdfs/data</value>
        </property>
        <property>
            <name>dfs.datanode.data.dir</name>
            <value>file:///usr/local/hadoop/hdfs/data</value>
        </property>
    </configuration>
   ```
4. Обновляем `yarn-site.xml`:
   ```xml
   <configuration>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
        <property>
            <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
            <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>
        <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>hadoop_private_namenode_ip</value>
        </property>
    </configuration>
   ```
5. Обновляем `mapred-site.xml`:
   ```xml
   <configuration>
        <property>
            <name>mapreduce.jobtracker.address</name>
            <value>hadoop_private_namenode_ip:54311</value>
        </property>
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
    </configuration>
   ```
6. Создаем папки для хранения данных на нодах:
   ```bash
   sudo mkdir -p /usr/local/hadoop/hdfs/data
   sudo chown ubuntu:ubuntu -R /usr/local/hadoop/hdfs/data
   chmod 700 /usr/local/hadoop/hdfs/data
   ```
7. Создаем `masters` файл по пути `$HADOOP_HOME/etc/hadoop/masters` и добавляем туда IP-адрес hadoopnn. Далее создаем `workers` файл по пути `$HADOOP_HOME/etc/hadoop/workers`. Добавляем туда адрес обеих нод. 
8. Форматируем namenode:
   ```bash
   hdfs namenode -format
   ```
9. Запускаем кластер командой `start-all.sh` и проверяем доступность web ui для HDFS и YARN, чтобы это сделать, так как адреса приватные, нужно пробросить порты командой `ssh` с флагом `-L`. Например: `ssh -i ~/.ssh/some_key -L 9870:127.0.0.1:9870 hadoop@hadoopnn`. YARN доступен на порту *8088*.

## Задания

1. Создайте папку с нашим именем в корне;
2. Затем положите туда файл `tweets.txt`;
3. Запустите MapReduce приложение `wordcount.jar` для этого файла;
4. Проверьте в YARN Web UI статус работы приложения;
5. Удалите файл `tweets.txt`;