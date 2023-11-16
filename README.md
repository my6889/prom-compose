# Deploy Pushgateway with TLS and Basic Auth

## 准备环境

* Docker
* Docker-compose
* Ubuntu 18.04 / Ubuntu 20.04 / Ubuntu 22.04

---

## 部署指南
**1.为pushgateway创建证书**   
[参考这篇文章进行签发](https://docs.foofish.cn/%E9%83%A8%E7%BD%B2OpenVPN%E6%9C%8D%E5%8A%A1%E7%AB%AF%E5%B9%B6%E7%AD%BE%E5%8F%91%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE/)

**2.为pushgateway配置基础认证**  
```angular2html
htpasswd -nBC 10 "" | tr -d ':\n'
```
将加密后的口令填入pushgateway.yml文件中。


**3.修改alertmanager配置**

```
$ vim alertmanager/alertmanager.yml
```

* 默认配置会将告警信息发送至[alertmanager-webhook-adapter](https://github.com/bougou/alertmanager-webhook-adapter)
* 如果需要将告警信息发送至邮箱，请自行配置邮箱

**4.修改prometheus配置**   
在prometheus配置中写入pushgateway的认证用户和密码

**5.启动项目**

```
docker-compose up -d 
```

**6.在被监控服务器上安装Node-exporter并配置主动推送**

安装Node-exporter
```angular2html
$ curl -L "https://publish-1300014129.cos.ap-beijing.myqcloud.com/Others/node_exporter-1.6.1" -o /usr/local/bin/node_exporter
$ chmod +x /usr/local/bin/node_exporter

$ cat >> /etc/systemd/system/node_exporter.service <<EOF
[Unit]
Description=Node Exporter
 
[Service]
User=root
ExecStart=/usr/local/bin/node_exporter --web.listen-address=":9100"
 
[Install]
WantedBy=default.target
EOF

$ systemctl daemon-reload
$ systemctl enable node_exporter
$ systemctl start node_exporter
$ systemctl status node_exporter
```

设置主动推送
```angular2html
crontab -e
```

```angular2html
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
* * * * * sleep 10; curl -s http://localhost:9100/metrics | curl -k --data-binary @- https://username:password@{your_pushgateway_address}/metrics/job/$HOSTNAME/instance/$HOSTNAME
```
