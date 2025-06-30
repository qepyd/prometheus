# 1.下载二进制软件包
```
wget https://github.com/prometheus/prometheus/releases/download/v2.24.0/prometheus-2.24.0.linux-amd64.tar.gz
ls -l prometheus-2.24.0.linux-amd64.tar.gz
```

# 2.安装node_exporter
```
tar xf prometheus-2.24.0.linux-amd64.tar.gz  -C /usr/local/
ls -ld /usr/local/prometheus-2.24.0.linux-amd64
ln -svf /usr/local/prometheus-2.24.0.linux-amd64   /usr/local/prometheus
```

# 3.配置/启动prometheus
**创建prometheus.service文件**
```
cat >/etc/systemd/system/prometheus.service<<'EOF'
[Unit]
Description=prometheus service
After=network.target

[Service]
Type=simple

User=root
Group=root

WorkingDirectory=/usr/local/prometheus/
ExecStart=/usr/local/prometheus/prometheus 
KillMode=control-group
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

**启动/加入开机自启动**
```
## 更改属主/组
chown -R root:root /usr/local/prometheus/

## 启动
systemctl daemon-reload
systemctl start prometheus.service
systemctl status prometheus.service
ss -lntup | grep 9090

## 加入开机自启动
systemctl enable     prometheus.service
systemctl is-enabled prometheus.service
```

**浏览器访问prometheus**
```
http://IP:9090
```
