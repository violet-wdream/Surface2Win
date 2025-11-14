# Lab5

## 实验目的  

1. 掌握gsql客户端工具本地连接数据库的方法; 
2. 掌握gsql客户端工具远程连接数据库的方法;  
3. 掌握gsql客户端工具使用方法
4.  掌握图形化界面客户端工具Data Studio的安装及使用方法。

 

## 实验步骤

### 确认连接信息

以操作系统用户omm登录数据库主节，检测数据库是否启动。

注意su这里有个`-`

```bash
su - omm
gs_om -t status --detail
```

![image-20251114134306548](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251114134306548.png)

这里显示没有启动，手动启动

```bash
gs_om -t start
```

![image-20251114134426882](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251114134426882.png)

这里显示成功启动。再次查询状态

![image-20251114134619385](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251114134619385.png)

集群状态为normal

如果显示不识别gs指令，说明没有添加环境变量，执行以下内容。

```bash
export PATH=$PATH:/opt/software/openGauss/script
```

显示`[Errno 13] Permission denied: '/opt/software/openGauss/version.cfg'`

说明你在su的时候命令少了`-`

### 确认数据库主节点的端口号

```bash
cat /gaussdb/data/db1/postgresql.conf | grep port 
```

![image-20251114134932124](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251114134932124.png)

数据库主节点实例的服务器IP地址，数据路径和端口号

```bash
192.168.1.156
/gaussdb/data/db1
26000
```

### gsql 命令使用

```bash
gsql -d postgres -p 26000 -r
```

查看帮助

```postgresql
help
\copyright
\h
\help CREATE DATABASE
\?
\q
```

![image-20251114135732449](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251114135732449.png)

直接执行一条显示版权信息的字符串命令

```postgresql
gsql -d postgres -p 26000 -c "\copyright"
\q
```

使用文件作为命令源而不是交互式输入

```bash
mkdir /home/omm/openGauss;
vim /home/omm/openGauss/mysql.sql;
```

输入内容

```sql
select * from pg_user;
```

执行

```bash
gsql -d postgres -p 26000 -f  /home/omm/openGauss/mysql.sql
```

![image-20251114140107404](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251114140107404.png)

如果FILENAME是-（连字符），则从标准输入读取。

```bash
gsql -d postgres -p 26000 -f -;
select * from pg_user;
```

![image-20251114140250779](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251114140250779.png)

列出所有可用的数据库（\l的l表示list） 

```bash
gsql -d postgres -p 26000 -l;
```

![image-20251114140357487](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251114140357487.png)

 设置 gsql 变量 NAME 为VALUE 

```bash
gsql -d postgres -p 26000 -v foo=bar;
\echo :foo;
\q
```

![image-20251114140509908](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251114140509908.png)

打印 gsql 版本信息。

```bash
gsql -V;
```

`gsql (openGauss 1.1.0 build 392c0438) compiled at 2020-12-31 20:08:06 commit 0 last mr`

 使用文件作为输出源

```bash
touch /home/omm/openGauss/output.txt;
vim /home/omm/openGauss/output.txt;
```

输入以下语句：

```postgresql
gsql -d postgres -p 26000 -L /home/omm/openGauss/output.txt;
create table mytable (firstcol int);
insert into mytable values(100);
select * from mytable;
\q
```

查看“output.txt”文档中的内容

```bash
cat /home/omm/openGauss/output.txt;
```

![image-20251114141933206](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251114141933206.png)

 将所有查询输出重定向到文件FILENAME 

```bash
touch /home/omm/openGauss/outputOnly.txt;

gsql -d postgres -p 26000 -o /home/omm/openGauss/outputOnly.txt;
drop table mytable;
create table mytable (firstcol int);
insert into mytable values(100);
select * from mytable;
\q;

cat /home/omm/openGauss/outputOnly.txt;
```

![image-20251114143232855](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251114143232855.png)

静默启动

```
gsql -d postgres -p 26000 -q;
create table t_test (firstcol int); 
insert into t_test values(200); 
select * from t_test; 
\q
```

![image-20251114143747023](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251114143747023.png)

单行运行模式

```bash
gsql -d postgres -p 26000 -S;
select * from t_test;
select * from t_test
\q
```

语句最后结尾有;号和没有;号，效果都一样。

![image-20251114144002236](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251114144002236.png)

**编辑模式**

这个还挺实用，有时候写错了要把前面的全部删除了才能改。

```bash
gsql -d postgres -p 26000 -r;
```

就是可以像常规的命令行一样进行编辑和修改。



远程使用用户名和密码连接数据库

之前记下的数据库相关信息：

```bash
192.168.1.156
/gaussdb/data/db1
26000
```

假设客户是wj，客户端主机IP 10.140.220.87

服务端主机192.168.1.156，需要配置白名单

```bash
gs_guc set -N all -I all -h "host all wj 10.140.220.87/32 sha256";
```

客户使用以下命令远程登录数据库某个用户。

```bash
gsql -d postgres -h 192.168.1.156 -U showmaker -p 26000 -W Bigdata@456;
```

-d 参数指定目标数据库名、-h参数指定主机名、-U参数指定数据库用户名、-p参数指定端口 号信息、-W参数指定数据库用户密码。 

###  gsql 元命令使用



