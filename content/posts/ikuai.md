---
title: Docker部署爱快grafana监控面板
description: 使用docker部署爱快grafana监控面板
slug: ikuai
date: 2025-06-15T23:35:40+08:00
# lastmod: 2025-06-15T23:35:40+08:00 # Last modification date
tags:
  - docker
  - ikuai
  - grafana
  - prometheus
categories:
  - 软路由
# draft: true
---

先使用docker部署grafan和Prometheus

```bash
docker run --name prometheus -d -p 127.0.0.1:9090:9090 prom/prometheus
docker run --name grafana -d -p 127.0.0.1:3000:3000 grafana/grafana
```

进入Prometheus

```bash
docker exec -it container-id /bin/sh
vim /etc/prometheus/prometheus.yml
```

添加如下命令

```yml
scrape_configs:
  - job_name: 'ikuai_exporter'
    scrape_interval: 2s
    scrape_timeout: 2s
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
      - target_label: __address__
        replacement: 10.10.1.202:8000  # exporter ip
    static_configs:
      - targets:
          - 10.10.1.253 #路由器ip
```

设置exporter

```bash
wget https://raw.githubusercontent.com/NERVEbing/ikuai-aio/master/deploy/docker-compose.yml
docker compose up -d
```

导入grafana面板`19247`

最终效果

![](/ikuai.png)

> 相关链接:

> https://blog.imoe.tech/2022/12/25/48-use-ikuai-exporter-to-gather-metrics/
> https://github.com/NERVEbing/ikuai-aio
