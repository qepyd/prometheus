**配置位置**
```
Prometheus 其 prometheus.yml中的 scrape_configs: 字段下进行配置
```

**检查语法格式**
```
## 在prometheus的工作目录下执行以下命令进行检查
./promtool check config ./prometheus.yml

## 重启prometheus
systemctl restart prometheus.service
```


