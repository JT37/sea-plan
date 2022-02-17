
# Docker 安装 ES 相关

## docker-compose 安装

- `install docker`：安装 `docker`，了解基本语法
- `mkdir docker_es && cd docker_es`：新建配置文件所在文件夹并进入
- `vim docker-compose.yaml`：创建 `docker` 生成容器的 `yaml` 的文件，内容如下代码块
- `docker-compose up`：在配置文件夹下执行此 `docker` 命令
- 注意事项：合理配置 `docker` 占用的机器资源，此配置文件生成 5 个容器，资源分配不够，会导致容器无法运行。成功示例：`CPUs：6 、Memory：4.00GB 、Swap：2.00GB、Disk image size：60 GB`

```yaml
version: '2.2'
services:
  cerebro:
    image: lmenezes/cerebro:0.8.3
    container_name: cerebro
    ports:
      - "9000:9000"
    command:
      - -Dhosts.0.host=http://es7_01:9200
    networks:
      - es7net
  kibana:
    image: docker.elastic.co/kibana/kibana:7.1.0
    container_name: kibana7
    environment:
      - I18N_LOCALE=zh-CN
      - XPACK_GRAPH_ENABLED=true
      - TIMELION_ENABLED=true
      - XPACK_MONITORING_COLLECTION_ENABLED="true"
    ports:
      - "5601:5601"
    networks:
      - es7net
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.1.0
    container_name: es7_01
    environment:
      - cluster.name=es1
      - node.name=es7_01
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.seed_hosts=es7_01,es7_02,es7_03
      - cluster.initial_master_nodes=es7_01,es7_02,es7_03
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es7data1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - es7net
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.1.0
    container_name: es7_02
    environment:
      - cluster.name=es2
      - node.name=es7_02
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.seed_hosts=es7_01,es7_02,es7_03
      - cluster.initial_master_nodes=es7_01,es7_02,es7_03
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es7data2:/usr/share/elasticsearch/data
    ports:
      - 9202:9200    
    networks:
      - es7net
  elasticsearch3:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.1.0
    container_name: es7_03
    environment:
      - cluster.name=es3
      - node.name=es7_03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.seed_hosts=es7_01,es7_02,es7_03
      - cluster.initial_master_nodes=es7_01,es7_02,es7_03
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es7data3:/usr/share/elasticsearch/data
    ports:
      - 9203:9200 
    networks:
      - es7net

volumes:
  es7data1:
    driver: local
  es7data2:
    driver: local
  es7data3:
    driver: local  

networks:
  es7net:
    driver: bridge

```
