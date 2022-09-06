![logo](./assets/logo.png)
# 自动引导车辆(AGV)与RCS之间通信接口

## VDA 5050

## 版本 2.0.0

**定义转换(翻译增加非原版)** 
1. the  control system -> RCS 机器人控制系统

2. automated guided vehicles -> AGV

3. node -> point(原文档没有point,如果出现point可以看做是node) 点

4. edge -> segment(原文档没有segment,如果出现segment可以看做是edge) 片段(直线 弧线)



![RCS和自动导向车辆](./assets/csagv.png) 



### 简短信息

无人驾驶运输系统的通信接口的定义 (DTS).
该建议描述了RCS和AGV之间交换任务和状态数据的内部通信接口.  



### 免责声明 

以下内容说明了自动引导车辆(AGV)和RCS之间通信接口,该接口可自由使用,适用于所有人且不具有约束力.

应用这些规则的人必须确保在特定情况下适当使用.
他们应考虑到每个问题时盛行的最新技术.
通过实施这些建议,任何人都不逃避自己行动的责任.
这些陈述并不声称是详尽无遗的,也不是对现有立法的确切解释.
它们不能取代对相关政策、法律和法规的研究.
此外,必须考虑到各产品的特殊特性及其不同的可能应用.
在这方面,每个人都有自己的风险.
应该把VDA以及参与提案的开发或应用的责任 排除在外.
如果您在应用时遇到任何不准确之处或可能出现不正确的解释,请立即通知VDA,以便纠正任何缺陷

**Publisher**
Verband der Automobilindustrie e.v. (VDA)
Behrenstrasse 35, 10117 Berlin 
www.vda.de

**Copyright**
Association of the Automotive Industry (VDA)
Reproduction and any other form of reproduction is only permitted with specification of the source.

Version 2.0



## 目录

[1 前言](#Foreword)<br>
[2 文档的目的](#Ootd)<br>
[3 范围](#Scope)<br>
[3.1 其他适用的文件](#Oad)<br>
[4 需求和协议定义](#Rapd)<br>
[5 通信的过程和内容](#Pacoc)<br>
[6 协议规范](#Ps)<br>
[6.1 Symbols of the tables and meaning of formatting](#Sottamof)<br>
[6.1.1 可选 字段](#Of)<br>
[6.1.2 允许的字符和字段长度](#Pcafl)<br>
[6.1.3 枚举的符号](#Noe) <br>
[6.1.4 JSON数据类型](#JD)<br>
[6.2 MQTT连接处理,安全性和QoS](#MchsaQ)<br>
[6.3 MQTT主题水平](#MTL)<br>
[6.4 协议标头](#PH)<br>
[6.5 通信Subtopics](#Sfc)<br>
[6.6 主题: "task" (从 RCS 到 AGV)](#TOfmctA)<br>
[6.6.1 概念和逻辑](#CaL)<br>
[6.6.2 任务和任务更新](#Oaou)<br>
[6.6.3 取消任务 (通过RCS)](#OCbMC)<br>
[6.6.3.1 取消后收到新任务](#Ranoac)<br>
[6.6.3.2 当AGV没有任务时,接收到取消订单](#RacawAhno)<br>
[6.6.4 任务拒绝](#Or)<br>
[6.6.4.1 车辆得到了格式不正确的新任务](#Vgamno)<br>
[6.6.4.2 车辆收到一个行动无法执行的任务 (例如 提起高度高于最大提升高度或没有安装举升设备的举升动作), 或与无法使用的字段 (例如 Trajectory)](#Vraowaicpeglhhtmlholaansii)<br>
[6.6.4.3 车辆获得了一项相同orderid的新任务,但orderUpdateId比当前的低](#Vehiclegets)<br>
[6.6.5 Maps](#Maps)<br>
[6.7 任务消息的实施](#Iotom)<br>
[6.8 actions](#actions)<br>
[6.8.1 预定义的动作,其参数,效果和范围](#Padtpeas)<br>
[6.8.2 预定义的动作,其参数,效果和范围](#Padtpeas1)<br>
[6.9 主题: "instantactions" (从 RCS 到 AGV)](#Tifmc)<br>
[6.10 主题: "state" (从 AGV 到 RCS)](#TSfAtmc)<br>
[6.10.1 概念和逻辑](#CaLe)<br>
[6.10.2 途径点和进入/离开细片段,动作触发](#Tonaeletoa)<br>
[6.10.3 Base请求](#Br)<br>
[6.10.4 信息Information](#Information)<br>
[6.10.5 错误Errors](#Errors)<br>
[6.10.6 执行Implementation](#Implementation)<br>
[6.11 actionStates](#actionStates)<br>
[6.12 action阻塞类型和顺序](#ABTas)<br>
[6.13 主题 "visualization"](#TV)<br>
[6.14 主题 "connection"](#Tc)<br>
[6.15 主题 "factsheet"](#Tf)<br>
[7 最佳实践](#Bp)<br>
[7.1 Error reference](#Er)<br>
[7.2 参数格式](#Fop)<br>
[8 词汇表](#Glossary)<br>
[8.1 定义](#Definition)<br>



# <a name="Foreword"></a> 1 Foreword 


该接口是 Verband der Automobilindustrie e. V. (German abbreviation VDA) 和 Verband Deutscher Maschinen-und Anlagenbau e. V. (German abbreviation VDMA)合作的.双方的目的是创建一个通用的接口. 更改接口的提议应提交给VDA与VDMA共同评估,并在做出积极决定的情况下被采用为新版本状态.非常感谢通过GitHub对本文档的贡献.可以在以下链接中找到: http://github.com/vda5050/vda5050.



# <a name="Ootd"></a> 2 文档的目的

本建议的目的是简化新车辆与现有RCS的连接,从而在汽车工业中使用时集成到现有的自动引导车辆(AGV)系统中,并在同一工作环境中与来自不同制造商的AGV和传统系统(库存系统)进行并行操作.

定义RCS和AGV之间的统一接口.
具体而言,通过以下几点实现：
- 描述AGV和RCS之间的通信标准,从而为使用合作运输车辆 将运输系统集成到连续自动化中奠定基础.

- 除其他外,通过增加车辆自主性、过程模块和接口,以及更好的将事件控制命令链的序列化分开,提高灵活性.

- 根据所需信息,由于高“即插即用”能力,缩短了实施时间(例如 任务信息)由RCS提供并且有效.考虑到职业安全的要求,车辆应能够独立于制造商投入运行,并采取相同的实施措施.

- 通过对所有运输车辆、车辆模型和制造商使用统一的总体协调以及相应的逻辑,降低复杂性并提高系统的“即插即用”能力.

- 使用车辆控制和协调层之间的通用接口,提高制造商的独立性.

- 通过在专有RCS和上级RCS之间实现垂直通信,集成专有DTS库存系统(参见图1)

![图1 DTS库存系统的集成](./assets/Figure1.png)
>图1 DTS库存系统的集成

为了实现上述目标,本文件描述了AGV和RCS之间任务和状态信息通信的接口.

AGV和RCS之间运行所需的其他接口(例如, 交换地图信息,等等.) 或用于与其他系统组件通信(例如, 外部外围设备,防火门,等等.) 不包括在本文件中.



# <a name="Scope"></a> 3 范围

本建议包含关于自动制导车辆(AGV)和RCS之间通信的定义和最佳实践.目标是允许AGV具有不同的特性(例如, 欠载牵引车或叉车AGV)以统一语言与RCS通信.这为在RCS中操作AGV的任何组合奠定了基础.RCS提供任务并协调AGV交通.

该接口基于汽车行业生产和工厂物流的要求.

根据制定的要求,内部物流的要求涵盖了物流部门的要求,即通过无控制导航车辆和引导车辆,从货物接收到生产供应到货物输出的物流流程.



与自动车辆相比,自动车辆独立地解决基于相应传感器系统和算法发生的问题,并且可以相应地对动态环境中的变化作出反应,或者在不久之后适应这些变化.自主特性,例如独立绕过障碍物,可以通过自由导航车辆和引导车辆来实现.然而,一旦在车辆本身上执行路径规划,本文档将描述自由导航车辆(参见术语表).自主系统不是完全分散的(群体智能),通过预定义的规则定义行为.

为了实现可持续解决方案,下面描述了一个接口,该接口可以在其结构中进行扩展.

这应该能够完全覆盖被引导车辆的RCS.可自由导航的车辆可集成到结构中；本建议不包括为此所需的详细规范.

对于专有库存系统的集成,可能需要接口的单独定义,这不被视为本建议的一部分.


## <a name="Oad"></a> 3.1 其他适用的文件

Document (Dokument) | Description 
----------------------------------| ----------------
VDI Guideline 2510 | Driverless transport systems (DTS)
VDI Guideline 4451 Sheet 7 | Compatibility of driverless transport systems (DTS) - DTS RCS 
DIN EN ISO 3691-4 | Industrial Trucks Safety Requirements and Verification-Part 4: Driverless trucks and their systems 



# <a name="Rapd"></a> 4 需求和协议定义 

交互接口旨在支持以下要求: 

- 控制最少. 1000辆车
- 能够集成不同自动程度的车辆
- 具备决策功能, 例如, 在十字路口选择路径或行为

车辆应定期传递其状态或有改变状态时候传递.

通过无线网络进行通信,要考虑到连接故障和消息丢失的影响.

消息日志采用MQTT,与JSON结构一起使用.MQTT 3.1.1在此协议的开发过程中进行测试,这是兼容性所需的最低版本.MQTT允许将消息分发给subchannels,这些消息称为"topics".MQTT网络的参与者订阅这些主题并接收有关或感兴趣的信息.

JSON结构允许扩展协议,并具有其他参数.这些参数以英语描述,以确保该协议在德语区域之外可读,可理解和适用.



# <a name="Pacoc"></a> 5 通信的过程和内容

如通往AGV操作的信息流中所示, 至少有以下参与者 (见图2): 

- 操作员提供基本信息配置
- RCS组织和管理操作
- AGV执行任务

图2描述了操作阶段的通信内容.在实施或修改过程中,AGV和RCS进行手动配置. 

![图2信息流的结构](./assets/Figure2.png)
>图2信息流的结构

在实施阶段,  DTS由RCS和AGV组成.
必要的框架条件由操作员定义,所需的信息要么由他手动输入,要么通过从其他系统中导入RCS中存储在RCS中. 
本质上,这涉及以下内容:

- 路线的定义: 使用 CAD 导入,  可以在RCS中接管路线.
另外,操作员也可以在RC中手动实现路线.
路线可以是单向路线,受到某些车辆组的限制 (基于尺寸比), 等等.
- 路径网络配置:
在路线定义,取放货站台,电池充电站,外围环境(门,电梯,障碍),等待位置,缓冲区站,等等. 
- 车辆配置: 操作员存储AGV的机械特性(大小,可用的货物装载安装座,等等).
AGV必须通过子主题`factsheet`传达此信息  以特定的方式定义文档 [AGV Factsheet section](#factsheet).

路线的配置和上述路径网络不是本文档的一部分.它构成了基于此信息和要完成的运输要求,是RCS任务控制和行驶分配的基础.然后,通过MQTT消息经broker将AGV的最终任务下发到车辆.然后,并行地向RCS连续报告其状态.这也是使用MQTT消息broker完成的.

RCS的功能是: 

- 将任务分配给AGV
- AGV的路线计算和指引 (考虑到每个AGV的单个物理特性的局限性, 例如, 大小, 机动性, 等等.)
- 检测和解决阻塞 ("死锁")
- 能源管理：充电任务可以中断转移任务
- 交通控制：缓冲路线和等待位置
- (暂时的) 环境变化, 例如释放某些区域或更改最大速度
- 与外围系统的通信 例如门,大门,电梯, 等等. 
- 检测和解决通信错误

AGV的功能是: 

- 定位
- 沿相关路线行走(规划指导或自主) 
- 连续传输车辆状态

此外, 配置整体系统时,集成商必须考虑以下内容 (列表不完整): 

- MAP配置：必须匹配RCS和AGV的坐标系.
- 中心点: 使用AGV的不同点或充电点作为支点会导致车辆的不同包络(不明确).参考点可能因情况而异, 例如,对于载货和不载货的AGV可能会有所不同.



# <a name="Ps"></a> 6 协议规范

以下部分描述了通信协议的详细信息.该协议定义RCS与AGV之间的通信.AGV和外围设备之间的通信, 例如, 在AGV和一个门之间通信被排除在外.

表中显示了不同的消息,描述了作为任务,state发送的JSON字段的内容,等等.

此外, JSON schemas可在公共GIT存储库中验证 (https://github.com/VDA5050/VDA5050/json_schemas).JSON schemas 随着VDA5050每个release更新.



## <a name="Sottamof"></a> 6.1 Symbols of the tables and meaning of formatting

该表包含标识符的名称,其单元,其数据类型以及描述(如果有).

Identification | Description [ENG]
---|----
standard | 变量是基本数据类型 
**bold** | 变量是一种非基本数据类型 (例如, JSON-object或者array)并且单独定义
*italic* | 变量可选 
[Square brackets] | 变量(这里arrayname) 是数据类型(方括号中包含)的数组 (这里的数据类型是squareBrackets)

所有关键字都是区分大小写的,所有字段名称均为驼峰命名法,所有枚举都是大写.



### <a name="Of"></a> 6.1.1 可选 字段

如果变量标记为可选,这意味着它是可选,因为变量在某些情况下对于发送人可能不适用 (例如, 当RCS将任务发送到AGV时,某些AGV可以自动寻迹,就可以省略任务段中的trajectory字段). 

如果AGV收到包含该协议中标记为可选的字段的消息,则期望AGV采取相应的行动,并且不能忽略该字段.如果AGV无法相应地处理消息,那么预期的行为是将其传达在错误消息中并拒绝任务.

RCS仅发送AGV支持的信息.

例子: 轨迹是可选. 
如果AGV无法处理轨迹,RCS不得向车辆发送轨迹.

AGV必须通过AGV FactSheet消息传达其需要的可选参数.


### <a name="Pcafl"></a> 6.1.2 允许的字符和字段长度

所有通信均在UTF-8中编码,以实现国际描述的适应.
建议是IDs仅应使用以下字符:

A-Z a-z 0-9 _ - . :

最大消息长度未定义.如果AGV内存不足以处理传入的任务,那就是拒绝任务.最大字段长度,字符串长度或值范围的匹配取决于集成符.
为了易于集成,AGV供应商必须提供一个详细介绍的AGV事实说明 [section 7 - AGV Factsheet](#factsheet).



### <a name="Noe"></a> 6.1.3 枚举的符号

枚举必须以大写字母写. 
这包括关键字,例如动作状态 (WAITING, FINISHED, 等等...) 或“direction”字段的值 (LEFT, RIGHT, 443MHZ, 等等...).



### <a name="JD"></a> 6.1.4 JSON 数据类型 

在可能的情况下,必须使用JSON数据类型.
因此,布尔值由“ true / false”编码,而不是枚举(true,false)或魔术数字.



## <a name="MchsaQ"></a> 6.2 MQTT连接处理,安全性和QoS

MQTT协议提供了为客户端设置最后一个will消息的选项.
如果客户出于任何原因意外断开连接,则最后的will由broker分发给其他订阅客户.
此功能的使用在6.14中描述.

如果AGV与broker断开连接,它将保留所有任务信息并将任务完成到最后一个发布点. 

协议 - 安全需要通过broker配置考虑.

为了减少沟通开销,MQTT QoS级别0(Best Effort最佳努力)将用于 主题:`task`, `state`, `factsheet` 和 `visualization`.主题'connection'应使用QoS级别1(At Least Once至少一次).



## <a name="MTL"></a> 6.3 MQTT-Topic 级别 

由于云提供商的强制性主题结构,MQTT主题结构并未严格定义.
对于基于云的MQTT-BROKER,必须单独调整主题结构以匹配此协议中定义的主题. 
这意味着以下各节中定义的主题名称是强制性的.

对于本地broker,建议MQTT主题级别如下：

**interfaceName/majorVersion/manufacturer/serialNumber/topic**

例子: uagv/v2/KIT/0001/task



MQTT Topic Level | Data type | Description 
---|-----|-----
interfaceName | string | 接口的名称 
majorVersion | string | 主要版本编号,先于"v"
manufacturer | string | AGV制造商 (例如, huashine)
serialNumber | string | 由以下字符组成的唯一AGV序列号: <br>A-Z <br>a-z <br>0-9 <br>_ <br>. <br>: <br>-
topic | string | Topic (例如 任务或系统状态) 查看 Cap. 6.5  

Note: `/` 字符用于定义主题层次结构,不得在上述任何字段中使用它.
 `$` 字符在某些MQTT broker中也用于特殊的内部主题,因此也不应使用它.

## <a name="PH"></a> 6.4 Protocol 头

每个JSON都包含协议header.在以下各节中,以下字段将被称为可读性的header. header包括以下元素. header不是JSON对象.

Object structure/Identifier | Data type | Description 
---|---|---
headerId | uint32 | 信息头ID.<br> headerId每个topic定义并且每次发送信息自增1(但不一定收到). 
timestamp | string | 	日期时间 (ISO 8601, UTC); YYYY-MM-DDTHH:mm:ss.ssZ (例如"2017-04-15T11:40:03.12Z”)
version | string | 	协议版本 [Major].[Minor].[Patch] (例如 1.3.2)
manufacturer | string | AGV厂商
serialNumber | string | AGV序列号 

### Protocol version

协议版本使用语义版本作用作为版本架构.

主要版本更改的示例: 

- 重大变更, 例如, 新的非可选字段

次要版本更改的示例: 

- 新功能,例如可视化的附加主题 

补丁版本的示例: 

- 电池充电的可用精度更高 



## <a name="Sfc"></a> 6.5 通信Subtopics

AGV协议使用以下主题进行RCS和AGV之间的信息交换

Subtopic name | Published by | Subscribed by | Used for | Implementation | Schema 
---|---|---|---|---|---
task | RCS | AGV | RCS给AGV任务的通信 | mandatory | task.schema 
instantactions | RCS | AGV | 立即执行动作的通信  | mandatory | instantactions.schema
state | AGV | RCS | AGV state状态上报 | mandatory | state.schema
visualization | AGV | Visualization systems | 仅用于可视化目的的位置topic较高频率 | 可选 | visualization.schema
connection | Broker/AGV | RCS | 指示何时丢失AGV连接, 不是用来RCS检测车辆健康状态, 为MQTT协议级别检查连接 | mandatory | connection.schema 
factsheet | AGV | RCS | RCS中设置AGV属性 | mandatory | factsheet.schema


## <a name="TOfmctA"></a> 6.6 Topic: "task"(从 RCS 到 AGV)

主题“task”是MQTT主题,AGV接收JSON封装的任务. 



### <a name="CaL"></a> 6.6.1 概念和逻辑 

任务的基本结构是点和段的图表.AGV将通过点和片段以完成任务.所有连接点和段的完整图由RCS保存.

RCS中的图表 包含路径限制,例如 允许哪种AGV通过哪种片段.这些限制不会传达给AGV.RCS仅在AGV任务中包含允许AGV途径的段.

应该避免RCS对每种类型AGV的有单独的图表表示.只要有可能, 一个位置, 例如, 防火门前等待点, 需要对所有车仅有一个点.然而,由于AGV的大小和规格不同, 在某些情况下可能有必要偏离此标准.

![图3 RCS中的图形表示和任务中传输的图形表示](./assets/Figure3.png) 
>图 3 RCS中的图形表示和任务中传输的图形表示

这些点和段作为任务消息中的两个列表传递.列表任务还控制着 必须途径点和段的顺序.

对于有效的任务,至少必须存在一个点. 可接受的段数是点的数量-1,而不是多或少.

任务的第一点必须在AGV上可以达到. 这意味着要么AGV已经站在该点上,要么AGV在该点偏差范围内.

点和片段都具有布尔属性"released”.如果点或段"released”,则预计AGV将穿过它. 如果点或段不是"released”,则AGV不能穿过它. 

一个片段可以被released,如果片段的起点和终点都被released.

在未released的段后,没有released的点或段可以按顺序排序. 

released点和片段集合被称为"base".unreleased点和片段集合被称为"horizon".

发送一个没有"horizon"的任务也是有效的.

任务消息不一定描述完整的运输任务. 用于交通管制并容纳资源约束车辆, 完整的运输任务 (这可能包括许多要点和片段)可以分为许多子任务, 通过其OrderID和OrderUpdateID连接. 下一节中描述更新任务的过程.



### <a name="Oaou"></a> 6.6.2 任务和任务更新 

对于交通控制,task-topic仅包括通往决策点的路径.在达到决策点之前,RCS将发送带有其他路径段的更新路径.要与AGV通信达到决策点后最有可能要做的事情,一个任务由两个单独的部分组成: 

- <u>行驶到决策点 "Base":</u> "Base" 是AGV必经行驶的路线. “Base”路线的所有点和段已经被RCS批准给了此车辆. 
- <u>从决策点进行估计的路径 "Horizon":</u> "Horizon" 是AGV可能行驶的路线, 如果没有交通拥堵. "Horizon" 路线尚未获得RCS的确认. 但是,AGV最初只能前往“Base”路线的最后一个点.

由于MQTT是一种异步协议,并且通过无线网络传输不可靠,因此请注意,“base”不能更改. 因此,RCS可以假设“base”是由AGV执行的.后面的一节描述了取消任务的过程,但由于上述通信限制,这也被认为是不可靠的.

RCS有可能更改“Horizon”路线. 在AGV通过“base”路线到达决策点之前, RCS将向AGV发送更新的路径,其中包括其他点. 更改Horizon路线的过程如图4所示.

![图4更改"Horizon"行驶路线的过程 ](./assets/Figure4.png)
>图4更改"Horizon"行驶路线的过程

在图4中,RCS首先 发送初始作业 在t = 1.图5显示了可能工作的伪代码.为了可读性,这里省略了一个完整的JSON示例.

```
{
	orderId: "1234"
	orderUpdateId:0,
	points: [
	 	 6 {released: True},
	 	 4 {released: True},
	 	 7 {released: True},
	 	 2 {released: False},
	 	 8 {released: False}
	],
	segments: [
		e1 {released: True},
		e3 {released: True},
		e8 {released: False},
		e9 {released: False}
	]
}
```
>图5任务的伪代码

在t = 3, 通过发送任务的扩展来更新任务(请参见图6中的示例). 
注意"orderUpdateId" 增加并且任务更新的第一点对应于上一个任务消息base路径的最后一个点.

这样可以确保AGV也可以执行任务更新,即,通过执行AGV已知的段来达到工作更新的第一点.

```
}
	orderId: 1234,
	orderUpdateId: 1,
	points: [
		7 {released: True},
		2 {released: True},
		8 {released: True},
		9 {released: False}
	],
	segments: [
		e8 {released: True},
		e9 {released: True},
		e10 {released: False}
	]
}
```
>图6任务更新的伪代码. 请注意"orderUpdateId"改变

这也有助于orderUpdate丢失的事件(由于不可靠的无线网络).AGV始终可以检查最后一个已知的base点是否具有相同的nodeid(nodeSequenceId, 大于前者) 作为第一个新base点.

另请注意,第7点是再次发送的唯一基点.由于无法更改base,因此第6和4点的重传是无效的.

重要的是,缝合点的内容(示例中的点7)没有更改. 对于动作,偏差范围,等等. AGV必须使用第一个任务中提供的说明(图5,OrderUpdateID 0).

![图7常规更新过程 - 任务扩展](./assets/Figure7.png)
>图7常规更新过程 - 任务扩展

图7描述了如何扩展任务.它显示了当前在AGV上可用的信息.orderID保持不变,并且orderUpdateId会增加.

上一个base的最后一点是更新任务的第一个base点.在这一点上,AGV可以将更新的任务添加到当前任务(缝线)中.上一个base的其他点和段不会再下发.

RCS可以通过将完全不同的点作为新base发送给AGV,以便对horizon进行更改.horizon也可以删除.


为了允许任务中的循环(例如从点1到2,然后返回1)sequenceId分配给点和段对象.该sequenceId在点和段上运行(任务的第一个点接收到0,然后第一个段获得1,第二点然后获得2点,然后获得2,依此类推).这可以更轻松地跟踪任务进度.

一旦分配了sequenceId,就不会随着任务更新而更改(请参见图7). 这对于确定AGV到哪个点是必不可少的.

图8 描述接受任务或orderUpdate的过程.

![图8接受任务或orderUpdate的过程](./assets/Figure8.png)
>图8接受任务或orderUpdate的过程



### <a name="OCbMC"></a> 6.6.3 取消任务 (通过 RCS)

如果base点发生未计划的更改,则必须通过使用instantaction cancelOrder来取消该任务.

在接收到instantaction cancelOrder, 车辆停止(根据其功能, 例如, 停在当前位置或下一个点).

如果计划执行action,则必须取消这些操作,并且应在其action state上报告“failed”.如果有运行动作,则应取消这些动作,并报告为失败.如果动作无法中断, 该动作的动作态应报告“running”,然后在此之后(如果成功的话,“完成”和“失败”(如果没有)).在操作运行时,取消行动必须报告“运行”,直到所有操作都取消/完成. 在所有车辆移动和所有操作都停止后,取消命令动作状态必须报告“完成”.

OrderID和OrderUpDateID保留. 

图9显示了不同AGV功能的预期行为.

![图9取消订单后的预期行为](./assets/Figure9.png)
>图9取消订单后的预期行为



#### <a name="Ranoac"></a> 6.6.3.1 取消后收到新任务

取消任务后,车辆必须处于一个state才能接收新任务. 

如果AGV通过标签将自己定位在点上,则新任务必须从AGV所在的点开始(另请参见图5). 

如果AGV可以停止在点之间,则选择应取决于RCS如何启动下一个任务.AGV必须接受这两种方法. 

有两个选择：

- 发送一个任务,其中第一个点是一个临时点,位于AGV当前所在的位置. 然后,AGV必须确认这个点并接受任务. 
- 发送一个任务,其中第一个点是上一个任务的最后一个穿越点,但设置了大偏差范围,使AGV在此范围内. 因此,AGV必须意识到这一点必须被算作穿过并接受任务.


#### <a name="RacawAhno"></a> 6.6.3.2 当AGV没有任务时,接收取消订单

如果AGV收到取消订单操作,但AGV当前没有任务,或者以前的任务已取消,则取消订单操作必须按失败报告. 

AGV必须报告“noOrderToCancel”错误,并将ErrorLevel设置为警告.instantaction的动作ID必须作为errorReference传递.



### <a name="Or"></a> 6.6.4 任务拒绝

有几种情况必须拒绝任务.在图8中描述.



#### <a name="Vgamno"></a> 6.6.4.1 车辆得到了错误的新任务

解决方法： 

1.车辆不会接这个新任务. 
2.车辆报告警告"validationError"在报警字段
3.必须报告警告,直到车辆接受新任务为止.


#### <a name="Vraowaicpeglhhtmlholaansii"></a> 6.6.4.2 车辆收到一个行动无法执行的任务 (例如 提起高度高于最大提升高度或没有安装举升设备的举升动作), 或与无法使用的字段 (例如 Trajectory)

解决方法： 

1.车辆不会接这个新任务. 
2.车辆报告警告"orderError"在报警字段
3.必须报告警告,直到车辆接受新任务为止.


#### <a name="Vehiclegets"></a> 6.6.4.3 车辆获得了一项相同orderid的新任务,但orderUpdateId比当前的低

解决方法： 

1.车辆不会接这个新任务. 
2.车辆保持之前的任务
3.车辆报告警告"orderUpdateError"在报警字段
4.车辆继续之前的任务.

如果AGV两次接收具有相同订购和OrderUpdateID的任务,则将忽略第二个任务. 
如果RCS再次发送任务,这可能会发生这种情况,因为状态消息来得太晚了,RCS无法验证收到的第一个任务.



### <a name="Maps"></a> 6.6.5 Maps

为了确保不同类型的AGV之间的稳定导航,用地图坐标系指定该位置(请参见图10).对于不同级别(地图)之间的差异化,使用了唯一的mapID. 
地图坐标系应指定为右撇子坐标系,Z轴指向天空.因此,正旋转应理解为逆时针旋转.车辆坐标系也被指定为右撇子坐标系,X轴指向车辆的正方向,Z轴指向天空. 
 这与DIN ISO 8855中的第2.11章一致.

![图10带有例子AGV和方向的坐标系](./assets/Figure10.png)
>图10带有例子AGV和方向的坐标系

X,Y和Z坐标必须为米.方向必须为弧度,并且必须在 +pi和–pi内.

![图11地图和车辆的坐标系](./assets/Figure11.png)
>图11地图和车辆的坐标系


## <a name="Iotom"></a> 6.7 任务消息的实施

Object structure | Unit | Data type | Description 
---|---|---|---
headerId | | uint32 | 信息头ID.<br> headerId每个topic定义并且每次发送信息自增1(但不一定收到). 
timestamp | | string | 日期时间 (ISO 8601, UTC); YYYY-MM-DDTHH:mm:ss.ssZ (例如, "2017-04-15T11:40:03.12Z”)
version | | string | 协议版本 [Major].[Minor].[Patch] (例如, 1.3.2).
manufacturer | | string | AGV厂商. 
serialNumber | | string | AGV序列号.
Actions [action] | | array | 需要立即执行的动作组并且不是常规任务中的一部分. 
orderId |  | string | 任务标识id.<br> 这将用于识别属于同一任务的多个任务消息. 
orderUpdateId |  | uint32 | 任务更新标识id.<br>对于每个orderId是orderUpdateId唯一的.<br>如果更新任务被拒绝,则将在拒绝消息中传递此orderUpdateId
zoneSetId |  | string | 区域集的唯一标识符, AGV用于导航或RCS用于规划. <br> <br> 可选:一些RCS系统不使用区域.<br> 一些AGV不了解区域.<br> 如果没有区域使用,请勿添加到任务消息. 
**points [point]** |  | array | 任务内要途径的点node对象数组. <br>有效任务可能只有一个点. <br>该情况使用空白的片段列表. 
**segments [segment]** |  | array | 任务内要途径的片段edge对象数组. <br>有效任务可能只有一个点. <br>该情况使用空白的片段列表.

Object structure | Unit | Data type | Description
---|---|---|---
**point** { |  | JSON-object|   
nodeId |   |  string | 唯一的点标识
sequenceId |  | uint32 | 跟踪任务中的点和段的顺序并简化任务更新. <br>主要目的是区分一个点,该点在一个orderid中可能不止一次出现. <br>sequenceId在同一任务的所有点和段中运行,并在发出新的orderID时重置.
*nodeDescription* |  | string | 有关该点的其他信息 
released |  | boolean | "true" 表示该点是base的一部分. <br> "false" 表示该点是horizon的一部分. 
***nodePosition*** |  | JSON-object | 点位置. <br>可选 对于不需要点位置的车辆类型(例如, line制导车辆).
**actions [action]** <br> } |  | array | 在某个点执行的一系列动作. <br>空数组,如果不需要操作. 

Object structure | Unit | Data type | Description 
---| --- |--- | ---
**nodePosition** { |  | JSON-object | 在世界坐标系中定义地图上的位置. <br>每个楼层都有自己的地图. <br>All maps must use the same project specific global origin. 
x | m | float64 | 地图上的X位置参考地图坐标系. <br>精度取决于特定的实现. 
y | m | float64 | 地图上的Y位置参考地图坐标系. <br>精度取决于特定的实现. 
*theta* | rad | float64 | 范围: [-Pi ... Pi] <br><br>AGV的绝对方向.<br> 可选: 车辆可以自己规划路径.<br>如果定义,AGV必须在此点达到theta角度.<br>如果以前的段不允许旋转,则AGV必须在点上旋转.<br>如果接下来的段定义了不同的方向并且段禁止旋转,则AGV需要在段的起点上旋转到所需的角度.
*allowedDeviationXY* |  | float64 | 指示AGV在任务中如何认为点已经通过. <br><br> If = 0: 不允许偏差 (没有偏差意味着在AGV制造商的正常偏差范围内). <br><br> If > 0: 允许偏离(米). <br>如果AGV在经过点的时候在deviation-radius内,则认为该点已被途径.
*allowedDeviationTheta* |  | float64 | 范围: [0 ... Pi] <br><br> 指示theta角度的偏差有多大. <br>最低可接受的角度是theta-allowedDeviationTheta,最高可接受的角度是theta +allowedDeviationTheta.
mapId |  | string | 位置所在地图的唯一标识. <br> 每张地图具有相同的项目特定的全局原始坐标. <br>当AGV使用电梯时, 例如, 从一个楼层到另一个楼层,它将消失在离开楼层的地图出现在目标楼层电梯点.
*mapDescription* <br> } |  | string | 地图上的其他信息.

Object structure | Unit | Data type | Description 
---|---|---|---
**action** { |  | JSON-object | 描述AGV可以执行的动作. 
actionType |  | string |行动名称，如第一列中所述"actions and Parameters”. <br>标识动作的功能. 
actionId |  | string | 唯一ID用来识别动作,并将其映射到state中的actionstate. <br>建议: 使用 UUIDs.
*actionDescription* |  | string | 有关动作的其他信息
blockingType |  | string | Enum {NOTE, SOFT, HARD}: <br> "NONE"- 允许行驶中执行和其他动作;<br>"SOFT"- 允许其他动作，但不能在行驶中执行;<br>"HARD"-是当时唯一允许的动作.
***actionParameters [actionParameter]*** <br><br> } |  | array | 对于指定的动作的actionParameter-objects组, 例如, deviceId, loadId, external Triggers. <br><br> 查看 "actions and Parameters"

Object structure | Unit | Data type | Description 
---|---|---|---
**segment** { |  | JSON-object | 两点之间的方向连接.
edgeId |  | string | 片段的唯一标识.
sequenceId |  | Integer | 跟踪任务中的点和段的顺序并简化任务更新. <br>sequenceId在同一任务的所有点和段中运行,并在发出新的orderID时重置.
*edgeDescription* |  | string | 有关片段的其他信息.
released |  | boolean | "true"表示该片段是base的一部分.<br>"false" 表示该片段是horizon的一部分. 
startNodeId |  | string | nodeId起始.
endNodeId |  | string | nodeId终点.
*maxSpeed* | m/s | float64 | 允许在片段上的最大速度. <br>速度由车辆的最大测量定义.
*maxHeight* | m | float64 | 允许车辆(包括负载)的车辆的最大高度.
*minHeight* | m | float64 | 允许载货处理设备的最小高度.
*orientation* | rad | float64 | AGV在片段上的方向. *orientationType*的值 定义它是否必须相对于全局项目特定地图坐标系或与线段相切进行解释.在与线段相切的情况下,0.0=向前,PI=向后<br>示例：方向Pi/2 rad将导致旋转90度 <br>如果AGV以不同的方向启动,如果rotationAllowed设置为"false",则该段上的车辆将旋转到所需的方向.<br>如果rotationAllowed为“false”,则在进入段之前旋转.<br>如果不可能,则拒绝该任务.<br><br>如果未定义轨迹,则将旋转应用于线段两个连接点之间的直接路径.<br>如果这个线段定义了轨迹,则将方向应用于轨迹. 
*orientationType* |  | string | Enum {`GLOBAL`, `TANGENTIAL`}: <br>"GLOBAL"- 相对于全局特定地图坐标系;<br>"TANGENTIAL"- 切线.<br><br>如果未定义,默认值为 "TANGENTIAL".
*direction* |  | string | 在连接处设置方向,以定义line引导或线引导车辆(车辆个体).<br> 例子: left,  right, straight, 433MHz.
*rotationAllowed* |  | boolean | "true”: 允许在片段上旋转.<br>"false”: 不允许在片段上旋转.<br><br>可选:<br>如果未设置,无限制.
*maxRotationSpeed* | rad/s | float64| 最大旋转速度<br><br>可选:<br>如果未设置,无限制.
***trajectory*** |  | JSON-object | 轨迹 JSON-object for this segment as a NURBS. <br>定义曲线, AGV应在启动节点和端节之间移动.<br><br>可选:<br>如果AGV无法处理轨迹或AGV计划自己的轨迹,则可以省略.
*length* | m | float64 | 从startnode到endnode的路径长度<br><br>可选:<br>line引导AGV使用此值在达到停止位置之前降低速度. 
**action [action]**<br><br><br> } |  | array | 在该片段上执行的一系列动作. <br>空数组,如果不需要操作. <br>一个段触发的动作只能在AGV通过片段触发动作的段的时间内活跃. <br>当AGV离开片段时,该动作将停止,并且在进入片段之前将恢复状态.

Object structure | Unit | Data type | Description 
---|---|---|---
**trajectory** { |  | JSON-object |  
degree |  | float64 | 范围: [1 ... 无穷大]<br><br>定义影响曲线上任何给定点的控制点的数量. 提高增加连续性.<br><br>如果未定义,默认值为1.
**knotVector [float64]** |  | array | 范围: [ 0.0 ... 1.0]<br><br>参数值的顺序确定控制点在何处以及如何影响NURBS曲线.<br><br>knotVector的大小为控制点数量+度+1.
**controlPoints [controlPoint]**<br><br> } |  | array | JSON控制点对象的列表定义NURBS的控制点,其中包括开始点和终点.

Object structure | Unit | Data type | Description 
---|---|---|---
**controlPoint** { |  | JSON-object |  
x |  | float64 | X坐标在世界坐标系统中描述. 
y |  | float64 | Y坐标在世界坐标系统中描述.actions
*weight* |  | float64 | 范围: (0 ... 无穷)<br><br>该控制点在曲线上拉出pulls的weight .<br>如果未定义,默认值将为1.0.
} |  |  |


## <a name="actions"></a> 6.8 actions

AGV如果支持驾驶以外的其他actions,则这些actions将通过附加到点或段的action字段执行,或通过单独的主题Instantaction发送(请参阅6.9).

在段上执行的actions,仅限AGV在片段上运行时执行(请参见6.10.2).

在点上触发的actions,可以在AGV需要的时候执行.

点上的actions应该是自动终止(完成)的(例如,蜂鸣器信号持续五秒钟或取货action,在取货后自动完成)或者成对设计(例如,AcivalateWarningLights和UnctivateWarninglights),尽管有可能存在.

以下部分介绍了AGV必须使用的预定义actions,....

如果有明确定义的参数,参数必须被使用.
额外的参数也可以被定义,以便成功执行action.

如果无法将某些action映射到以下部分的actions之一,则AGV制造商可以定义RCS必须使用的其他actions.

> 校准 2022年8月31日 09:22:02

### <a name="Padtpeas"></a> 6.8.1 预定义action 定义, 参数, 效果 和 范围

general |  | scope 
:---:|--- | :---:
action, counter action, Description, idempotent, Parameter | linked state |  instant, point, segment 
action,counter action,描述,幂等,参数| 链接状态| 即时(立即),点,片段

action | counter action | Description | idempotent | Parameter | linked state | instant | point | segment
---|---|---|---|---|---|---|---|---
startPause | stopPause | 激活暂停模式. <br>连接状态是必须的,因为很多AGVs可以被硬件开关暂停. <br>AGV不继续运动 - 到下一个点不是必须的.<br>actions可以继续. <br>task是可以恢复的. | yes | - | paused | yes | no | no 
stopPause | startPause | 停用暂停模式. <br>移动和所有其他actions将恢复 (如果有的话).<br>连接状态是必须的,因为很多AGVs可以被硬件开关暂停. <br>stopPause可以恢复硬件触发的停止车辆(例如软停)(如果配置). | yes | - | paused | yes | no | no 
startCharging | stopCharging | 激活充电流程. <br>可以在充电位置进行充电 (停车状态)或者在一个charging lane (运行时). <br>防止过度充电是车辆的责任. | yes | - | .batteryState.charging | yes | yes | no
stopCharging | startCharging | 解除充电流程去接任务. <br>充电过程也可以被车辆或者充电站中断, 例如,如果电池已满. <br>电池状态仅被允许为 "false”, 当AGV准备接收任务时. | yes | - |.batteryState.charging | yes | yes | no
initPosition | - | 重新设置 (overrides) 具有给定参数的AGV的位置姿态. | yes | x  (float64)<br>y  (float64)<br>theta  (float64)<br>mapId  (string)<br>lastNodeId  (string) | .agvPosition.x<br>.agvPosition.y<br>.agvPosition.theta<br>.agvPosition.mapId<br>.lastNodeId | yes | yes<br>(Elevator) | no 
stateRequest | - | 请求AGV发送新的状态报告. | yes | - | - | yes | no | no 
logReport | - | 请求AGV生成和存储日志报告. | yes | reason<br>(string) | - | yes | no | no 
pick | drop<br><br>(如果自动化) | 请求AGV取货. <br>带有多个负载处理设备的AGV可以并行处理多个取货操作. <br>在这种情况下,需要存在参数LHD (例如. LHD1). <br>参数stationType 说明如何详细处理取货操作 (例如, 楼层位置, 货架位置, 被动输送机, 主动输送机, 等等.). <br>load type 展示load unit 并且可以用来切换field 例如 (例如, EPAL, INDU, 等等). <br>用于准备负载处理设备 (例如, 基于高度参数的提升前动作), 动作(action)可以在horizon高级项里定义. <br>注意, 提升前动作(pre-Lift)等, 不会再AGV运行中上报,因为关联点尚未释放.<br>如果车辆在一个片段上,可以使用它自己的传感器设备检测取货点的位置. | no |lhd (string, 可选)<br>stationType (string)<br>stationName(string, 可选)<br>loadType (string) <br>loadId(string, 可选)<br>height (float64) (可选)<br>定义货物底部高度related to the floor<br>depth (float64) (可选) for forklifts<br>side(string) (可选) 例如 conveyor | .load | no | yes | yes 
drop | pick<br><br>(如果自动化) | 请求AGV放货. <br>更多细节查看取货action. | no | lhd (string, 可选)<br>stationType (string, 可选)<br>stationName (string, 可选)<br>loadType (string, 可选)<br>loadId(string, 可选)<br>height (float64, 可选)<br>depth (float64, 可选) <br>… | .load | no | yes | yes
detectObject | - | AGV检测对象(例如 货物, 充电点, 自由停车位置). | yes | objectType(string, 可选) | - | no | yes | yes 
finePositioning<br>精准寻迹(上线) | - | 对于站点, AGV将精确寻迹到目标点.<br>AGV允许偏离点位置.<br>对于片段, AGV will 例如 align on stationary equipment while traversing an segment.<br>Instantaction: AGV starts positioning exactly on a target. | yes | stationType(string, 可选)<br>stationName(string, 可选) | - | no | yes | yes
waitForTrigger | - | AGV需要等待触发信号(例如按压按钮,手动装货). <br>如果需要,RCS负责处理超时和取消任务. | yes | triggerType(string) | - | no | yes | no 
cancelOrder | - | AGV应尽可能停止. <br>需要立即执行或者到下一个点. <br>然后任务删除,所有actions取消. | yes | - | - | yes | no | no 
factsheetRequest | - | 请求AGV资料单factsheet | yes | - | - | yes | no | no 



### <a name="Padtpeas1"></a> 6.8.2 预定义action 的定义和状态描述 

action | action states 
---|---
   初始, 运行, 暂停, 完成, 失败 |

action | 初始|运行|暂停|完成|失败 
---|---|---|---|---|---
startPause | - | 该模式的切换(激活)正在准备中. <br>如果AGV支持立即切换(暂停状态),这个(运行)状态可以被忽略. | - | 车辆静止不动. <br>所有actions将暂停. <br>暂停模式被激活. <br>AGV上报 .paused: true. | 某些情况下不能被激活(例如,被硬件开关控制).
stopPause | - | 该模式的切换(解除)正在准备中. <br>如果AGV支持立即切换(暂停解除状态),这个(运行)状态可以被忽略. | - | 暂停被解除. <br>所有暂停的actions将恢复继续. <br>AGV上报 .paused: false. | 某些情况下不能被激活 (例如,被硬件开关控制). 
startCharging | - | 激活充电流程正在进行中 (正在与充电桩交互中). <br>如果AGV支持立即切换(充电状态),这个(运行)状态可以被忽略. | - | 开始充电. <br>AGV上报 .batteryState.charging: true. | 某些情况下不能被激活充电 (例如, 没有对齐充电桩).充电问题应对应错误码. 
stopCharging | - | 解除充电流程正在进行中 (正在与充电桩交互中). <br>如果AGV支持立即切换(非充电状态),这个(运行)状态可以被忽略. | - | 充电流程终止. <br>AGV上报 .batteryState.charging: false | 某些情况下充电流程不能停止 (例如, 没有对齐充电桩).<br> 充电问题应对应错误码. 
initPosition | - | 新姿势的初始化 (confidence 检查 等等.). <br>如果AGV支持立即切换,这个(运行)状态可以被忽略. | - | pose重置了. <br>AGV 上报 <br>.agvPosition.x = x, <br>.agvPosition.y = y, <br>.agvPosition.theta = theta <br>.agvPosition.mapId = mapId <br>.agvPosition.lastNodeId = lastNodeId | pose无效或者不能被重置. <br>定位问题应有错误码.
stateRequest | - | - | - | state已经发送 | - 
logReport | - | 正在创建日志. <br>如果AGV支持立即切换,这个(运行)状态可以被忽略. | - | 日志已经完成记录. <br>日志名称将在上报状态内. | 日志无法保存 (例如,没有空间).
pick | 初始化取货流程, 例如, outstanding 举升操作. | 取货流程执行中 (AGV进入站点, 货物处理设备忙碌, 与站台的通信正在进行中, 等等.). | 取货流程暂停中, 例如,安全防护检测异常. <br>安全防护异常解除后, 操作继续. | 取货完成. <br>货物到位并且AGV上报新的负载状态. | 取货失败, 例如, 站台无货. <br> 取货失败应有错误码.
drop | 初始化放货流程, 例如, outstanding 举升操作. | 放货流程执行中  (AGV进入站点, 货物处理设备忙碌, 与站台的通信正在进行中, 等等.). | 放货流程执行中, 例如, ,安全防护检测异常. <br>安全防护异常解除后, 操作继续. | 放货完成. <br>货物离开并且AGV上报新的负载状态. | 放货失败, 例如, 站台被占用.  <br>放货失败应有错误码. 
detectObject | - | 目标检测运行中. | - | 目标检测到. | AGV无法检测到目标. 
finePositioning | - | AGV精确定位自己到一个目标上. | 精准寻迹暂停中, 例如,安全防护检测异常. <br>安全防护异常解除后, 寻迹继续. | 到达提供的目标位置. | 提供的目标位置无法达到. 
waitForTrigger | - | AGV正在等待触发信号 | - | 触发信号获取到. | 如果任务被取消waitForTrigger失败. 
cancelOrder | - | AGV正在停止或者运行,直到到达下个点. | - | AGV静止不动并且取消任务. | - 
factsheetRequest | - | - | - | 资料单factsheet已经传递 | - 



## <a name="Tifmc"></a> 6.9 MQTT Topic: "instantActions" (从RCS to control to AGV)

在某些情况下,需要将actions发送到AGV,并且立即执行.通过将instantAction消息发布instantActions主题来实现.instantActions不得与AGV当前任务的内容相抵触(例如:instantAction降低货叉,而任务说要抬高货叉). 

 一些立即执行动作的例子: 
  - AGV暂停,不更改当前任务中的任何内容; 
  - 暂停后恢复任务; 
  - 激活信号(灯光,蜂鸣器等). 

额外信息,请参考第8章最佳实践.

Object structure | | Data type | Description 
---|---|---|---
headerId | | uint32 | 信息头ID.<br> headerId每个topic定义并且每次发送信息自增1(但不一定收到). 
timestamp | | string | 日期时间 (ISO 8601, UTC); YYYY-MM-DDTHH:mm:ss.ssZ (例如, "2017-04-15T11:40:03.12Z”)
version | | string | 协议版本 [Major].[Minor].[Patch] (例如, 1.3.2).
manufacturer | | string | AGV厂商. 
serialNumber | | string | AGV序列号.
Actions [action] | | array | 需要立即执行的动作组并且不是常规任务重的一部分. 

当AGV收到一个立即动作,AGV状态(state)需要加入适合的actionStatus到actionStates;actionStatus依据动作的进度进行更新,查看图12,区分actionStatus的不同transitions;


## <a name="TSfAtmc"></a> 6.10 MQTT Topic: "state" (从 AGV 到 RCS)

AGV状态将仅在一个主题topic上传输.相比不同的消息 (例如, 针对 任务, 电池状态和错误码) 使用同一个topic将减少broker/RCS的工作量,同时还保持AGV状态的信息同步.AGV-State信号 和相关事件触发一起或者至少每30s 通过MQTT-Broker发布给RCS.

触发传输AGV状态消息的事件:
- 接收任务 
- 接收任务更新
- 负载状态的变化
- 错误或警告
- 运行通过一个点
- 切换操作模式
- "driving" 变化 
- nodeStates, edgeStates 或者 actionStates 变化

~~应该努力尽量减少通信次数~~.如果2个事件有关联(例如,接收新任务通常强制一个点和边的状态更新;像是运行通过一个点),应该触发一个状态更新而不是多个.


### <a name="CaLe"></a> 6.10.1 概念和逻辑

任务进度由 `nodeStates` 和 `edgeStates`跟踪. 另外,如果AGV可以获得当前位置,可以通过"position”字段发布它的位置.

如果AGV自己规划路线,必须将它的计算轨迹(包含base和horizon) 以NURBS(曲线)形式通过状态消息内的`trajectory`对象传递, 除非RCS不能使用这个字段并且在集成时候确定了,那么这个自动不能被发送.一旦点被RCS确定(released), AGV不允许改变轨迹路线.

`nodeStates` 和 `edgeStates`包含所有的点/片段,AGV仍旧必须要通过的.

![Figure 12 任务信息由state topic提供. 仅传输最后一点和剩余点和段的ID](./assets/Figure12.png) 
>Figure 12 任务信息由state topic提供. 仅传输最后一点和剩余点和段的ID

### <a name="Tonaeletoa"></a> 6.10.2 途径点和进入/离开片段,动作触发 

AGV自己判断一个点什么时候被计算为已经通过.通常,AGV控制点需要在 the node’s `deviationRangeXY` 并且方向角度在`deviationRangeTheta`.

AGV上报途径点通过从`nodeStates`数组移除`nodeState`并且设置`lastNodeId`, `lastNodeSequenceNumber`为途径点的值;

当AGV上报途径点的时候,必须触发这个点设置的Actions,如果存在的情况;

点的途径同样标志着离开指向点的片段.这个片段必须从`edgeStates`删除并且这个片段上激活的Actions必须完成;

该点的途径也标志着一个时刻, AGV进入记下来的片段,如果有一个片段的话,这个片段的Actions必须立即触发.这条规则里外的情况是,如果AGV在片段上暂停 (因为软停或者hard blocking segment,或者其他) – 然后AGV进入片段当它开始重新移动.

![Figure 13 nodeStates, edgeStates, actionStates 在任务处理过程中](./assets/Figure13.png)
>Figure 13 nodeStates, edgeStates, actionStates 在任务处理过程中



### <a name="Br"></a> 6.10.3 基础请求Base request 

If the AGV detects, that its base is running low, it can set the `newBaseRequest` flag to `true` to prevent unnecessary braking.
如果AGV检测到,它的base运行过短,可以设置`newBaseRequest`标志为`true`避免不必要的刹车.


### <a name="Information"></a> 6.10.4 Information 


AGV可以通过`information`数组提交任意的其他信息给RCS.它通过information消息传递,取决于AGV多久上报information;

RCS逻辑上不能使用这心信息消息,只能用来可视化或者debug目的.



### <a name="Errors"></a> 6.10.5 Errors 

AGV通过`errors`数组上报错误码. 错误有两种级别`WARNING` 和 `FATAL`.`WARNING`是一个可以自动解除的错误,例如,防护入侵. `FATAL`错误需要人干预.错误可以传递说明,有助于通过errorReferences组查找错误的原因.

### <a name="Implementation"></a> 6.10.6 Implementation(任务?)

Object structure | Unit | Data type | Description 
---|---|---|---
headerId | | uint32 | 消息Header ID.<br> headerId每个topic定义并且每次发送信息自增1(但不一定收到). 
timestamp | | string | 日期时间 (ISO 8601, UTC); YYYY-MM-DDTHH:mm:ss.ssZ (例如"2017-04-15T11:40:03.12Z”).
version | | string | 通讯协议 [Major].[Minor].[Patch] (例如 1.3.2).
manufacturer | | string | AGV厂商.
serialNumber | | string | AGV序列号.
orderId|  | string | 当前任务或先前完成任务的唯一任务标识. <br>orderId保持不变直到新任务. <br>空字符串 (""),如果没有有效的前orderId. 
orderUpdateId |  | uint32 | `用来识别任务更新标识,已经被AGV获取的任务更新`. <br>"0”如果没有有效的前orderUpdateId. 
*zoneSetId* |  |string |AGV当前用于路径规划的区域(不同楼或者位置区域)的唯一ID. <br>必须与任务中使用的区域id相同,否则AGV必须拒绝任务.<br><br>可选:如果AGV不使用zones,字段可以忽略.
lastNodeId |  | string | 上次到达的点id或者AGV当前点id (例如, "node7”). 如果没有有效的lastNodeId,留空字符串 ("").
lastNodeSequenceId |  | uint32 | 最后到达点的序列ID 或者AGV当前点的序列ID. <br>"0"如果没有有效的前lastNodeSequenced. 
**nodeStates [nodeState]** |  |array | nodeState-Objects数组, 需要穿过以完成任务<br>(空闲时空list)
**edgeStates [edgeState]** |  |array | edgeState-Objects数组, 需要穿过以完成任务<br>(空闲时空list)
***agvPosition*** |  | JSON-object | AGV当前在地图上的位置.<br><br>可选:<br><br>只能被无定位能力的AGV忽略, 例如, 线引导AGV(磁导航?).
***velocity*** |  | JSON-object | 车辆坐标的AGV速度. 
***loads [load]*** |  | array | 负载(多个), 当前被AGV控制的.<br><br>可选:如果AGV不能确定负载状态,不要放入state. <br>如果AGV可以确定负载状态,但是数组为空,则将AGV视为空载(无货).
driving |  | boolean | "true”: 表示,AGV在行走并且/或者在旋转.AGV其他移动(例如, 货叉移动)不包含在这里.<br><br>"false”:表示AGV既不是在行走也不在旋转.
*paused* |  | boolean | "true”: AGV是暂停,要么是物理按钮暂停要么是立即动作暂停.<br>AGV可以恢复任务.<br><br>"false”: 当前AGV不在暂停模式.
*newBaseRequest* |  | boolean | "true”: AGV几乎到达base的终点,如果没有新的base传达,将会减速. <br>触发RCS下发新base.<br><br>"false”: 不需要base更新.
*distanceSinceLastNode* | meter | float64 | line guided 车辆使用,显示行走过"lastNodeId"的距离. <br>距离是米为单位.
**actionStates [actionstate]** |  | array | 包含一组尚未完成当前动作和动作组. <br>可能包含之前的点的动作,仍旧在执行中.<br><br>当一个动作完成,一个更新的state消息发布并且设置actionStatus为完成并且如果可行包含相应结果的描述. <br><br>action state持续保持到收到新任务.
**batteryState** |  | JSON-object | 包含所有与电池相关的信息.
operatingMode |  | string | Enum {AUTOMATIC, SEMIAUTOMATIC, MANUAL,  SERVICE,  TEACHIN}<br>有关其他信息, 查看表格OperatingModes 6.10.6. 
**errors [error]** |  | array |错误对象的数组. <br>AGV的所有当前激活的错误都应在列表中.<br>一个空列表表示AGV没有当前激活的错误.
***information [info]*** |  | array | 信息对象数组. <br>一个空数组表明AGV没有信息. <br>这仅应用于可视化或调试 – 它不能在RCS中用于逻辑判断.
**safetyState** |  | JSON-object | 包含所有与安全有关的信息. 

Object structure | Unit | Data type | Description 
---|---|---|---
**nodeState** { | JSON-object |  |
nodeId |  | string | 点的唯一标识.
sequenceId |  | uint32 | sequenceId(顺序id?)用来分辨同一个nodeId的多个点.
*nodeDescription* |  | string | 有关该点的其他信息.
***nodePosition*** |  | JSON-object | 点位置. <br>对象在6.6章节中定义  <br>可选: <br>RCS有此信息. <br>可以另外发送, 例如. 用来debug.
released<br><br>}|  | boolean | "true”表示点是base的一部分.<br>"false”表示点是horizon的一部分.

Object structure | Unit | Data type | Description 
---|---|---|---
**edgeState** { |  | JSON-object |  |
edgeId |  | string | 段的唯一标识.
sequenceId |  | uint32 | sequenceId(顺序id?)用来分辨同一个edgeId的多个片段.
*edgeDescription* |  | string | 有关片段的其他信息.
released |  | boolean | "true” 表示该段是base的一部分.<br>"false” 表示该段是horizon的一部分.
***trajectory*** <br><br>} |  | JSON-object | 轨迹以NUBS曲线传递,在6.4定义<br><br>轨迹段是从AGV开始进入段的点,直到报告的下一个途径点.

Object structure | Unit | Data type | Description
---|---|---|---
**agvPosition** { |  | JSON-object | 定义世界坐标系中地图上的位置.每个楼层都有自己的地图.
positionInitialized |  | boolean | "true”: 位置正在初始化.<br>"false”: 位置没有初始化.
*localizationScore* |  | float64 | 范围: [0.0 ... 1.0]<br><br>描述定位的质量,以便可以使用, 例如 被SLAM-AGV描述, 当前位置信息的准确程度.<br><br>0.0: 位置未知<br>1.0: 已知位置<br><br>可选 对于车辆无法估计其定位质量的.<br><br>仅用于记录和可视化目的. 
*deviationRange* | m | float64 | 用米定义的位置偏差范围.<br><br>可选 对于无法估计其偏差的车辆 例如 grid-based 定位.<br><br>仅用于记录和可视化目的.
x | m | float64 | 地图上的X位置坐标 参考地图坐标系. <br>精度取决于特定的实现(任务?).
y | m | float64 | 地图上的Y位置坐标 参考地图坐标系. <br>精度取决于特定的实现(任务?).
theta |  | float64 | 范围: [-Pi ... Pi]<br><br>AGV的方向(弧度?). 
mapId |  | string | 地图位移标识id,用来定位的.<br><br>每个地图具有相同的原始坐标. <br>当AGV使用电梯时, 例如, 从一个楼层到另一个楼层,它将消失在离开楼层的地图出现在目标楼层电梯点.
*mapDescription*<br>} |  | string | 地图上的其他信息. 

Object structure | Unit | Data type | Description 
---|---|---|---
**velocity** { |  | JSON-object |  
*vx* | m/s | float64 | AVGS在其X方向上的速度.
*vy* | m/s | float64 | AVGS沿其Y方向上的速度.
*omega*<br>}| Rad/s | float64 | AVG在其Z轴上转动速度.

Object structure | Unit | Data type | Description 
---|---|---|---
**load** { |  | JSON-object |  
*loadId* |  | string | 负载的唯一标识号 (例如, 条形码 或者 RFID).<br><br>空字段,如果AGV可以识别负载,但尚未识别负载.<br><br>可选, 如果AGV无法识别负载.
*loadType* |  | string | 负载类型.
*loadPosition* |  | string | 指示使用AGV的哪个负载处理/载荷单元, 例如, 如果AGV有多个spots/位置可以载货.<br><br>例如: "前", "后", "positionC1”, 等等.<br><br>可选 对于只有一个负载位置的车辆
***boundingBoxReference*** |  | JSON-object | 货物边界 位置的参考点. <br>参考点通常是货物边界 底部表面的中心 (在height = 0) 并且 在AGV坐标系的坐标中描述.
***loadDimensions*** |  | JSON-object | 负载货物边界的尺寸(米). 
*weight*<br><br>} | kg | float64 | 范围: [0.0 ... 无穷)<br><br>测量负载的绝对重量(kg). 

Object structure | Unit | Data type | Description 
---|---|---|---
**boundingBoxReference** { |  | JSON-object | 货物边界位置的参考点. <br>参考点通常是货物边界 底部表面的中心 (在height = 0) 并且 在AGV坐标系的坐标中描述.
x |  | float64 | 参考点的x坐标. 
y |  | float64 | 参考点的y坐标.
z |  | float 64 | 参考点的z坐标. 
*theta*<br> } |  | float64 | 货物边界的方向. <br>对于输送机tugger, trains等很重要. 

Object structure | Unit | Data type | Description 
---|---|---|---
**loadDimensions** { |  | JSON-object | 货物边界框的尺寸(米). 
length | m | float64 | 货物边界框的绝对长度. 
width | m | float64 | 货物边界框的绝对宽度. 
*height* <br><br><br><br>}| m | float64 | 货物边界框的绝对高度.<br><br>可选:<br><br>仅在已知的情况下设置值.

Object structure | Unit | Data type | Description 
---|---|---|---
**actionState** { |  | JSON-object |  
actionId |  |string  | action_ID
*actionType* |  | string | 动作的动作类型.<br><br>可选: 仅出于信息或可视化目的.任务知道类型.
*actionDescription* |  | string | 有关当前动作的其他信息. 
actionStatus |  | string | Enum {WAITING; INITIALIZING; RUNNING; PAUSED; FINISHED; FAILED}<br><br>WAITING: 等待触发<br>(passing the mode, 进入片段)<br> PAUSED: 通过立即动作或外部触发暂停<br>FAILED: 无法执行动作. 
*resultDescription*<br><br><br><br>} |  | string | 结果的描述, 例如, RFID-读取数据.<br><br>错误将在错误消息中传达.<br><br>results例子在6.5

Object structure | Unit | Data type | Description 
---|---|---|---
**batteryState** { |  | JSON-object |  
batteryCharge | % | float64 | 充电状态: <br> 如果AGV仅提供好或差电池水平的值,则将这些值表示为20％(差)和80％(好). 
*batteryVoltage* | V | float64 | 电池电压.
*batteryHealth* | % | int8 | 范围: [0 .. 100]<br><br>健康状况. 
charging |  | boolean | "true”: 正在充电中.<br>"false”: AGV目前不充电.
*reach* <br><br>}| m | uint32 | 范围: [0 ... 无穷)<br><br>估算当前充电状态. 

Object structure | Unit | Data type | Description 
---|---|---|---
**error** { |  | JSON-object |  
errorType |  | string | Type/name of error 
***errorReferences [errorReference]*** |  | array | 参考列表 用来识别错误的来源 (例如, headerId, orderId, actionId, 等等.).<br>有关其他信息,请参见"Best practices" 第8章.
*errorDescription* |  | string | 错误说明. 
errorLevel <br><br> }|  | string | Enum {WARNING, FATAL}<br><br>WARNING: AGV准备开始 (例如 维护保养周期到期警告).<br>FATAL: AGV不处于运行状态,需要用户干预 (例如 激光扫描仪脏了).

<a name="errorReferenceImpl"></a>
Object structure | Unit | Data type | Description 
---|---|---|---
**errorReference** { |  | JSON-object |  
referenceKey |  | string | 参考类型References the type of reference (例如, headerId, orderId, actionId, 等等.).
referenceValue <br>} |  | string | 参考值.

Object structure | Unit | Data type | Description 
---|---|---|--- 
**info** { |  | JSON-object |  
infoType |  | string | Type/name of information. 
*infoReferences [infoReference]* |  | array | 参考数组. 
*infoDescription* |  | string | Info说明. 
infoLevel <br><br><br>}|  | string | Enum {DEBUG,INFO}<br><br>DEBUG: 用来调试.<br> INFO: 用于可视化. 

Object structure | Unit | Data type | Description 
---|---|---|---
**infoReference** { |  | JSON-object |  
referenceKey |  | string | 参考类型References the type of reference(例如, headerId, orderId, actionId, 等等.).
referenceValue <br>} |  | string | 参考值.


Object structure | Unit | Data type | Description 
---|---|---|---
**safetyState** { |  | JSON-object |  
eStop |  | string | Enum {AUTOACK,MANUAL,REMOTE,NONE}<br><br>Acknowledge-Type of eStop:<br>AUTOACK: auto-acknowledgeable e-stop激活, 例如, 通过防撞条或安全防护区.<br>MANUAL: e-stop 将在车辆上手动承认.<br>REMOTE: facility e-stop has to be acknowledged remotely.<br>NONE: 无e-stop激活.
fieldViolation<br><br>} |  | boolean | 防护区域侵犯.<br>"true":区域被侵犯<br>"false":区域没有被侵犯.

#### Operating Mode Description
以下说明列出了"states"中的操作模式.

Identifier | Description 
---|---
AUTOMATIC | AGV完全控制RCS. <br>AGV根据RCS的任务运行和执行操作.
SEMIAUTOMATIC | AGV被RCS控制.<br> AGV根据RCS的任务运行和执行操作. <br>行驶速度由HMI控制 (速度不能超过自动模式的速度).<br>转向在自动控制下 (非安全的HMI).
MANUAL | RCS不控制AGV. <br>管理者不会将驾驶任务或动作发送到AGV. <br>HMI可用于控制AGV的转向和速度和处理设备. <br>AGV的位置发送给RCS. <br>当AGV进入或离开此模式时,立即清除所有任务 (安全HMI需要).
SERVICE | RCS不控制AGV. <br>管理者不会将驾驶任务或动作发送到AGV. <br>授权个人可以重新配置AGV. 
TEACHIN | RCS不控制AGV. <br>管理者不会将驾驶任务或动作发送到AGV. <br>正在训练AGV, 例如,录入地图?mapping is done by a RCS.



## <a name="actionStates"></a> 6.11 actionStates

当AGV接收到`action` (不论是关联在`point`或者`segment`或者通过`instantaction`), 必须将`action`反馈在`actionstate` 在`actionStates`组里.

`actionStates` 在字段`actionStatus`中描述了动作生命周期的哪个阶段.

表 1 描述, 哪个actionStatus枚举可以保持. 

actionStatus | Description 
---|---
WAITING | AGV接收到动作,但是AGV没有到达触发的点或者没有进入激活的片段.
INITIALIZING | 动作触发, 启动准备措施.
RUNNING | 动作正在运行.
PAUSED | 由于立即动作或外部触发(AGV上的暂停按钮),该动作被暂停 
FINISHED | 动作完成了. <br>结果通过resultDescription报道
FAILED | 无论出于何种原因,都无法完成动作.

>表 1 actionStatus 字段的可接受值

图14中提供了状态过渡图.

![图 14 actionStates所有可能的过渡状态](./assets/Figure14.png)
>图 14 actionStates所有可能的过渡状态



## <a name="ABTas"></a> 6.12 动作阻塞类型和顺序Action Blocking Types and Sequence

包含多个动作的任务 在需要执行的动作列表中定义顺序. 
动作的并行执行被它们的`blockingType`管理.

动作可以具有三种不同的阻塞类型, 表2中描述. 

actionStatus | Description 
---|---
NONE | 动作可以与其他动作并行执行,可以在车辆行驶时执行.
SOFT | 动作可以与其他动作并行执行,不能在车辆行驶中执行.
HARD | 不得与其他动作并行执行操作,不能在车辆行驶中执行.

>表2动作阻塞类型

如果在同一点上有多个动作,不同的阻塞类型,图15描述了AGV应如何处理这些动作.

![图15处理多个动作](./assets/Figure15.png)
>图15处理多个动作



## <a name="TV"></a> 6.13 MQTT Topic "visualization" 

对于近乎实时的位置,AGV可以广播其位置和速度通过消息 `visualization`.

位置消息的结构 与状态中的位置和速度消息 结构相同,有关其他信息,请参见第6.7章实施,该消息主题的更新速率由集成商控制.



## <a name="Tc"></a> 6.14 MQTT Topic "connection"

在将AGV客户端连接到broker时,可以设置最后一个will主题和消息,该主题和消息是由broker与AGV client断开连接后发布的. 
因此,RCS可以通过订阅所有AGV的连接主题来检测断开事件. 
通过broker和客户端之间通过心跳检测到断开连接. 
该间隔在大多数brokers中都是可配置的,应设置约15秒钟. 
"connection"主题的服务质量至少应为1-至少一次.

建议的最后一个主题结构是:

**uagv/v2/manufacturer/SN/connection**

最后一条will消息定义为带有以下字段的JSON封装消息:

Identifier | Data type | Description 
---|---|---
headerId |uint32 | 消息Header ID.<br> headerId每个topic定义并且每次发送信息自增1(但不一定收到). 
timestamp | string | 日期时间 (ISO 8601, UTC); YYYY-MM-DDTHH:mm:ss.ssZ (例如"2017-04-15T11:40:03.12Z”).
version | string | 通讯协议 [Major].[Minor].[Patch] (例如 1.3.2).
manufacturer | string | AGV厂商.
serialNumber | string | AGV序列号.
connectionState | string | Enum {`ONLINE`, `OFFLINE`, `CONNECTIONBROKEN`}<br><br>`ONLINE`:AGV和broker连接着<br><br>`OFFLINE`: AGV与broker的连接以一种协调的方式离线. <br><br> `CONNECTIONBROKEN`: AGV与broker的连接以外结束. 

当连接以优雅的方式结束(使用MQTT断开连接命令以优雅的方式结束时),将不会发送最后一条will消息. 
如果连接意外中断,则仅由broker发送最后一条will消息.

**注意**: 由于MQTT中最后一个will功能的性质, 最后一条will消息 是在AGV和MQTT broker之间连接阶段 定义的.
结果,时间戳和headerld字段将始终过期.

AGV希望优雅地断开连接: 

1. AGV发送 "uagv/v2/manufacturer/SN/connection" 包含`connectionState`设置`OFFLINE`.
2. 用断开命令断开MQTT连接.

AGV上线: 

1. 将最后一个will设置为 "uagv/v2/manufacturer/SN/connection" 包含字段`connectionState`且设置`CONNECTIONBROKEN`, 当创建MQTT连接时.
2. 发送主题 "uagv/v2/manufacturer/SN/connection" 包含 `connectionState` 设置`ONLINE`.

该主题上的所有消息均应带有保留标志.

当AGV和broker之间的连接意外停止时, broker将发送最后一个will主题: "uagv/v2/manufacturer/SN/connection" 包含字段`connectionState`,且设置 `CONNECTIONBROKEN`.

## <a name="Tf"></a> 6.16 MATT Topic "factsheet"

FactSheet提供了有关特定AGV类型系列的基本信息.
该信息允许比较不同的AGV类型,可以应用于AGV系统的规划,尺寸和仿真.FactSheet还包括有关将AGV类型系列集成到一个AGV VDA-5050-compliant RCS通信接口的信息.

AGV FactSheet中某些字段的值只能在系统集成期间指定,例如,特定于项目的负载和站台类型的分配,以及该AGV支持的站点和负载类型列表.

factsheet既旨在作为人类可读文档,又用于机器处理, 例如, RCS应用程序导入, 因此被指定为JSON文档.

RCS可以通过发送立即动作`factsheetRequest` 从AGV请求事实表.

该主题上的所有消息均应带有保留标志.

### 6.16.1 Factsheet JSON strcture
The factsheet consists of the JSON-objects listed in the following table.

| **Field**                  | **data type** | **description**                                              |
| -------------------------- | ------------- | ------------------------------------------------------------ |
| headerId                   | uint32        | 消息Header ID.<br> headerId每个topic定义并且每次发送信息自增1(但不一定收到). |
| timestamp                  | string        | 日期时间 (ISO8601, UTC); YYYY-MM-DDTHH:mm:ss.ssZ(例如"2017-04-15T11:40:03.12Z”). |
| version                    | string        | 协议版本 [Major].[Minor].[Patch] (例如 1.3.2). |
| manufacturer               | string        | AGV厂商.                                     |
| serialNumber               | string        | AGV序列号.                                     |
| **typeSpecification**      | JSON-object   | 这些参数通常指定AGV的类和功能. |
| **physicalParameters**     | JSON-object   | 这些参数指定AGV的基本物理属性. |
| **protocolLimits**         | JSON-object   | 标识符,数组,字符串和类似的 在MQTT通信中的限制. |
| **protocolFeatures**       | JSON-object   | 支持的功能 of VDA5050 protocol.                      |
| **agvGeometry**            | JSON-object   | AGV几何学的详细定义.                         |
| **loadSpecification**      | JSON-object   | 负载功能的抽象规范.                 |
| **localizationParameters** | JSON-object   | 定位的详细规范.                      |

#### typeSpecification

This JSON object 描述AGV类型的一般特性.

| **Field**           | **data type**   | **description** |
|---------------------|-----------------|-----------------|
| seriesName          | string          | 制造商指定的通用系列名称.任意文本 |
| *seriesDescription* | string          | AGV类型系列的可读描述.任意文本    |
| agvKinematic        | string          | 简化的AGV运动学类型的描述.<br/> [DIFF, OMNI, THREEWHEEL]<br/>DIFF: 差速器驱动器<br/>OMNI: 全向车辆<br/>THREEWHEEL: 三轮驱动的车辆 或者 具有类似运动学的车辆 |
| agvClass            | string          | 简化的AGV类描述.<br/>[FORKLIFT, CONVEYOR, TUGGER, CARRIER]<br/>FORKLIFT: 叉车.<br/>CONVEYOR: 带有输送机的AGV.</br>TUGGER: 拖车.<br/>CARRIER: 货物 carrier 有或者没有顶升机构. |
| maxLoadMass         | float64         | [kg], 最大负荷质量. |
| localizationTypes   | Array of String  | 定位类型的简化描述.<br/>示例值:<br/>NATURAL: 自然地标;<br/>REFLECTOR: 激光反射器;<br/>RFID: RFID-tags;<br/>DMC: data matrix code;<br/>SPOT: magnetic spots;<br/>GRID: magnetic grid.<br/>
| navigationTypes     | Array of String | AGV支持的路径规划类型列表,按优先级排序.<br/>示例值:<br/>PHYSICAL_LINE_GUIDED: 没有路径规划, AGV跟踪物理安装的路径.<br/>VIRTUAL_LINE_GUIDED: AGV行走固定(虚拟)路径.<br/>AUTONOMOUS: AGV自动规划其路径.|

#### physicalParameters

This JSON-object describes physical properties of the AGV.

| **Field**       | **data type** | **description**                                       |
|-----------------|---------------|-------------------------------------------------------|
| speedMin        | float64       | [m/s] AGV的最小控制连续速度.  |
| speedMax        | float64       | [m/s] AGV的最大速度.                        |
| accelerationMax | float64       | [m/s²] 最大负载下的最大加速度.         |
| decelerationMax | float64       | [m/s²] 最大负载下的最大减速.         |
| heightMin       | float64       | [m] AGV的最小高度.                             |
| heightMax       | float64       | [m] AGV的最大高度.                             |
| width           | float64       | [m] AGV的宽度.                                      |
| length          | float64       | [m] AGV的长度.                                     |

#### protocolLimits

This JSON-object describes the protocol limitations of the AGV.
If a parameter is not defined or set to zero then there is no explicit limit for this parameter.

| **Field**                     | **data type** | **description**                             |
|-------------------------------|---------------|---------------------------------------------|
| **maxStringLens** {           | JSON-object   | 字符串的最大长度.                 |
| &emsp;*msgLen*                      | uint32        | 最大MQTT消息长度                 |
| &emsp;*topicSerialLen*              | uint32        | MQTT-Topics中serial-number的最大长度.<br/><br/>受影响的参数:<br/>task.serialNumber<br/>instantactions.serialNumber<br/>state.SerialNumber<br/>visualization.serialNumber<br/>connection.serialNumber   |
| &emsp;*topicElemLen*                | uint32        | MQTT-Topics中所有其他部分的最大长度.<br/><br/>受影响的参数:<br/>task.timestamp<br/>task.version<br/>task.manufacturer<br/>instantactions.timestamp<br/>instantactions.version<br/>instantactions.manufacturer<br/>state.timestamp<br/>state.version<br/>state.manufacturer<br/>visualization.timestamp<br/>visualization.version<br/>visualization.manufacturer<br/>connection.timestamp<br/>connection.version<br/>connection.manufacturer |
| &emsp;*idLen*                       | uint32        | ID-Strings的最大长度.<br/><br/>受影响的参数:<br/>task.orderId<br/>task.zoneSetId<br/>point.nodeId<br/>nodePosition.mapId<br/>action.actionId<br/>segment.edgeId<br/>segment.startNodeId<br/>segment.endNodeId |
| &emsp;*idNumericalOnly*             | boolean          | If "true" ID-strings 需要仅包含数字. |
| &emsp;*enumLen*                     | uint32        | 最大长度ENUM- and Key-Strings.<br/><br/>受影响的参数:<br/>action.actionType action.blockingType<br/>segment.direction<br/>actionParameter.key<br/>state.operatingMode<br/>load.loadPosition<br/>load.loadType<br/>actionstate.actionStatus<br/>error.errorType<br/>error.errorLevel<br/>errorReference.referenceKey<br/>info.infoType<br/>info.infoLevel<br/>safetyState.eStop<br/>connection.connectionState                                               |
| &emsp;*loadIdLen*                   | uint32        | 最大长度loadId Strings |
| }                             |               |                                  |
| **maxArrayLens** {            | JSON-object   | 最大数组长度.                                 |
| &emsp;*task.points*                 | uint32        | AGV每个任务可处理的的最大点数.  |
| &emsp;*task.segments*                 | uint32        | AGV每个任务可处理的的最大片段数.  |
| &emsp;*point.actions*                | uint32        | AGV每个点可处理的的最大actions数. |
| &emsp;*segment.actions*                | uint32        | AGV每个片段可处理的的最大actions数. |
| &emsp;*actions.actionsParameters*   | uint32        | AGV每个操作可处理的最大参数个数. |
| &emsp;*instantactions*              | uint32        | 每条消息AGV可以处理的最大立即动作数量. |
| &emsp;*trajectory.knotVector*       | uint32        | AGV可处理的每个轨迹的最大结数knots. |
| &emsp;*trajectory.controlPoints*    | uint32        | AGV可处理的每个轨迹的最大控制点数. |
| &emsp;*state.nodeStates*            | uint32        | AGV发送的最大nodeStates数,AGV base中的最大点数. |
| &emsp;*state.edgeStates*            | uint32        | AGV发送的最大edgeStates数,AGV base中的最大片段数. |
| &emsp;*state.loads*                 | uint32        | AGV发送的最大负载对象load-objects数.                |
| &emsp;*state.actionStates*          | uint32        | AGV发送的最大actionStates数.                |
| &emsp;*state.errors*                | uint32        | AGV在一个state消息中发送的最大错误数量. |
| &emsp;*state.informations*          | uint32        | AGV在一个state消息中发送的最大informations数量.    |
| &emsp;*error.errorReferences*       | uint32        | AGV发送的每个错误的最大错误参考errorReferences数量.      |
| &emsp;*informations.infoReferences* | uint32        | AGV发送的每个信息的最大信息参考infoReferences数量. |
| }                             |               |                                                                        |
| **timing** {                  | JSON-object   | Timing information.                                            |
| &emsp;minOrderInterval              | float32       | [s], 将任务消息发送到AGV最小间隔时间.        |
| &emsp;minStateInterval              | float32       | [s], 发送state-messages的最小间隔.               |
| &emsp;*defaultStateInterval*        | float32       | [s], 发送state-messages的默认间隔, *如果未定义,则使用主文档的默认值*. |
|  &emsp;*visualizationInterval*      | float32       | [s], 用于发送可视化主题消息的默认间隔.       |
| }                             |               |                                                               |

#### agvProtocolFeatures

该JSON对象定义了由AGV支持的 操作action和参数parameter.

| **Field**    | **data type** | **description**  |
|--------------|---------------|------------------|
| **optionalParameters** [**optionalParameter**] | JSON-object数组 | 支持和/或必须的 可选参数列表.<br/>此处未列出的可选参数,假定AGV不支持这些参数. |
| {            |               |                  |
| &emsp;parameter    | string        | 可选参数的全名, 例如 "*task.points.nodePosition.allowedDeviationTheta”*.|
| &emsp;support      | enum      | 可选参数支持的类型, 以下值是可能的:<br/>SUPPORTED: 可选参数像指定的一样支持.<br/>REQUIRED: 可选 适当的AGV操作需要参数. |
| &emsp;*description*| string        | 任意形式文字: 可选参数描述, 例如:<ul><li>原因, 为什么此AGV类型需要可选参数‘direction’及其可以包含的值.</li><li>参数 ‘nodeMarker’ 必须只包含unsigned int值.</li><li>NURBS-Support 仅限于直线和圆形片段.</li>|
| }            |               |                  |
| **agvactions** [**agvaction**] | JSON-object数组 | 这个AGV支持所有带参数的动作列表. 这包括VDA5050中指定的标准操作和制造商的特定动作. |
| {            |               |                  |
| &emsp;actionType   | string        | 唯一actionType 与action.actionType一致. |
| &emsp;*actionDescription* | string  | 任意形式文字: action描述. |
| &emsp;actionscopes | array of enum  | 使用此允许的范围列表action-type.<br/><br/>INSTANT: 可用作为立即.<br/>point: 可在点上使用.<br/>segment: 可在片段上使用.<br/><br/>例如: ```["INSTANT", "NODE"]```|
| &emsp;***actionParameters** [**actionParameter**]* | JSON-object数组 | 参数列表<br/>如果未定义,则该动作没有参数 |
|&emsp;*{*     |               |                  |
|&emsp;&emsp;key     | string        | Key-String for 参数. |
|&emsp;&emsp;valueDataType | enum    | 数据类型, 可能的数据类型是: BOOL, NUMBER, INTEGER, FLOAT, STRING, OBJECT, ARRAY. |
|&emsp;&emsp;*description* | string  | 任意形式文字: 参数描述. |
|&emsp;&emsp;*isOptional*  | boolean    | "true": 可选 参数. |
|&emsp;*}*           |         |                          |
|*resultDescription* | string  | 任意形式文字: resultDescription描述. |
|*}*                 |         |                          |

### agvGeometry

此JSON对象定义了AGV的结构特性, 例如, 轮廓和轮子位置.

| **Field**                            | **data type**        | **description**                                        |
|--------------------------------------|----------------------|--------------------------------------------------------|
| ***wheelDefinitions** [**wheelDefinition**]* | JSON-object数组 | List of wheels, containing wheel-arrangement and geometry. |
| {                                    |                      |                                                        |
| &emsp;type                                 | enum                 | 车轮类型<br/>```DRIVE, CASTER, FIXED, MECANUM```.     |
| &emsp;isActiveDriven                       | boolean                 | "true": 主动驱动车轮.       |
| &emsp;isActiveSteered                      | boolean                 | "true": 车轮主动转向.    |
| &emsp;**position** {                           | JSON-object          |                                                        |
|&emsp;&emsp; x                              | float64              | [m], x位置在AGV坐标中. system          |
|&emsp;&emsp; y                              | float64              | [m], y位置在AGV坐标中. system          |
|&emsp;&emsp; *theta*                        | float64              | [rad], 固定车轮所需的AGV坐标系统中的车轮方向. |
| &emsp;}                                    |                      |                                                        |
| &emsp;diameter                             | float64              | [m], 车轮直径.                          |
| &emsp;width                                | float64              | [m], 车轮宽度.                             |
| &emsp;*centerDisplacement*                 | float64              | [m], 车轮中心到旋转点的位移 (caster wheels必要).<br/> 如果未定义参数,则假定为0.            |
| &emsp;*constraints*                        | string               | 任意形式文字: 制造商可以用来定义约束. |
| }                                    |                      |                                                        |
| ***envelopes2d** [**envelope2d**]*   | JSON-object数组  | 2D下的AGV-包络曲线列表(german: "Hüllkurven"), 例如, 机械包络在载货和不载货状态, 不同速度的安全防护. |
| {                                    |                      |                                                        |
| &emsp;set                             | string               | 包络曲线集合数量.                         |
| &emsp;**polygonPoints**  **[polygonPoint]**         | JSON-object数组  | 包络曲线 X/Y-Polygon多边形预期是封闭的,必须是非自交的. |
| &emsp;{                                    |                      |                                                        |
|&emsp;&emsp; x                              | float64              | [m], 多边形点的x位置.                        |
|&emsp;&emsp; y                              | float64              | [m], 多边形点的y位置.                        |
| &emsp;}                                    |                      |                                                        |
| &emsp;*description*                        | string               | 任意形式文字: 包络曲线集合的描述.   |
| *}*                                  |                      |                                                        |
| ***envelopes3d [envelope3d]***       | JSON-object数组  | 3D下的AGV-包络曲线列表. |
| *{*                                  |                      |                                                        |
| &emsp;set                                  | string               | 包络曲线集合数量.                         |
| &emsp;format                               | string               | 数据格式, 例如, DXF.                                |
| &emsp;***data***                           | JSON-object          | 3D-包络曲线数据, 格式以'format'指定.   |
| &emsp;*url*                                | string               | 下载3D-包络曲线数据的协议和URL定义, 例如 <ftp://xxx.yyy.com/ac4dgvhoif5tghji>. |
| &emsp;*description*                        | string               | 任意形式文字: 包络曲线集合的描述          |
| *}*                                  |                      |                                                        |

#### loadSpecification

此JSON对象定义 AGV的负载处理和负载类型.

| **Field**                        | **data type**        | **description**                                                      |
|----------------------------------|----------------------|----------------------------------------------------------------------|
| *loadPositions*         | Array of String      | 负载位置清单 / 负载处理设备.<br/>这列表包含"state.loads[].loadPosition”参数的有效值 和 对于action取货和放货的'lhd'参数.<br/>*如果此列表不存在或为空,则AGV没有负载处理设备.* |
| ***loadSets [loadSet]*** | JSON-object数组  | 可以由AGV处理的load-sets列表                     |
| {                                    |                      |                                                        |
|&emsp; setName                 | string               | load-set的唯一名称, 例如, DEFAULT, SET1, 等等.                 |
|&emsp; loadType                | string               | 负载类型, 例如, EPAL, XLT1200, 等等.                                  |
|&emsp; *loadPositions*         | Array of String      | 负载位置列表btw.负载处理设备,此load-set有效.<br/>*如果此参数不存在或为空,则此load-set对此AGV上的所有负载处理设备有效.* |
|&emsp; ***boundingBoxReference***  | JSON-object          | Bounding box reference在state-message中的loads[]中定义. |
|&emsp; ***loadDimensions***        | JSON-object          | Load dimensions在state-message中的loads[]中定义.     |
|&emsp; *maxWeight*             | float64              | [kg], 负载类型的最大重量.                                      |
|&emsp; *minLoadhandlingHeight* | float64              | [m], 最小允许的处理高度 针对load-type 和 –weight<br/>参考boundingBoxReference. |
|&emsp;  *maxLoadhandlingHeight* | float64              | [m], 最大允许的处理高度 针对load-type 和 –weight<br/>参考boundingBoxReference. |
|&emsp; *minLoadhandlingDepth*  | float64              | [m], 最小允许的深度 针对load-type and –weight<br/>参考boundingBoxReference. |
|&emsp; *maxLoadhandlingDepth*  | float64              | [m], 最大允许的深度 针对load-type 和 –weight<br/>参考boundingBoxReference. |
|&emsp; *minLoadhandlingTilt*   | float64              | [rad], 最低允许的倾斜度 针对load-type 和 –weight.            |
|&emsp; *maxLoadhandlingTilt*   | float64              | [rad], 最大允许的倾斜 针对load-type 和 –weight.            |
|&emsp; *agvSpeedLimit*         | float64              | [m/s], 最大允许速度 针对load-type 和 –weight.           |
|&emsp; *agvAccelerationLimit*  | float64              | [m/s²], 最大允许加速 针对load-type 和 –weight.   |
|&emsp; *agvDecelerationLimit*  | float64              | [m/s²], 最大允许减速 针对load-type 和 –weight.   |
|&emsp; *pickTime*              | float64              | [s], 大约. 取货时间                  |
|&emsp; *dropTime*              | float64              | [s], 大约. 放货时间                    |
|&emsp; *description*           | string               | 任意形式文字: 负载处理集合 的描述.            |
| }                       |                      |                                                           |


# <a name="Bp"></a> 7 Best practice

本节包括其他信息,有助于促进与协议逻辑同时发生的共同理解.


## <a name="Er"></a> 7.1 Error reference 

如果由于erroneous任务而发生错误, AGV需要返回一个有含义的错误在errorReference字段 (参考 [6.10.6 Implementation](#errorReferenceImpl)).
这可以包括以下信息:

- headerId
- Topic (任务 或者 立即动作)
- orderId 和 orderUpdateId如果错误是因为任务更新导致
- actionId如果错误是因为一个动作导致
- 参数列表如果是因为erroneous动作参数导致 

如果由于外部因素无法完成操作 (例如 预期位置没有负载),actionId需要referenced.



## <a name="Fop"></a> 7.2 Format of parameters 

错误,信息,操作(errors, information, action)的参数被设计为带有键值对 的JSON-Objects数组.
动作参数的"someaction”带有键值对 stationType和loadType的例子:

``
"actionParameters":[
{"key":"stationType", "value": "floor"},
{"key": "loadType", "value": "pallet_eu"}
]
``

使用这个方案的原因 "key”: "actualKey”, "value”: "actualValue” 是保持实施通用.这是在多次会议中进行了彻底而有争议的讨论.



# <a name="Glossary"></a> 8 名词Glossary 



## <a name="Definition"></a> 8.1 定义

Concept | Description 
---|---
自由导航AGV | 使用地图规划自己道路的车辆. <br> RCS仅发送启动和目标坐标.<br>车辆将其路径发送到RCS.<br>当与RCS的通信断开时, 车辆能够继续路径.<br>自由导航车辆可以被允许绕过障碍.<br>也有可能对车辆本身对接收/分配位置进行精细调整.
引导车辆 (物理或虚拟) | 车辆从RCS得到路径. <br>在RCS中进行路径的计算.<br>当与RCS的通信断开后,该车辆终止其released的点和片段 (the "base") 并且停止.<br>可以允许引导车辆绕过障碍.<br>也有可能对车辆本身对接收/分配位置进行精细调整..
中央地图 | 将在RCS中心保存的地图.<br>创建然后使用.<br> 未来的接口版本 可以将此地图转移到车辆 (例如, 针对自由导航).
