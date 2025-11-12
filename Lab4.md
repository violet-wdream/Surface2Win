### 购买弹性云服务器ECS（openEuler ARM 操作系统）



### 设置字符集参数

SSH远程链接，工具为1remote

![image-20251112144933809](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251112144933809.png)

使用命令设置字符集参数

```bash
cat >>/etc/profile<<EOF export LANG=en_US.UTF‐8 EOF;
source /etc/profile;
```

![image-20251112144755637](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251112144755637.png)

###  修改 python 版本并安装libaio 包

```bash
cd /usr/bin
mv python python.bak 
ln -s python3 /usr/bin/python 
python -V 
```

![image-20251112145338521](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251112145338521.png)

```bash
yum install libaio* -y
```

![image-20251112145508647](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251112145508647.png)

###  安装 openGauss 数据库 

```bash
mkdir -p /opt/software/openGauss;
chmod 755 -R /opt/software;
cd /opt/software/openGauss;
wget https://opengauss.obs.cn-south-1.myhuaweicloud.com/1.1.0/arm/openGauss-1.1.0-openEuler-64bit-all.tar.gz;
```

注意openGauss-1.1.0-这里后面有个`-`，从实验手册上复制下来的时候没复制到。

![image-20251112150305658](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251112150305658.png)

###  创建 XML配置文件

```bash
cd /opt/software/openGauss;
vim clusterconfig.xml; 
```



编辑服务器名称和私有IP

```bash
ip addr
```

1. 名称`ecs-7c8a-5aaf`
2. 私有IP`192.168.1.156`

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<ROOT>
    <CLUSTER> 
        <PARAM name="clusterName" value="dbCluster" />  
        <PARAM name="nodeNames" value="ecs-7c8a-5aaf" />  
        <PARAM name="backIp1s" value="192.168.1.156"/>  
        <PARAM name="gaussdbAppPath" value="/opt/gaussdb/app" />  
        <PARAM name="gaussdbLogPath" value="/var/log/gaussdb" />  
        <PARAM name="gaussdbToolPath" value="/opt/huawei/wisequery" />  
        <PARAM name="corePath" value="/opt/opengauss/corefile"/>  
        <PARAM name="clusterType" value="single-inst"/>  
	</CLUSTER>
    
	<DEVICELIST>
        
		<DEVICE sn="1000001">  
            <PARAM name="name" value="ecs-7c8a-5aaf"/>  
            <PARAM name="azName" value="AZ1"/>  
            <PARAM name="azPriority" value="1"/> 
            <PARAM name="backIp1" value="192.168.1.156"/>  
            <PARAM name="sshIp1" value="192.168.1.156"/>  
              
     <!--dbnode-->  
     <PARAM name="dataNum" value="1"/>  
     <PARAM name="dataPortBase" value="26000"/>  
     <PARAM name="dataNode1" value="/gaussdb/data/db1"/>  
        </DEVICE>  
    </DEVICELIST>  
</ROOT> 
```

![](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251112151758727.png)

### 初始化安装环境

修改performance.sh文件。

```bash
vim /etc/profile.d/performance.sh
```

用#注释`sysctl -w vm.min_free_kbytes=112640 &> /dev/null `这行。 

```sh
CPUNO=`cat /proc/cpuinfo|grep processor|wc -l` 
export GOMP_CPU_AFFINITY=0-$[CPUNO - 1] 
#sysctl -w vm.min_free_kbytes=112640 &> /dev/null 
sysctl -w vm.dirty_ratio=60 &> /dev/null 
sysctl -w kernel.sched_autogroup_enabled=0 &> /dev/null 
```

![image-20251112152105652](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251112152105652.png)

执行预安装前加载安装包中lib库。

```bash
vim /etc/profile
```



```bash
export packagePath=/opt/software/openGauss 
export LD_LIBRARY_PATH=$packagePath/script/gspylib/clib:$LD_LIBRARY_PATH 
```



```bash
source /etc/profile
```

