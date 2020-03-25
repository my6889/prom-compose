# 一键部署Prometheus-Grafana-Alertmanager

## 准备环境

* Docker
* Docker-compose
* Ubuntu 18.04

**安装Docker**

```
# Ubuntu
$ curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
$ add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
$ apt install docker-ce -y
$ systemctl enable docker
$ systemctl start docker
```

**安装Docker-compose**

```
$ sudo curl -L "https://wood-bucket.oss-cn-beijing.aliyuncs.com/Linux/docker-compose-Linux-x86_64" -o /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```

---

## 项目说明
本项目通过docker-compose整合五个容器，分别为`prometheus`,`grafana`,`alertmanager`,`pg_prometheus`,`prometheus-postgresql-adapter`

**容器说明：**

prometheus：采集各个exporter的数据采集器

grafana：可视化图形面板

alertmanager：负责告警推送

pg_prometheus：远程持久化存储prometheus数据的PostgreSQL数据库容器

prometheus-postgresql-adapter：prometheus和pg_prometheus数据库中间的数据读写接口


---

## 部署指南

**克隆项目**

```
$ git clone https://github.com/my6889/prom-compose
$ cd prom-compose
```

**修改alertmanager配置**

```
$ vim alertmanager/alertmanager.yml
```

* 设置配置文件中smtp发件邮箱`line3-6`
* 设置接收报警邮件的邮箱`line23`
* 设置邮件主题`line26`
* 其余高阶配置请自行查找资料

**修改告警规则配置**

```
$ cd prometheus/rules
$ ls 
disk_rules.yml  down_rules.yml   load_rules.yml  memory_rules.yml
```

这里给出了四项基本的告警规则，如果不满足需求，可自行修改或再添加

**添加export被监控实例**

```
vim prometheus/prometheus.yml
```

* 依照`line25-29`的格式修改或添加export实例
* 其余配置可按需修改，默认不用改

**启动项目**

```
docker-compose up -d 
```

**访问服务**

```
http://宿主机IP:3000    # Grafana
http://宿主机IP:9090    # Prometheus
http://宿主机IP:9093    # Alertmanager
```

<font color=#FF0000 >**完全移除**</font> 

```
# 慎重操作
docker-compose down -v 
rm -r prom-compose
```

---



## 其它说明

* 项目中Prometheus和Alertmanager的配置会挂载到容器中
* Grafana的挂载数据卷默认名称为`grafana-pv`; Prometheus的挂载数据卷默认名称为`prom-tsdb`; pg_prometheus的挂载数据卷默认名称为`pgdata`
* `docker-compose.yml`中容器均设置了`restart=always`，可按需修改；网段默认设置为`172.21.18.0/24`,如果有冲突请自行修改
* Grafana Web登录默认账号密码为`admin/admin`，添加Prometheus数据源时，地址可以是`http://宿主机IP:9090`或`http://prometheus:9090`


