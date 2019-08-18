---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "JanusGraph搭建及简单使用"
subtitle: ""
summary: ""
authors: []
tags: ['graphdb','janusgraph','knowledge graph']
categories: ['KG']
date: 2019-03-17T21:50:19+08:00
lastmod: 2019-03-17T21:50:19+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

{{% toc %}}

## 基本环境

底层数据库基于HBase，检索服务基于Elasticserach。

系统运行服务大致如下：

```bash
[root@xnode208 ~] jps
21584 GremlinServer
27857 DataNode
23218 Jps
3251 HMaster
27283 NameNode
21707 Console
17131 Elasticsearch
29503 SecondaryNameNode
```

## 安装

下载[janusgraph0.22安装包](https://github.com/JanusGraph/janusgraph/releases/download/v0.2.2/janusgraph-0.2.2-hadoop2.zip)并解压

```bash
wget https://github.com/JanusGraph/janusgraph/releases/download/v0.2.2/janusgraph-0.2.2-hadoop2.zip
unzip janusgraph-0.2.2-hadoop2.zip
cd janusgraph-0.2.2-hadoop2
```

## 配置

在这里我们采用的是启动gremlin-server服务时，加载指定配置文件的方法创建图表，所以需要设置gremlin-server端以及图表的配置文件。

1. 创建gremlin-server端配置文件`janusgraph-0.2.2-hadoop2/conf/gremlin-server/socket-gremlin-server.yaml`，这里我们复制gremlin服务默认的配置文件，在此基础上进行修改

    ```bash
    cp conf/gremlin-server/gremlin-server.yaml conf/gremlin-server/socket-gremlin-server.yaml
    ```

2. 修改`socket-gremlin-server.yaml`：在配置文件中添加graphManager;并在graphs项中添加graph键及其值，一个键值代表一个图表，值表示对图表的设置(可添加多个图表，每一个图表都有自己的配置文件)，如：

    ```bash
    graphManager: org.janusgraph.graphdb.management.JanusGraphManager
    graphs: {
      blablabla,
      graph: conf/gremlin-server/socket-jg-hbase_fyk-server-configraph.properties
    }
    ```

3. 准备上述`socket-gremlin-server.yaml`中对图表进行设置的properties文件，`conf/gremlin-server/socket-jg-hbase_fyk-server-configraph.properties`文件内容如下：

    ```bash
    gremlin.graph=org.janusgraph.core.JanusGraphFactory
    graph.graphname=graph

    storage.backend=hbase  # 设置我们本地启动的hbase作为底层数据库
    storage.hostname=127.0.0.1

    index.search.backend=elasticsearch  # 设置Janus graph自带的es作为我们的检索服务
    index.search.hostname=127.0.0.1

    cache.db-cache = true
    cache.db-cache-clean-wait = 20
    cache.db-cache-time = 180000
    cache.db-cache-size = 0.5
    ```

## 基本用法

依次启动hbase,elasticsearch以及gremlin-server，最后进入gremlin.sh客户端对图表进行操作

- hbase

    ```bash
    [root@xnode208 ~] start-hbase.sh
    ```

- elasticsearch.(注：Janusgraph自带的elastic search服务启动时为确保安全被禁止使用root用户)

    ```bash
    [zkr@xnode208 ~] cd /usr/local/janusgraph-0.2.2-hadoop2
    [zkr@xnode208 elasticsearch] ./bin/elasticsearch
    ```

- gremlin-server(启动成功后，会创建我们在配置文件中设计的图表)

    ```bash
    [zkr@xnode208 janusgraph0.2] ./bin/gremlin-server.sh ./conf/gremlin-server/socket-gremlin-server.yaml
    ```

- gremlin.sh(进入gremlin交互式客户端)

    ```bash
    [root@xnode208 janusgraph0.2] ./bin/gremlin.sh
    gremlin>
    ```

## 加载诸神图

```bash
# 连接gremlin server
gremlin> :remote connect tinkerpop.server conf/remote.yaml session
==>Configured localhost/127.0.0.1:8182-[f6db862e-752c-48db-839b-1b5b16f1786a]
gremlin> :remote console
==>All scripts will now be sent to Gremlin Server - [localhost/127.0.0.1:8182]-[f6db862e-752c-48db-839b-1b5b16f1786a] - type ':remote console' to return to local mode

# 加载诸神图到我们创建的空图表中
gremlin> GraphOfTheGodsFactory.load(graph)
==>null
```

示例数据描述了一部分希腊诸神以及他们居住的诸神殿的相关关系。

![graph-of-the-gods](/img/graph-of-the-gods-2.png)

| 符号         | 含义                     |
| ------------ | ------------------------ |
| 粗体键       | 带索引的键               |
| 星标粗体键   | 具有唯一值的带索引的键   |
| 带下划线的键 | 以顶点为核心的带索引的键 |
| 空心箭头边   | 不能有多个指向的唯一边   |
| 尾部划线的边 | 单向边                   |

在JanusGraph中，实体以顶点表示，关系以边表示，顶点和边都可以具有属性。

## 一些基本操作

### 增操作

```bash
# 添加顶点
v1 = graph.addVertex(label, 'student');  # 创建第一个顶点v1并增加标签
v2 = graph.addVertex();  # 创建第二个顶点没有标签

# 为顶点添加属性
v1.property('id', '1');  # 为顶点v1添加id属性，值为1
v3 = graph.addVertex(label,'girl','name','huahua');  # 创建第三个顶点并且增加标签，属性以及属性值
v4 = graph.addVertex(label,'boy','name','wuyanzu','age',18)  # 创建第四个顶点添加标签以及多个属性属性值

# 添加边
t1 = v1.addEdge('friends', v2);  # 为v1添加关系到v2,并定义这个关系为t1
t2 = v1.addEdge('boyfriend', v2);  # 两个顶点之间可以增加多种关系

# 为边增加属性
t1.property('reason','cool');  # 为t1增加属性
v3.addEdge('boyfriend',v4,'reason','because the reason');  # v3添加关系到v4并且增加关系属性及属性值

# 提交修改
graph.tx().commit();
```

### 删操作

```bash
# 清空
g.V().drop();  # 删除所有点/图
g.E().drop();  # 删除所有边
graph1.close();
JanusGraphFactory.drop(graph1);  # 清空图中的所有数据

# 删除顶点
pluto = g.V().has('name','pluto').next();g.V(pluto).drop().iterate();  # 删除name属性为"pluto"的顶点
g.V().has('keys','ll').drop().iterate();  # 删除keys属性为"ll"的顶点
g.V().hasLabel('student').has('name','ll').drop().iterate();  # 删除标签为student，并且顶点属性name的值为"ll"的顶点

# 删除边
g.E().has('uuu','because the reason').drop().iterate();  # 删除边属性uuu的属性值为because the reason的边
g.E().hasLabel('boyfriend').has('event','the reason').drop().iterate();  # 删除边标签为boyfriend并且边属性event的值为the reason的边

# 删除顶点标签以及顶点属性
g.V().hasLabel('girl').drop();  # 删除标签girl以及标签为girl的所有顶点
g.V().properties('name').drop();  # 删除顶点属性name

# 删除边标签、边属性以及属性值
g.E().hasLabel('boyfriend').drop();  # 删除边标签boyfriend
g.E().properties('uuu').drop();  # 删除边属性uuu
g.E().hasLabel("friend").properties().drop();  # 删除边标签为friend的所有属性以及属性值
g.E().values('because the reason').drop();  # 删除边属性值为because the reason以及对应的属性


graph.tx().commit();  # 提交
```

## 查询

```bash
# 设置g=graph.traversal(),方便查询
gremlin> g = graph.traversal()
==>graphtraversalsource[standardjanusgraph[hbase:[127.0.0.1]], standard]

# 顶点标签查询
g.V();  # 查看所有顶点id
g.V().label();  # 查看所有顶点标签
g.V().hasLabel("god");  # 查看所有标签为god的顶点id
g.V().filter(label().is('god'));  # 用filter查看所有标签为god的顶点id
g.V().has('name','hercules');  # 查看属性为name,值为hercules的顶点

# 顶点属性及属性值查询
g.V().valueMap();  # 遍历每个顶点的属性及属性值(若没有展示空集)
g.V().properties();  # 查看所有顶点的属性及属性值(不展示空)
g.V().hasLabel("god").values();  # 查看所有顶点标签为god的属性值
g.V().hasLabel("god").properties();  # 查看顶点标签为god的所有顶点属性以及属性值
g.V().values('id');  # 查看顶点属性为id的属性值
g.V().properties('id')  # 查看顶点属性为id的属性及属性值

# 边标签查询
g.E();  # 查看所有顶点之间的边 顶点id--->边--->顶点id
g.E().label();  # 查看所有边的标签(关系)
g.E().hasLabel("battled")  # 查看标签为battled的所有边
g.E().filter(label().is('battled'));  # 用filter查看标签为battled的所有边
g.E().has('time',12);  # 查看属性time的值为12的所有边

# 边属性及属性值查询
g.E().valueMap();  # 遍历所有边属性及属性值
g.E().properties();  # 查看所有边属性及属性值
g.E().hasLabel("battled").values();  # 查看所有标签为battled的边属性值
g.E().hasLabel("battled").properties();  # 查看所有标签为battled的边属性以及属性值
g.E().values('reason');  # 查看边属性为reason的属性值
g.E().properties('reason');  # 查看边属性为reason的属性及属性值
```
