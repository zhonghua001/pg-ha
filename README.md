# PostgreSQL双机热备、高可用
postgresql HA based on pacemaker.

该项目是postgresql双机热备、主从同步方案的部署脚本。该方案中使用pacemaker+corosync组件完成pg的高可用。

详细的部署说明文档，请看博客：

[postgresql双机热备、高可用方案（采用pacemaker+corosync实现）](http://blog.csdn.net/qguanri/article/details/51151974)

网络拓扑：
![topo](images/pg_host.png)

# PostgreSQL双机热备、高可用自动安装脚本

## 环境准备

* 两台CentOS 7 主机，分别为Master和Slave。
* 防火墙使用firewalld，打开如下端口：
	```
	tcp端口：5432
	udp端口: 5405
	```
* 两个未被占用的虚拟IP，要求虚IP和主机在同一子网段。
* 将所有安装脚本及压缩包上传到master/slave主机的/root目录下解压缩 ```tar xvf pg_ha_install.tar```，最后目录结构如下：

	```
	[root@localhost pg_ha_install]# pwd
	/root/pg_ha_install
	[root@localhost pg_ha_install]# tree
	.
	├── auto_change.sh
	├── check_pg.sh
	├── come_over.sh
	├── config.sh
	├── create_repo.sh
	├── ha.conf
	├── install_pacemaker.sh
	├── install_pg_master.sh
	├── install_pg_slave.sh
	├── open_port.sh
	├── readme.md
	├── yum_pacemaker.tar
	└── yum.tar

	0 directories, 13 files
	```
* PostgreSQL使用5432端口，IP使用VIP_PG_MASTER对外提供服务

**所有操作都在root账户下执行，请严格按照如下顺序执行,否则将出错**

**所有操作都在root账户下执行，请严格按照如下顺序执行,否则将出错**

**所有操作都在root账户下执行，请严格按照如下顺序执行,否则将出错**

## 一、安装步骤

(1) 在master和slave主机上都要执行。检查安装环境、配置master、salve的IP

```
cp -r pg_ha_install /root

cd /root/pg_ha_install
./config.sh MASTER_IP=1.2.3.4 SLAVE_IP=1.2.3.5 VIP_PG_MASTER=1.2.3.6 VIP_PG_SLAVE=1.2.3.7 NET=1.2.0.0/16 PG_DIR=/data/postgresql
```

说明：MASTER_IP/SLAVE_IP分别为pg_master/pg_slave主机的真实IP. VIP_PG_MASTER/VIP_PG_SLAVE分别为pg_master/pg_slave的虚拟IP，这些虚拟IP需要确保是同一网段，未被占用的IP。NET为MSATER_IP/SLAVE_IP所在的子网。PG_DIR为数据库的启动路径。


(2) 在master和slave主机上都要执行。打开所需端口

```
./open_port.sh
```

(3) 在master和slave主机上都要执行。创建repo离线安装包

```
./create_repo.sh
```

(4) 在master和slave主机上都要执行。安装pacemaker和pg

```
./install_pacemaker.sh
```

(5) 在master主机上执行。初始化pg master。注意，必须先在master主机上执行pg初始化，再到slave主机上执行pg初始化。

```
./install_pg_master.sh
```

(6) 在salve主机上执行。初始化pg salve。需要输入postgres密码：postgres

```
./install_pg_slave.sh
```

(7) 在master主机上执行。配置主备自动切换

```
./auto_change.sh
```

## 二、检验安装结果

(1) 在master或slave主机上执行。大约需要等待30秒后，数据库启动完成，执行以下脚本检查安装是否正确。

```
./check_pg.sh
```

## 三、模拟master故障、检验切换效果

(1) 查看数据库主从同步状态

```
pcs status

crm_mon -Afr -1
```

(2) 杀掉master主机上的postgres进程，然后查看数据库主从同步状态

```
killall postgres

pcs status

crm_mon -Afr -1
```

## 四、修复故障，恢复双机热备

```
./open_port.sh 

./come_over.sh
```
