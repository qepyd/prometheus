# 1.下载二进制软件包
```
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
ls -l alertmanager-0.27.0.linux-amd64.tar.gz
```

# 2.安装node_exporter
```
tar xf alertmanager-0.27.0.linux-amd64.tar.gz  -C /usr/local/
ls -ld /usr/local/alertmanager-0.27.0.linux-amd64/

ln -svf /usr/local/alertmanager-0.27.0.linux-amd64/  /usr/local/alertmanager
```

# 3.启动alertmanager
**创建alertmanager.service文件**
```
cat >/etc/systemd/system/alertmanager.service<<'EOF'
[Unit]
Description=alertmanager service
After=network.target

[Service]
Type=simple

User=root
Group=root

WorkingDirectory=/usr/local/alertmanager/
ExecStart=/usr/local/alertmanager/alertmanager
KillMode=control-group
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

**启动/加入开机自启动**
```
## 更改属主/组
chown -R root:root /usr/local/alertmanager/ 

## 启动
systemctl daemon-reload
systemctl start  alertmanager.service
systemctl status alertmanager.service
ss -lntup | grep 9093

## 加入开机自启动
systemctl enable     alertmanager.service
systemctl is-enabled alertmanager.service
```

**浏览器访问alertmanager**
```
http://IP:9093
```

