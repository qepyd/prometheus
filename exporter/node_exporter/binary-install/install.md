# 1.下载二进制软件包
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz
ls  -l node_exporter-1.9.1.linux-amd64.tar.gz
```

# 2.安装node_exporter
```
tar xf  node_exporter-1.9.1.linux-amd64.tar.gz  -C /usr/local/
ls -ld  /usr/local/node_exporter-1.9.1.linux-amd64
ln -svf /usr/local/node_exporter-1.9.1.linux-amd64   /usr/local/node_exporter
```

# 3.配置/启动node_exporter
**创建node_exporter.service文件**
```
cat >/etc/systemd/system/node_exporter.service<<'EOF'
[Unit]
Description=node_exporter
Documentation=https://prometheus.io/
After=network.target

[Service]
Type=simple

User=root
Group=root

ExecStart=/usr/local/node_exporter/node_exporter \
  --collector.ntp                                \
  --collector.mountstats                         \
  --collector.systemd                            \
  --collector.sysctl                             \
  --collector.ethtool                            \
  --collector.tcpstat
ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=20s
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

**启动/加入开机自启动**
```
## 更改属主/组
chown -R root:root /usr/local/node_exporter/

## 启动
systemctl daemon-reload
systemctl start node_exporter.service
systemctl status node_exporter.service
ss -lntup | grep 9100

## 加入开机自启动
systemctl enable     node_exporter.service
systemctl is-enabled node_exporter.service
```

