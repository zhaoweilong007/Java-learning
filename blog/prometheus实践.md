# Prometheus实践

本项目github地址:<https://github.com/zhaoweilong007/spring-boot-matrix/tree/main/prometheus>

**Prometheus负责收集数据，Grafana负责展示数据。其中采用Prometheus 中的 Exporter含：**

- Node Exporter，负责收集 host 硬件和操作系统数据
- cAdvisor，负责收集容器数据
- AlertManager，负责告警管理

## 安装

### docker部署

**docker-compose.yml配置文件**

```yaml
version: '3.7'

networks:
  monitor:
    driver: bridge

services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    hostname: prometheus
    volumes:
      - ./config/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./config/node_down.yml:/etc/prometheus/node_down.yml
    ports:
      - 9090:9090
    links:
      - cadvisor:cadvisor
      - alertmanager:alertmanager
    depends_on:
      - cadvisor
    networks:
      - monitor
    restart: always

  node-exporter:
    image: prom/node-exporter
    container_name: node-exporter
    hostname: node-exporter
    ports:
      - 9100:9100
    networks:
      - monitor
    restart: always

  alertmanager:
    image: prom/alertmanager
    container_name: alertmanager
    hostname: alertmanager
    ports:
      - 9093:9093
    volumes:
      - ./config/alertmanager.yml:/etc/alertmanager/alertmanager.yml
    networks:
      - monitor
    restart: always

  cadvisor:
    image: google/cadvisor
    container_name: cadvisor
    hostname: cadvisor
    ports:
      - 8080:8080
    networks:
      - monitor
    restart: always

  grafana:
    image: grafana/grafana
    depends_on:
      - prometheus
    ports:
      - 3000:3000
    networks:
      - monitor
    restart: always
```

在docker-compose.yml文件目录下，文件config文件夹，增加以下三个配置文件

**prometheus.yml**

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets: [ '192.168.2.83:9093' ]
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "node_down.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    static_configs:
      - targets: [ '192.168.2.83:9090' ]

  - job_name: 'cadvisor'
    static_configs:
      - targets: [ '192.168.2.83:8080' ]

  - job_name: 'node'
    scrape_interval: 8s
    static_configs:
      - targets: [ '192.168.2.83:9100' ]
```

ps: 其中IP设置成本地IP，scrape_configs配置就是prometheus的指标，每个job对应一个指标

**alertmanager.yml**

```yaml
global:
  #全局邮箱配置
  smtp_smarthost: 'smtp.qq.com:465'
  smtp_from: '374244818@qq.com'
  smtp_auth_username: '374244818@qq.com'
  smtp_auth_password: '*****'
  smtp_require_tls: false

route:
  group_by: [ 'alertname' ]
  # 分组等待的时间
  group_wait: 10s
  # 前后两组发送告警的间隔时间
  group_interval: 10s
  # 重复发送告警时间。默认为 1h
  repeat_interval: 10m
  #告警接收器
  receiver: live-monitoring

#告警接收器配置
receivers:
  - name: 'live-monitoring'
    email_configs:
      - to: '374244818@qq.com'
        send_resolved: true # 发送已解决通知
```

**node_down.yml**

```yaml
groups:
  - name: node_down
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          user: test
        annotations:
          summary: "Instance {{ $labels.instance }} down"
          description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minutes."
      - alert: demo-rule-01 # 告警名称
        expr: up{job="prometheus"} < 2 # 告警条件
        for: 10s # 查询时间间隔
        labels:
          severity: critical # 告警级别
        annotations: # 注释，用于完善告警详情
          summary: "{{$labels.job}}: 示例提示" # 概要
          description: "示例描述" # 描述
```

启动容器

```shell
docker-compose up -d
```

打开浏览器

访问prometheus <http://127.0.0.1:9090/>,

![001](../images/prometheus-home.png)

查看prometheus的targets，<http://127.0.0.1:9090/targets>,可以看到注册的指标

![002](../images/prometheus-targets.png)

访问grafana <http://127.0.0.1:3000/>,默认账号admin/admin

![003](../images/grafana.png)

访问alertmanager <http://localhost:9093/>

![004](../images/alert-home.png)


### grafana配置

**配置数据源**

在configuration配置dataSources

![005](../images/grafana-da1.png)
![006](../images/grafana-da2.png)
![007](../images/grafana-da3.png)

选择save&test 成功后back，就可以看到配置好的数据源了

**配置仪表盘**

仪表盘可以自己配置，也可以选择网上的导入，Grafana 提供了 Dashboard 市场<https://grafana.com/grafana/dashboards/> ，提供了大量直接可用的 Dashboard


这里我们导入[Go Metrics](https://grafana.com/grafana/dashboards/10826)，url:https://grafana.com/grafana/dashboards/10826

![008](../images/grafana-db1.png)

注意选择datasources为之前新增的prometheus数据源

![009](../images/grafana-db2.png)


自己还可以接入其他的dashboard


### 热更新prometheus配置

热更新加载方法有两种：

- kill -HUP pid
- curl -X POST http://IP/-/reload

## 项目接入


### 添加依赖

```groovy
dependencies {
    implementation('org.springframework.boot:spring-boot-starter-web')
    implementation('org.springframework.boot:spring-boot-starter-actuator')
    implementation 'io.micrometer:micrometer-registry-prometheus:1.8.5'
}
```


启动项目，访问<http://127.0.0.1:8080/actuator/prometheus>,可以看到应用的 Prometheus 所需的格式的 Metrics 指标数据


再之前的`prometheus.yml`配置文件中增加job节点

```yaml
- job_name: 'prometheusApp'
# 采集地址
metrics_path: '/actuator/prometheus'
# 目标服务器
static_configs:
  - targets: ['192.168.2.83:8787']
```
IP和端口注意修改成自己的

![010](../images/actuator.png)

使用请求方式更新配置文件，然后刷新页面可以看到已经有了

![010](../images/actuator02.png)

导入springboot的仪表盘 <https://grafana.com/grafana/dashboards/10280>

![011](../images/springboot.png)

导入JVM仪表盘 <https://grafana.com/grafana/dashboards/11955>

![011](../images/jvm.png)

以上就是springboot集成prometheus，当然功能远不止如此，更多功能查看官网的文档吧

### 资料

prometheu官网:<https://prometheus.io/>
dashborad仪表盘：<https://grafana.com/grafana/dashboards/>

参考blog:
<https://www.iocoder.cn/Spring-Boot/Prometheus-and-Grafana/?github>，
<https://juejin.cn/post/6844903809517371406>
