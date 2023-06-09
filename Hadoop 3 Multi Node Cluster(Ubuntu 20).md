##  Multi_Node_Cluster(Hadoop 3)


## Connect To the DataCenter with Public Key or Private in case of windows (cmd ,linux subsystem, VM)
 your key is in Downloads folder or Desktop 
```
cd Downloads
```
 put your key_name in place of "file"  & public_dns = your instance      ip_address(53.0.0.0.1)

```
ssh -i file.pem ubuntu@public_dns_address
```
## Update the system
```
sudo apt-get update && sudo apt-get dist-upgrade -y
```
## Copy public key on to the DataCenter main server
#put your key name in place of "multi"  & public_dns = your instance ip_address(53.0.0.0.1)
#open another terminal and fire the below command (were you saved the .pem key cd Downloads || cd Desktop )
```
scp -i multi.pem multi.pem ubuntu@public_dns:~/.ssh
```
## Create a Hadoop user for accessing HDFS
```
sudo addgroup hadoop
```
```
sudo adduser hduser --ingroup hadoop
```
```
sudo adduser hduser sudo
```
```
sudo su hduser
```
## Create local key
```
ssh-keygen
```
```
cd .ssh/
```
```
cat id_rsa.pub >> authorized_keys
```
```
ssh localhost
```
## Copy the instance public key (multi.pem) to hduser's directory
```
sudo su
```
```
cp /home/ubuntu/.ssh/multi.pem /home/hduser/.ssh/
```
```
chown hduser:hadoop /home/hduser/.ssh/multi.pem
```
```
exit
```

## Install Java 8 (Open-JDK)
```
sudo apt install openjdk-8-jdk openjdk-8-jre -y
```
```
java -version
```

## Download and Install Hadoop
```
wget https://dlcdn.apache.org/hadoop/common/stable/hadoop-3.3.5.tar.gz
```
```
tar -xzvf hadoop-3.3.5.tar.gz
```
```
sudo mv hadoop-3.3.5 /usr/local/hadoop
```
```
sudo chown -R hduser:hadoop /usr/local/hadoop
```

## Set Enviornment Variable
```
readlink -f $(which java)
```
```
nano ~/.bashrc
```
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export PATH=$PATH:/usr/local/hadoop/bin/
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop

```
```
source ~/.bashrc
```
```
cd /usr/local/hadoop/etc/hadoop/
```

## Update hadoop-env.sh
```
nano hadoop-env.sh
```
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_LOG_DIR=/var/log/hadoop
```
```
sudo mkdir /var/log/hadoop/
```
```
sudo chown -R hduser:hadoop /var/log/hadoop
```
#Disable FireWall iptables
```
sudo iptables -L -n
```
```
sudo ufw status
```
```
sudo ufw disable
```
## Disabling Transparent Hugepage Compaction
```
cat /sys/kernel/mm/transparent_hugepage/defrag
```
```
sudo nano /etc/init.d/disable-transparent-hugepages
```
```
#!/bin/sh
### BEGIN INIT INFO
# Provides:          disable-transparent-hugepages
# Required-Start:    $local_fs
# Required-Stop:
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Disable Linux transparent huge pages
# Description:       Disable Linux transparent huge pages, to improve
#                    database performance.
### END INIT INFO

case $1 in
  start)
    if [ -d /sys/kernel/mm/transparent_hugepage ]; then
      thp_path=/sys/kernel/mm/transparent_hugepage
    elif [ -d /sys/kernel/mm/redhat_transparent_hugepage ]; then
      thp_path=/sys/kernel/mm/redhat_transparent_hugepage
    else
      return 0
    fi

    echo 'never' > ${thp_path}/enabled
    echo 'never' > ${thp_path}/defrag

    unset thp_path
    ;;
esac

```
```
sudo chmod 755 /etc/init.d/disable-transparent-hugepages
```
```
sudo update-rc.d disable-transparent-hugepages defaults
```

## Restart server or Reboot the instance


## Set Swappiness
```
sudo sysctl -a | grep vm.swappiness
```
```
sudo sysctl vm.swappiness=1
```
## Configure NTP
```
timedatectl status
```
```
timedatectl list-timezones
```
Choose your own timezone
```
sudo timedatectl set-timezone Asia/Kolkata
```
```
sudo apt install ntp -y
```
```
sudo service ssh restart
```

## Configure .profile (make sure you are on NN)
```
 nano .profile
```
copy and paste the script at last of the Description
```
 eval `ssh-agent` ssh-add /home/hduser/.ssh/abc.pem
```
```
 source .profile
```

## Create a AMI at this point
* click on running instance - click on actions - click on image template - click on create image<br>
* After status = Available - click on running AMI - click on Launch instance  from AMI
* Number of instance = 4  
* select security group = default and then Launch the instances

Create 4 nodes from this AMI image
```
ssh rm
```
```
exit
```
```
sudo nano /etc/hosts
```
#fire the above command and copy All (hosts_name)and paste below #127.0.0.1 localhost<br>

Private_ip Private_dns  nn<br>
Private_ip Private_dns  rm<br>
Private_ip Private_dns  1dn                      
Private_ip Private_dns  2dn<br>
Private_ip Private_dns  3dn<br>

#For Example

172.31.18.137 ip-172-31-18-137.ec2.internal nn<br>
172.31.18.137 ip-172-31-18-137.ec2.internal 1dn<br>
172.31.23.79  ip-172-31-23-79.ec2.internal  2dn<br>
172.31.16.236 ip-172-31-16-236.ec2.internal 3dn

#For example : ssh 1dn then sudo nano /etc/hosts then copy all host name and paste and then exit command till 3dn.


Do this for all nodes


## Install and Configure dsh
```
sudo apt install dsh -y
```
```
sudo nano /etc/dsh/machines.list
```
copy and paste the below lines
#localhost
nn
rm
1dn
2dn
3dn
```
dsh -a uptime
```
```
dsh -a source .profile
```
```
cd /usr/local/hadoop/etc/hadoop
```
## Configure masters and slaves
```
nano masters
```
copy and paste below line
```
#localhost
rm
```
```
nano workers
```
```
#localhost
1dn
2dn
3dn
```
## Update core-site.xml
```
nano core-site.xml
```
```
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://nn:9000</value>
  </property>
```
## Update hdfs-site.xml on name node
```
mkdir -p /usr/local/hadoop/data/hdfs/namenode
```
```
nano hdfs-site.xml
```
```
<property>
    <name>dfs.replication</name>
    <value>3</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///usr/local/hadoop/data/hdfs/namenode</value>
  </property>
   <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///usr/local/hadoop/data/hdfs/datanode</value>
  </property>
```

## Create proper directories on datanode's
```
dsh -m 1dn,2dn,3dn mkdir -p /usr/local/hadoop/data/hdfs/datanode
```


## Update yarn-site.xml
```
nano yarn-site.xml
```
```
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>rm</value>
  </property>
<property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
  </property>
```

## Update mapred-site.xml
```
nano mapred-site.xml
```
```
<property>
    <name>mapreduce.jobtracker.address</name>
    <value>rm:54311</value>
  </property>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
<property>
  <name>yarn.app.mapreduce.am.env</name>
  <value>HADOOP_MAPRED_HOME=$HADOOP_MAPRED_HOME</value>
</property>
<property>
  <name>mapreduce.map.env</name>
  <value>HADOOP_MAPRED_HOME=$HADOOP_MAPRED_HOME</value>
</property>
<property>
  <name>mapreduce.reduce.env</name>
  <value>HADOOP_MAPRED_HOME=$HADOOP_MAPRED_HOME</value>
</property>
```
```
sudo chown -R hduser:hadoop $HADOOP_HOME
```
## SCP all the files
```
cd /usr/local/hadoop/etc/hadoop
```
For all nodes

```
scp core-site.xml hdfs-site.xml mapred-site.xml yarn-site.xml workers rm:/usr/local/hadoop/etc/hadoop
```


for remote in "rm:/usr/local/hadoop/etc/hadoop" "1DN:/usr/local/hadoop/etc/hadoop" "2DN:/usr/local/hadoop/etc/hadoop" "3DN:/usr/local/hadoop/etc/hadoop" ; do scp core-site.xml hdfs-site.xml mapred-site.xml yarn-site.xml workers $REMOTE; done


## Format Namenode
```
hdfs namenode -format
```

## Start the cluster of hduser
```
start-dfs.sh
```
## Start the cluster in rm
```
ssh rm
```
```
start-yarn.sh
```
```
exit

```
```
dsh -a jps
```
Lets do some work on it
```
hdfs dfs -mkdir /user
```
```
hdfs dfs -mkdir /user/ubuntu
```
```
hdfs dfs -put hadoop-3.3.5.tar.gz /user/ubuntu
```
```
hdfs dfs -ls
```
```
hdfs dfs -ls -R
```
## WebUI copy the ip_address of your instance and paste in the url  
```
 ip_address:9870
```
Active

## Click on utilities in that click on Browse the folder/directory

## For Mapreduce the ip_address of rm instances
```
ip_address:8088
```
