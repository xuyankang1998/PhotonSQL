# PhotonSQL

Darker, darker, yet darker.
Lighter, lighter, yet lighter.

## 需求

加粗是想加的东西——你们要说是鹅颈就鹅颈好了。

- 定义表结构  
  - 最多32个属性（**任意属性**）  
  - unique属性  
  - 单属性主键定义  
  - 数据类型：int，char（**varchar**, 255），float  

- 建立索引  
  - 对主键自动建立B+树索引  
  - 对unique属性可以建立或者删除B+树索引  
  - B+树索引均为单属性、单值（**其实是不是单属性都一样，反正B+树嘛**）  
  
- 增删改查  
  - 查找（等值，范围，and或者**or**连接的条件查询）  
  - 插入（单条或者**多条**记录）  
  - 删除（单条或者多条记录，注意删除本来就要支持多条）  
  
## 组件

- 核心：存储结构  
  - 使用B+树实现set（集合）  
  - 咦，好象就没有了？  
- 输入：解读SQL语句并执行  
  - 按语法拆分语句  
  - 逐单元解析  
- 输出：表（视图，查询结果什么的）  
  - 执行语句（这个很简单）  
  - 打表（格式排版？）
- 接口  
  - 我们的核心组件可能会做的和SQLite一样是嵌入式的，当然也可以封装成一个应用，不过connector还是要提供的  
- 缓冲区  
  - 一个hash table  
  - 一个队列  

## 语言

CPP +　Python混编写。  
Python主要做SQL的解析。  
CPP写底层数据结构  

## 架构

分为4层：应用层，传输层，网络层，链路层。（Neta一下TCP/IP模型）  
下面的组件很可能就是CPP的类或者python的模块结构了。  

### 应用层

对应讲义中的Interpreter和API（那按照OSI模型叫做表示层可能更好？）。  
包含对外接口组件。解析SQL语句并且通过向传输层下放动词来实现。  
  
包含的组件：  
- Connector  
- SQLInterpreter  
- ThreadManager（或者ProcessManager，取决于多线程并行还是多进程并行）  

与下层传输的方式：  
- 连接池

与上层传输的方式：  
- Socket I/O

### 传输层

对应讲义中的中间三个Manager。  
包含连接外界与数据集的核心部分。主要功能是验证及分发数据到缓冲区。  
  
包含的组件：  
- RecordManager
- IndexManager
- CatalogManager
- BplusTree

与下层传输的方式：  
- 调用下层对象方法

与上层传输的方式：  
- 连接池

### 网络层

对应讲义中的Buffer Manager。  
包含管理缓冲区的组件。无论是读还是写都应该具有缓冲。  

包含的组件：  

- BufferManager
- HashTable

与下层传输的方式：  
- 调用下层对象方法

与上层传输的方式：  
- 返回值

### 链路层

在BufferManager和Files中间的模块。  
包含底层数据链路。提供高效管理文件目录结构的模式。  

包含的组件：  
- FileManager

与下层传输的方式：  
- 文件IO

与上层传输的方式：  
- 返回值

## 分工

确认了这个结构之后就可以开始领锅了。  
分配好了之后写在这里，然后计划各个组件的ddl。
