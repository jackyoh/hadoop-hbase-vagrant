# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
require 'fileutils'

def read_yaml_file(filename)
  filePath = File.join(File.dirname(__FILE__), filename)
  if !File.file?(filePath)
    puts "\e[31mCannot find file: #{ filePath }\e[0m"
    Kernel.exit(0)
  end
  return YAML.load_file(filePath)
end

hadoopVmEnvPath = ENV['HADOOP_VM_ENV_PATH']
if hadoopVmEnvPath == nil
  puts "\e[31mPlease Setting the HADOOP_VM_ENV_PATH Environment Variable\e[0m"
  Kernel.exit(0)
end

$node = read_yaml_file(hadoopVmEnvPath)

Vagrant.configure("2") do |config|
  config.vm.box = $node["box"]
  config.ssh.username = $node["username"]
  config.ssh.password = $node["password"]
  config.vm.box_check_update = false

  # Network Setting
  if ($node["network"]["bridge"])
      config.vm.network $node["network"]["identifier"], \
                        ip: $node["network"]["ip"], \
                        netmask: $node["network"]["netmask"], \
                        bridge: $node["network"]["bridge"]
  else
      config.vm.network $node["network"]["identifier"], \
                        ip: $node["network"]["ip"], \
                        netmask: $node["network"]["netmask"]
  end
      
  # Resource Setting
  config.vm.provider "virtualbox" do | vb|
     vb.cpus = $node["cpus"]
     vb.memory = $node["memory"]
     vb.name = $node["hostname"]
  end

  # SET HOSTNAME
  config.vm.provision "shell", name: "SETUP_HOST_NAME", \
                               inline: "hostnamectl set-hostname " + $node["hostname"]

  config.vm.provision "shell", name: "SETUP_ETC_HOSTS", \
                               inline: "echo " + $node["network"]["ip"] +  "       " + $node["hostname"] + ">>/etc/hosts"

  # Running Shell Script 
  config.vm.provision "shell", name: "SETUP_ENVIRONMENT", \
                               inline: $SETUP_ENVIRONMENT

  config.vm.provision "shell", name: "SETUP_HADOOP_HOME", \
                               inline: $SETUP_HADOOP_HOME

  config.vm.provision "shell", name: "SETUP_HBASE_HOME", \
                               inline: $SETUP_HBASE_HOME
end

$SETUP_ENVIRONMENT = <<SCRIPT
  ssh-keygen -t rsa
  ssh-copy-id root@localhost
  echo "export JAVA_HOME=/root/.sdkman/candidates/java/8u131-zulu" >> /etc/profile
  echo "export HADOOP_HOME=/opt/hadoop-2.7.0" >> /etc/profile
  echo "export PATH=$JAVA_HOME/bin:$HADOOP_HOME/bin:$PATH" >> /etc/profile
  source /etc/profile
SCRIPT

# SETUP Hadoop
$SETUP_HADOOP_HOME = <<SCRIPT
  cd /opt
  wget http://ftp.tc.edu.tw/pub/Apache/hadoop/common/hadoop-2.7.0/hadoop-2.7.0.tar.gz
  tar zxvf hadoop-2.7.0.tar.gz
  chown -R root:root hadoop-2.7.0
  
  mkdir -p /root/hadooptmp
  chmod 755 /root/hadooptmp
  
  mkdir -p /root/namenode
  chmod 755 /root/namenode
  
  mkdir -p /root/dfs
  chmod 755 /root/dfs

  rm -rf /opt/hadoop-2.7.0/etc/hadoop/core-site.xml
  rm -rf /opt/hadoop-2.7.0/etc/hadoop/hadoop-env.sh
  rm -rf /opt/hadoop-2.7.0/etc/hadoop/hdfs-site.xml
  rm -rf /opt/hadoop-2.7.0/etc/hadoop/slaves  
  rm -rf /opt/hadoop-2.7.0/etc/hadoop/yarn-site.xml

  cp /vagrant/conf/core-site.xml /opt/hadoop-2.7.0/etc/hadoop
  cp /vagrant/conf/hadoop-env.sh /opt/hadoop-2.7.0/etc/hadoop
  cp /vagrant/conf/hdfs-site.xml /opt/hadoop-2.7.0/etc/hadoop
  cp /vagrant/conf/slaves /opt/hadoop-2.7.0/etc/hadoop
  cp /vagrant/conf/yarn-site.xml /opt/hadoop-2.7.0/etc/hadoop

  /opt/hadoop-2.7.0/bin/hadoop namenode -format
SCRIPT

# SETUP HBase
$SETUP_HBASE_HOME = <<SCRIPT
  cd /opt
  wget http://ftp.tc.edu.tw/pub/Apache/hbase/1.1.11/hbase-1.1.11-bin.tar.gz  
  tar zxvf hbase-1.1.11-bin.tar.gz
  chown -R root:root hbase-1.1.11
SCRIPT
