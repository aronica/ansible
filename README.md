# ansible

## 前置条件
1. 建立SSH授信(略)
2. 配置Ansible管理的集群
```bash
cat /etc/ansible/hosts
[cluster]
192.168.0.1 ansible_ssh_port=2000 ansible_ssh_user=master
192.168.0.2 ansible_ssh_port=2000 ansible_ssh_user=master
192.168.0.3 ansible_ssh_port=2000 ansible_ssh_user=master


[es]
192.168.0.1 ansible_ssh_port=2000 ansible_ssh_user=master
192.168.0.2 ansible_ssh_port=2000 ansible_ssh_user=master
192.168.0.3 ansible_ssh_port=2000 ansible_ssh_user=master
```
3.每台主机配置ansible local variable
```bash
[master@192.168.0.1 ansible]$ cat /etc/ansible/facts.d/base.fact
[general]
ip=192.168.0.1
node_name=host1
```
4.在ansible宿主机上提前下载好cassandra/elasticsearch/kibana的安装包（使用ansible shell/file模块也能够下载，但是下载到本地分发到其他机器会比每台机器单独下载更快,多次执行也不需要重复下载）,我的下载目录是这样的
```
[master@192.168.0.1 ansible]$ ls /data/bigdata/app/download/
apache-cassandra-3.0.13-bin.tar.gz  kibana-5.4.2-linux-x86_64
elasticsearch-5.4.2                 kibana-5.4.2-linux-x86_64.tar.gz
elasticsearch-5.4.2.zip
```

## Cassandra 安装
1. 修改vars/main.yaml文件中的cassandra配置，可以同步更改roles/cassandra/templates文件中的模板文件以及vars/main.yaml中的变量值
```bash
[master@192.168.0.1 cassandra]$ cat vars/main.yaml
cluster_name: feature_cluster
data_file_directories: /data/var/lib/cassandra/data
commitlog_directory: /data/var/lib/cassandra/log
saved_caches_directory: /data/var/lib/cassandra/caches
hints_directory: /data/var/lib/cassandra/hints
seed: 192.168.80.138
cassandra_version: apache-cassandra-3.0.13-bin
cassandra_version_name: apache-cassandra-3.0.13
```
2. 安装cassandra3并使用supervisord管理进程并同时启动
```bash
ansible-playbook cassandra.yml
```
## Elasticsearch 安装
1. 修改vars/main.yaml文件中的配置（如果需要定制更多配置，可以同步更改roles/elasticsearch/templates文件中的模板文件以及vars/main.yaml中的变量值）
```bash
[master@192.168.0.1 elasticsearch]$ cat vars/main.yaml
cluster_name: dtapp
data_file_directories: /data/var/lib/cassandra/data
commitlog_directory: /data/var/lib/cassandra/log
saved_caches_directory: /data/var/lib/cassandra/caches
hints_directory: /data/var/lib/cassandra/hints
seed: wg-ais-dispatch-arck-7
es_version: elasticsearch-5.4.2
es_version_name: elasticsearch-5.4.2
```
2. 安装Elasticsearch5
```bash
ansible-playbook es.yml
```
## Kibana 安装
1. 修改vars/main.yaml文件中的配置（如果需要定制更多配置，可以同步更改roles/kibana/templates文件中的模板文件以及vars/main.yaml中的变量值）
```bash
[master@192.168.0.1 kibana]$ cat vars/main.yaml
es_url: http://192.168.80.1:9200
seed: 192.168.80.1
kibana_version_gz: kibana-5.4.2-linux-x86_64.tar.gz
kibana_version: kibana-5.4.2-linux-x86_64
```
2.安装Kibana，并且配置ES机器
```bash
ansible-playbook kibana.yml
```
