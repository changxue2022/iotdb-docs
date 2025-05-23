<!--

    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at
    
        http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.

-->

# Java 原生接口

## 安装

### 依赖

* JDK >= 1.8
* Maven >= 3.6



### 在 MAVEN 中使用原生接口

```xml
<dependencies>
    <dependency>
      <groupId>org.apache.iotdb</groupId>
      <artifactId>iotdb-session</artifactId>
      <version>${project.version}</version>
    </dependency>
</dependencies>
```

## 语法说明

 - 对于 IoTDB-SQL 接口：传入的 SQL 参数需要符合 [语法规范](../User-Manual/Syntax-Rule.md#字面值常量) ，并且针对 JAVA 字符串进行反转义，如双引号前需要加反斜杠。（即：经 JAVA 转义之后与命令行执行的 SQL 语句一致。） 
 - 对于其他接口： 
   - 经参数传入的路径或路径前缀中的节点： 在 SQL 语句中需要使用反引号（`）进行转义的，此处均需要进行转义。 
   - 经参数传入的标识符（如模板名）：在 SQL 语句中需要使用反引号（`）进行转义的，均可以不用进行转义。
 - 语法说明相关代码示例可以参考：`example/session/src/main/java/org/apache/iotdb/SyntaxConventionRelatedExample.java`

## 基本接口说明

下面将给出 Session 对应的接口的简要介绍和对应参数：

### 初始化

* 初始化 Session

``` java
// 全部使用默认配置
session = new Session.Builder.build();

// 指定一个可连接节点
session = 
    new Session.Builder()
        .host(String host)
        .port(int port)
        .build();

// 指定多个可连接节点
session = 
    new Session.Builder()
        .nodeUrls(List<String> nodeUrls)
        .build();

// 其他配置项
session = 
    new Session.Builder()
        .fetchSize(int fetchSize)
        .username(String username)
        .password(String password)
        .thriftDefaultBufferSize(int thriftDefaultBufferSize)
        .thriftMaxFrameSize(int thriftMaxFrameSize)
        .enableRedirection(boolean enableRedirection)
        .version(Version version)
        .build();
```

其中，version 表示客户端使用的 SQL 语义版本，用于升级 0.13 时兼容 0.12 的 SQL 语义，可能取值有：`V_0_12`、`V_0_13`、`V_1_0`。

* 开启 Session

``` java
void open()
```

* 开启 Session，并决定是否开启 RPC 压缩

``` java
void open(boolean enableRPCCompression)
```

注意: 客户端的 RPC 压缩开启状态需和服务端一致

* 关闭 Session

``` java
void close()
```

### 数据定义接口 DDL

#### Database 管理

* 设置 database

``` java
void setStorageGroup(String storageGroupId)
```

* 删除单个或多个 database

``` java
void deleteStorageGroup(String storageGroup)
void deleteStorageGroups(List<String> storageGroups)
```
#### 时间序列管理

* 创建单个或多个时间序列

``` java
void createTimeseries(String path, TSDataType dataType,
      TSEncoding encoding, CompressionType compressor, Map<String, String> props,
      Map<String, String> tags, Map<String, String> attributes, String measurementAlias)
      
void createMultiTimeseries(List<String> paths, List<TSDataType> dataTypes,
      List<TSEncoding> encodings, List<CompressionType> compressors,
      List<Map<String, String>> propsList, List<Map<String, String>> tagsList,
      List<Map<String, String>> attributesList, List<String> measurementAliasList)
```

* 创建对齐时间序列

```
void createAlignedTimeseries(String prefixPath, List<String> measurements,
      List<TSDataType> dataTypes, List<TSEncoding> encodings,
      List <CompressionType> compressors, List<String> measurementAliasList);
```

注意：目前**暂不支持**使用传感器别名。

* 删除一个或多个时间序列

``` java
void deleteTimeseries(String path)
void deleteTimeseries(List<String> paths)
```

* 检测时间序列是否存在

``` java
boolean checkTimeseriesExists(String path)
```

#### 元数据模版

* 创建元数据模板，可以通过先后创建 Template、MeasurementNode 的对象，描述模板内物理量结构与类型、编码方式、压缩方式等信息，并通过以下接口创建模板

``` java
public void createSchemaTemplate(Template template);

Class Template {
    private String name;
    private boolean directShareTime;
    Map<String, Node> children;
    public Template(String name, boolean isShareTime);
    
    public void addToTemplate(Node node);
    public void deleteFromTemplate(String name);
    public void setShareTime(boolean shareTime);
}

Abstract Class Node {
    private String name;
    public void addChild(Node node);
    public void deleteChild(Node node);
}

Class MeasurementNode extends Node {
    TSDataType dataType;
    TSEncoding encoding;
    CompressionType compressor;
    public MeasurementNode(String name, 
                           TSDataType dataType, 
                           TSEncoding encoding,
                          CompressionType compressor);
}
```

通过上述类的实例描述模板时，Template 内应当仅能包含单层的 MeasurementNode，具体可以参见如下示例：

``` java
MeasurementNode nodeX = new MeasurementNode("x", TSDataType.FLOAT, TSEncoding.RLE, CompressionType.SNAPPY);
MeasurementNode nodeY = new MeasurementNode("y", TSDataType.FLOAT, TSEncoding.RLE, CompressionType.SNAPPY);
MeasurementNode nodeSpeed = new MeasurementNode("speed", TSDataType.DOUBLE, TSEncoding.GORILLA, CompressionType.SNAPPY);

// This is the template we suggest to implement
Template flatTemplate = new Template("flatTemplate");
template.addToTemplate(nodeX);
template.addToTemplate(nodeY);
template.addToTemplate(nodeSpeed);

createSchemaTemplate(flatTemplate);
```

* 完成模板挂载操作后，可以通过如下的接口在给定的设备上使用模板注册序列，或者也可以直接向相应的设备写入数据以自动使用模板注册序列。

``` java
void createTimeseriesUsingSchemaTemplate(List<String> devicePathList)
```

* 将名为'templateName'的元数据模板挂载到'prefixPath'路径下，在执行这一步之前，你需要创建名为'templateName'的元数据模板
* **请注意，我们强烈建议您将模板设置在 database 或 database 下层的节点中，以更好地适配未来版本更新及各模块的协作**

``` java
void setSchemaTemplate(String templateName, String prefixPath)
```

- 将模板挂载到 MTree 上之后，你可以随时查询所有模板的名称、某模板被设置到 MTree 的所有路径、所有正在使用某模板的所有路径，即如下接口：

``` java
/** @return All template names. */
public List<String> showAllTemplates();

/** @return All paths have been set to designated template. */
public List<String> showPathsTemplateSetOn(String templateName);

/** @return All paths are using designated template. */
public List<String> showPathsTemplateUsingOn(String templateName)
```

- 如果你需要删除某一个模板，请确保在进行删除之前，MTree 上已经没有节点被挂载了模板，对于已经被挂载模板的节点，可以用如下接口卸载模板；


``` java
void unsetSchemaTemplate(String prefixPath, String templateName);
public void dropSchemaTemplate(String templateName);
```

* 请注意，如果一个子树中有多个孩子节点需要使用模板，可以在其共同父母节点上使用 setSchemaTemplate 。而只有在已有数据点插入模板对应的物理量时，模板才会被设置为激活状态，进而被 show timeseries 等查询检测到。
* 卸载'prefixPath'路径下的名为'templateName'的元数据模板。你需要保证给定的路径'prefixPath'下需要有名为'templateName'的元数据模板。

注意：目前不支持从曾经在'prefixPath'路径及其后代节点使用模板插入数据后（即使数据已被删除）卸载模板。


### 数据操作接口 DML

#### 数据写入

推荐使用 insertTablet 帮助提高写入效率

* 插入一个 Tablet，Tablet 是一个设备若干行数据块，每一行的列都相同
  * **写入效率高**
  * **支持批量写入**
  * **支持写入空值**：空值处可以填入任意值，然后通过 BitMap 标记空值

``` java
void insertTablet(Tablet tablet)

public class Tablet {
  /** deviceId of this tablet */
  public String prefixPath;
  /** the list of measurement schemas for creating the tablet */
  private List<MeasurementSchema> schemas;
  /** timestamps in this tablet */
  public long[] timestamps;
  /** each object is a primitive type array, which represents values of one measurement */
  public Object[] values;
  /** each bitmap represents the existence of each value in the current column. */
  public BitMap[] bitMaps;
  /** the number of rows to include in this tablet */
  public int rowSize;
  /** the maximum number of rows for this tablet */
  private int maxRowNumber;
  /** whether this tablet store data of aligned timeseries or not */
  private boolean isAligned;
}
```

* 插入多个 Tablet

``` java
void insertTablets(Map<String, Tablet> tablets)
```

* 插入一个 Record，一个 Record 是一个设备一个时间戳下多个测点的数据。这里的 value 是 Object 类型，相当于提供了一个公用接口，后面可以通过 TSDataType 将 value 强转为原类型

  其中，Object 类型与 TSDataType 类型的对应关系如下表所示：

  | TSDataType | Object         |
  | ---------- | -------------- |
  | BOOLEAN    | Boolean        |
  | INT32      | Integer        |
  | INT64      | Long           |
  | FLOAT      | Float          |
  | DOUBLE     | Double         |
  | TEXT       | String, Binary |

``` java
void insertRecord(String prefixPath, long time, List<String> measurements,
   List<TSDataType> types, List<Object> values)
```

* 插入多个 Record

``` java
void insertRecords(List<String> deviceIds,
        List<Long> times,
        List<List<String>> measurementsList,
        List<List<TSDataType>> typesList,
        List<List<Object>> valuesList)
```

* 插入同属于一个 device 的多个 Record

``` java
void insertRecordsOfOneDevice(String deviceId, List<Long> times,
    List<List<String>> measurementsList, List<List<TSDataType>> typesList,
    List<List<Object>> valuesList)
```

#### 带有类型推断的写入

当数据均是 String 类型时，我们可以使用如下接口，根据 value 的值进行类型推断。例如：value 为 "true" ，就可以自动推断为布尔类型。value 为 "3.2" ，就可以自动推断为数值类型。服务器需要做类型推断，可能会有额外耗时，速度较无需类型推断的写入慢

* 插入一个 Record，一个 Record 是一个设备一个时间戳下多个测点的数据

``` java
void insertRecord(String prefixPath, long time, List<String> measurements, List<String> values)
```

* 插入多个 Record

``` java
void insertRecords(List<String> deviceIds, List<Long> times,
   List<List<String>> measurementsList, List<List<String>> valuesList)
```

* 插入同属于一个 device 的多个 Record

``` java
void insertStringRecordsOfOneDevice(String deviceId, List<Long> times,
    List<List<String>> measurementsList, List<List<String>> valuesList)
```

#### 对齐时间序列的写入

对齐时间序列的写入使用 insertAlignedXXX 接口，其余与上述接口类似：

* insertAlignedRecord
* insertAlignedRecords
* insertAlignedRecordsOfOneDevice
* insertAlignedStringRecordsOfOneDevice
* insertAlignedTablet
* insertAlignedTablets

#### 数据删除

* 删除一个或多个时间序列在某个时间点前或这个时间点的数据

``` java
void deleteData(String path, long endTime)
void deleteData(List<String> paths, long endTime)
```

#### 数据查询

* 时间序列原始数据范围查询：
  - 指定的查询时间范围为左闭右开区间，包含开始时间但不包含结束时间。

``` java
SessionDataSet executeRawDataQuery(List<String> paths, long startTime, long endTime);
```

* 最新点查询：
  - 查询最后一条时间戳大于等于某个时间点的数据。
  ``` java
  SessionDataSet executeLastDataQuery(List<String> paths, long lastTime);
  ```
  - 快速查询单设备下指定序列最新点，支持重定向；如果您确认使用的查询路径是合法的，可将`isLegalPathNodes`置为true以避免路径校验带来的性能损失。
  ``` java
  SessionDataSet executeLastDataQueryForOneDevice(
      String db, String device, List<String> sensors, boolean isLegalPathNodes);
  ```

* 聚合查询：
  - 支持指定查询时间范围。指定的查询时间范围为左闭右开区间，包含开始时间但不包含结束时间。
  - 支持按照时间区间分段查询。

``` java
SessionDataSet executeAggregationQuery(List<String> paths, List<Aggregation> aggregations);

SessionDataSet executeAggregationQuery(
    List<String> paths, List<Aggregation> aggregations, long startTime, long endTime);

SessionDataSet executeAggregationQuery(
    List<String> paths,
    List<Aggregation> aggregations,
    long startTime,
    long endTime,
    long interval);

SessionDataSet executeAggregationQuery(
    List<String> paths,
    List<Aggregation> aggregations,
    long startTime,
    long endTime,
    long interval,
    long slidingStep);
```

### IoTDB-SQL 接口

* 执行查询语句

``` java
SessionDataSet executeQueryStatement(String sql)
```

* 执行非查询语句

``` java
void executeNonQueryStatement(String sql)
```

### 写入测试接口 (用于分析网络带宽)

不实际写入数据，只将数据传输到 server 即返回

* 测试 insertRecord

``` java
void testInsertRecord(String deviceId, long time, List<String> measurements, List<String> values)

void testInsertRecord(String deviceId, long time, List<String> measurements,
        List<TSDataType> types, List<Object> values)
```

* 测试 testInsertRecords

``` java
void testInsertRecords(List<String> deviceIds, List<Long> times,
        List<List<String>> measurementsList, List<List<String>> valuesList)

void testInsertRecords(List<String> deviceIds, List<Long> times,
        List<List<String>> measurementsList, List<List<TSDataType>> typesList,
        List<List<Object>> valuesList)
```

* 测试 insertTablet

``` java
void testInsertTablet(Tablet tablet)
```

* 测试 insertTablets

``` java
void testInsertTablets(Map<String, Tablet> tablets)
```

### 示例代码

浏览上述接口的详细信息，请参阅代码 ```session/src/main/java/org/apache/iotdb/session/Session.java```

使用上述接口的示例代码在 ```example/session/src/main/java/org/apache/iotdb/SessionExample.java```

使用对齐时间序列和元数据模板的示例可以参见 `example/session/src/main/java/org/apache/iotdb/AlignedTimeseriesSessionExample.java`

## 针对原生接口的连接池

我们提供了一个针对原生接口的连接池 (`SessionPool`)，使用该接口时，你只需要指定连接池的大小，就可以在使用时从池中获取连接。
如果超过 60s 都没得到一个连接的话，那么会打印一条警告日志，但是程序仍将继续等待。

当一个连接被用完后，他会自动返回池中等待下次被使用；
当一个连接损坏后，他会从池中被删除，并重建一个连接重新执行用户的操作；
你还可以像创建 Session 那样在创建 SessionPool 时指定多个可连接节点的 url，以保证分布式集群中客户端的高可用性。

对于查询操作：

1. 使用 SessionPool 进行查询时，得到的结果集是`SessionDataSet`的封装类`SessionDataSetWrapper`;
2. 若对于一个查询的结果集，用户并没有遍历完且不再想继续遍历时，需要手动调用释放连接的操作`closeResultSet`;
3. 若对一个查询的结果集遍历时出现异常，也需要手动调用释放连接的操作`closeResultSet`.
4. 可以调用 `SessionDataSetWrapper` 的 `getColumnNames()` 方法得到结果集列名 

使用示例可以参见 `session/src/test/java/org/apache/iotdb/session/pool/SessionPoolTest.java`

或 `example/session/src/main/java/org/apache/iotdb/SessionPoolExample.java`

