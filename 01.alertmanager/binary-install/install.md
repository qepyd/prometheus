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

# 3.启动altermanager
**创建altermanager.service文件**
```
cat >/etc/systemd/system/altermanager.service<<'EOF'
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
systemctl start  altermanager.service
systemctl status altermanager.service
ss -lntup | grep 9093

## 加入开机自启动
systemctl enable     altermanager.service
systemctl is-enabled altermanager.service
```

**浏览器访问altermanager**
```
http://IP:9093
```

