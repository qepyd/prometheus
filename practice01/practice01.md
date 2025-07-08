# 第1章 实践的相关环境说明
## 1.1 prometheus相关介绍
**相关网址**
```
所属组织:
    https://github.com/prometheus/
代理托管：
    https://github.com/prometheus/prometheus
官方网站:
    https://prometheus.io/
```

**已从CNCF中毕业**  
Prometheus是继 Kubernetes 之后第二个加入云原生计算基金会 （CNCF） 并毕业的项目。https://landscape.cncf.io/  
CNCF生态将Prometheus分类于Observability（可观测性）类。

<image src="./picture/prometheus-cncf.jpg" style="width: 100%; height: auto;">


**Prometheus和一些生态系统组件所组成的平面图**官方：https://prometheus.io/docs/introduction/overview/#architecture  

<image src="./picture/prometheus-architecture.jpg" style="width: 100%; height: auto;">




## 1.2 为实践所准备的服务器
基于传统环境下的实践(理解prometheus的相关知识点)，不涉及容器环境(docker、kubernetes)。
```
操作系统       主机名     IP地址        部署的应用
ubuntu 20.04   pag01      172.31.8.201  prometheus、alertmanager、grafana
ubuntu 20.04   pag02      172.31.8.202  prometheus、alertmanager、grafana

ubuntu 20.04   vmssi01    172.31.8.203  prometheus的远端存储(VctoriaMetrics的vmstorage、vmselect、vminsert)
ubuntu 20.04   vmssi02    172.31.8.204  prometheus的远端存储(VictoriaMetrics的vmstorage、vmselect、vminsert)
                           #
                           # 以上两台服务器在最后的最后才会涉及到
                           # 
ubuntu 20.04   wyc01      172.31.8.205  wyc项目生产环境服务器(storage层、db层、中间件层、....、应用层、代理层)
ubuntu 20.04   wyc02      172.31.8.206  wyc项目生产环境服务器(storage层、db层、中间件层、....、应用层、代理层)

ubuntu 20.04   jmsco01    172.31.8.205  jmsco项目生产环境服务器(storage层、db层、中间件层、....、应用层、代理层)
ubuntu 20.04   jmsco02    172.31.8.206  jmsco项目生产环境服务器(storage层、db层、中间件层、....、应用层、代理层)
```

# 第2章 相关软件的安装及相关项介绍和配置
## 2.1 Prometheus的安装
**说明**  
在 pag01、page02 服务器上安装prometheus，主要是实现prometheus在抓取指标时的高可用，数据存储没有高可用，因为每个
prometheus都内置有TSDB来作为本地存储。

**架构图**
```
Targes(A实例、B实例、C实例、....)
  \               \
   \pull metrics   \ pull metrics
    \               \
prometheus      promtheus
     |               |
     |               |
  内置TSDB      内置TSDB
     |               |
     |               |
  本地磁盘      本地磁盘  
```

**安装步骤**  
```
#### 下载LTS版本之符合平台的二进制软件包
wget https://github.com/prometheus/prometheus/releases/download/v2.53.5/prometheus-2.53.5.linux-amd64.tar.gz
ls -l prometheus-2.53.5.linux-amd64.tar.gz

tar xf prometheus-2.53.5.linux-amd64.tar.gz  /usr/local/
ls -ld /usr/local/prometheus-2.53.5.linux-amd64

ln -svf /usr/local/prometheus-2.53.5.linux-amd64  /usr/local/prometheus


#### 准备systemctl的services
#  启动参数有指定本地存储TSDB的存储路径、保存时长、block的大小
#  启动参数的--web.enable-lifecycle可支持HTTP关闭prometheus及热加载配置
#  
cat >/etc/systemd/system/prometheus.service<'EOF'
[Unit]
Description=prometheus service
After=network.target

[Service]
Type=simple
# 指定运行进程的用户
User=root
Group=root

# 工作目录,即prometheus的安装后路径
WorkingDirectory=/usr/local/prometheus/

# 启动命令,未指定远端存储,即本地存储,指定了数据存放路径,保存数据有效期
ExecStart=/usr/local/prometheus/prometheus --storage.tsdb.path="data/" \
          --storage.tsdb.retention.time=15d                            \
          --storage.tsdb.retention.size=512MB                          \
          --web.enable-lifecycle

KillMode=control-group
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

#### 更改权限、启动/加入开机自启动
chown -R root:root /usr/local/prometheus/

systemctl daemon-reload
systemctl start  prometheus.service
ss -lntup | grep 9090
systemctl enable prometheus.service

#### 浏览器访问
http://172.31.8.201:9090
http://172.31.8.202:9090
```

## 2.2 添加Targes(prometheus) 

