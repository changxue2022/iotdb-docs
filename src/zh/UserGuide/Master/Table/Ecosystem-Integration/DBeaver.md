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

# DBeaver(IoTDB)

DBeaver 是一个 SQL 客户端和数据库管理工具。DBeaver 可以使用 IoTDB 的 JDBC 驱动与 IoTDB 进行交互。

## 1. DBeaver 安装

* DBeaver 下载地址：https://dbeaver.io/download/

## 2. IoTDB 安装

* 方法1：下载 IoTDB 二进制版本
  * IoTDB 下载地址：https://iotdb.apache.org/Download/
  * 版本 >= 2.0.1
* 方法2：从 IoTDB 源代码编译
  * 源代码： https://github.com/apache/iotdb
  * 编译方法参见以上链接中的 README.md 或 README_ZH.md

## 3. 连接 IoTDB 与 DBeaver

1. 启动 IoTDB 服务

   ```shell
   ./sbin/start-server.sh
   ```
2. 启动 DBeaver

3. 打开 Driver Manager

   ![](/img/UserGuide/Ecosystem-Integration/DBeaver/01.png?raw=true)
4. 为 IoTDB 新建一个驱动类型

   ![](/img/UserGuide/Ecosystem-Integration/DBeaver/02.png)

5. 下载 jdbc 驱动， 点击下列网址 [地址1](https://maven.proxy.ustclug.org/maven2/org/apache/iotdb/iotdb-jdbc/) 或 [地址2](https://repo1.maven.org/maven2/org/apache/iotdb/iotdb-jdbc/)，选择对应版本的 jar 包，下载后缀 jar-with-dependencies.jar 的包

    ![](/img/table-dbeaver-8.png)

6. 添加刚刚下载的驱动包，点击 Find Class

   ![](/img/table-dbeaver-9.png)

7. 编辑驱动设置

   ![](/img/table-dbeaver-10.png)

8. 新建 DataBase Connection， 选择 iotdb

   ![](/img/UserGuide/Ecosystem-Integration/DBeaver/06.png)

9.  编辑 JDBC 连接设置

```
JDBC URL: jdbc:iotdb://127.0.0.1:6667/?sql_dialect=table
Username: root
Password: root
```
    
   ![](/img/table-dbeaver-11.png)

10. 测试连接

    ![](/img/table-dbeaver-12.png)

11. 可以开始通过 DBeaver 使用 IoTDB

![](/img/table-dbeaver-7.png)