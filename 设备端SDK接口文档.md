
©2017  **云智易**物联云平台（http://www.xlink.cn）


# 硬件SDK

## 1.硬件SDK介绍

云智易物联硬件SDK主要为传统厂商的产品接入互联网，并能通过APP对产品进行监测、控制和统计分析等。

### 1.1主要特点
* 本地设备自动探测和扫描联网；
* 内置云端连接服务，无需额外对接开发；
* 支持内、外网网络自动检测和切换；
* 提供云端固件升级和远程配置、诊断和管理服务；
* 支持设备安全认证和数据加密机制；
* 纯C代码，支持ARM、x86、x64以及MIPS等不同处理器架构；
* 可移植性强，使用操作简单快捷。

### 1.2Xlink SDK结构

结构图如下：

![](https://raw.githubusercontent.com/xlink-corp/device-sdk/master/docs/images/SDK结构.bmp)

### 1.3术语
* 设备：指使用xlink SDK的硬件；
* 服务器：指云平台，设备所要连接的服务器；
* 外网：指设备通过互联网连接到服务器的通道；
* 内网：指设备通过本地udp通讯与APP之间连接的通道。

## 2.使用方法

###2.1使用步骤

* 首先在平台[http://admin.xlink.cn](http://admin.xlink.cn)上注册一个账号，并创建一个产品，这时可获得产品PID和产品KEY；
* 在平台上所创建的产品上的“设备管理”页面上添加接入设备的MAC地址；
* 然后转到接入设备开发，先选择对应模组SDK；
* 设备初始化TCP并连接上云智易TCP服务器，初始化UDP；
* 调用“xlink_sdk_start()”启动SDK；
* 循环执行“xlink_sdk_process()”；
* 如果使用外网，需等待“EVENT_TYPE_ONLINE”事件触发后才能向服务器发送数据；
* 设备连接平台成功后，便可收发数据，如果需要使用到数据端点，请参考“数据端点文档”；
* 需要停止时，调用“xlink_sdk_stop()”停止SDK。

## 3.接口说明

### 3.1启动SDK

```
xlink_int32 xlink_sdk_start(struct xlink_sdk_cfg_t* sdk_cfg, xlink_int8 cloud_enable, xlink_int8 local_enable);
```

#### 功能：

启动SDK函数，在调用其他说有的SDK接口时，必须先要调用此接口初始化。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | sdk参数配置结构体，用户需要定义一个全局变量并设置好参数|
| cloud_enable | 0：不使用外网，1：使用外网，其他保留|
| local_enable | 0：不使用内网，1：使用内网，其他保留|

注：cloud_enable与local_enable至少要有一个为1。

#### 返回值：

| 值 | 说明 |
| --- | --- |
| 0 | 成功| 
| 1 | 失败| 

### 3.2停止SDK

```
xlink_int32 xlink_sdk_stop(struct xlink_sdk_cfg_t* sdk_cfg, xlink_int8 cloud_enable, xlink_int8 local_enable);
```

#### 功能：

停止SDK函数，如果当外网网络断开或设备离线时候需要停止SDK外网，然后断开tcp连接后再重新连接上服务器后启用SDK外网。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | 为调用“xlink_sdk_start()”时的结构体参数 |
| cloud_enable | 0：不停止外网，1：停止外网，其他保留|
| local_enable | 0：不停止内网，1：停止内网，其他保留|

#### 返回值：

| 值 | 说明 |
| --- | --- |
| 0 | 成功| 
| 1 | 失败| 

### 3.3 SDK后台执行函数

```
extern void xlink_sdk_process(struct xlink_sdk_cfg_t* sdk_cfg, xlink_int32 time_s);
```

#### 功能：

SDK后台执行函数，用户需要一直循环调用。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | 为调用“xlink_sdk_start()”时的结构体参数 |
| time_s | 系统tick时间，单位毫秒 |

#### 返回值：
| 值 | 说明 |
| --- | --- |
| none | --- |


### 3.4重置SDK

```
xlink_int32 xlink_sdk_reset(struct xlink_sdk_cfg_t* sdk_cfg);
```

#### 功能：

重置SDK，将SDK所保持的到设备flash的数据清空，下次启动SDK时候设备需要重新激活。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | 为调用“xlink_sdk_start()”时的结构体参数 |

#### 返回值：

| 值 | 说明 |
| --- | --- |
| 0 | 成功| 
| 1 | 失败| 

### 3.5 SDK写入Flash数据

```
xlink_int32 xlink_write_flash_cb(struct xlink_sdk_cfg_t* sdk_cfg, xlink_int8* data, xlink_int32 datalength);
```

#### 功能：

此函数需要用户去实现内容，写入需要保持SDK数据到flash。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | 为调用“xlink_sdk_start()”时的结构体参数 |
| *data | 写入数据指针 |
| datalength | 写入数据指针长度 |

#### 返回值：

| 值 | 说明 |
| --- | --- |
| xlink_int32 | 返回实际长度，失败返回-1 |

### 3.6 SDK读取Flash数据

```
xlink_int32 xlink_read_flash_cb(struct xlink_sdk_cfg_t* sdk_cfg, xlink_int8* data, xlink_int32 datalength);
```

#### 功能：

此函数需要用户去实现内容，读取保存到flash的SDK数据。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | 为调用“xlink_sdk_start()”时的结构体参数 |
| *data | 存放数据指针 |
| datalength | 存放数据指针可用空间 |

#### 返回值：

| 值 | 说明 |
| --- | --- |
| xlink_int32 | 返回实际长度，失败返回-1 |

### 3.7发送数据到外网

```
xlink_int32 xlink_send_to_cloud_cb(struct xlink_sdk_cfg_t* sdk_cfg, xlink_int8* data, xlink_int32 datalength);
```

#### 功能：

此函数需要用户去实现内容，将数据内容发送到外网，如通过tcp发送数据给服务器。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | 为调用“xlink_sdk_start()”时的结构体参数 |
| *data | 数据指针 |
| datalength | 数据指针长度 |

#### 返回值：

| 值 | 说明 |
| --- | --- |
| xlink_int32 | 返回实际长度，失败返回-1 |

### 3.8发送数据到内网

```
xlink_int32 xlink_send_to_local_cb(struct xlink_sdk_cfg_t* sdk_cfg, xlink_int8* data, xlink_int32 datalength);
```

#### 功能：

此函数需要用户去实现内容，将数据内容发送到内网，如通过udp发送数据给APP。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | 为调用“xlink_sdk_start()”时的结构体参数 |
| *data | 数据指针 |
| datalength | 数据指针长度 |

#### 返回值：

| 值 | 说明 |
| --- | --- |
| xlink_int32 | 返回实际长度，失败返回-1 |

### 3.9获取数据端点

```
xlink_int32 xlink_get_datapoint_cb(struct xlink_sdk_cfg_t* sdk_cfg, xlink_int8* data, xlink_int32 datalength);
```

#### 功能：

此函数需要用户去实现内容，通常是服务器或者APP向设备获取数据端点时被调用，用户将设备所有的数据内容存放到data。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | 为调用“xlink_sdk_start()”时的结构体参数 |
| *data | 存放数据指针 |
| datalength | 存放数据指针可用空间 |

#### 返回值：

| 值 | 说明 |
| --- | --- |
| xlink_int32 | 返回实际长度 |

### 3.10设置数据端点

```
xlink_int32 xlink_set_datapoint_cb(struct xlink_sdk_cfg_t* sdk_cfg, xlink_int8* data, xlink_int32 datalength);
```

#### 功能：

此函数需要用户去实现内容，通常是服务器或者APP向设备发送数据端点时被调用。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | 为调用“xlink_sdk_start()”时的结构体参数 |
| *data | 数据端点内容 |
| datalength | 数据端点内容长度 |

#### 返回值：

| 值 | 说明 |
| --- | --- |
| xlink_int32 | 返回处理实际长度 |

### 3.11 SDK事件通知

```
void xlink_event_cb(struct xlink_sdk_cfg_t* sdk_cfg, struct xlink_sdk_event_t* event_t);
```

#### 功能：

此函数需要用户去实现内容，当SDK事件被触发时会进入此函数，用户判断事件内容来处理对应的事务。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | 为调用“xlink_sdk_start()”时的结构体参数 |
| *event_t | 事件内容，具体参考“xlink_sdk_event_t”定义 |

#### 返回值：

| 值 | 说明 |
| --- | --- |
| xlink_int32 | 返回处理实际长度 |

### 3.12 SDK消息推送

```
void xlink_notify_cb(struct xlink_sdk_cfg_t* sdk_cfg, struct xlink_sdk_notify_t* notify_t);
```

#### 功能：

此函数需要用户去实现内容，当SDK收到APP或者服务器推送的消息会进入此函数，用户可以读取到相应的消息内容。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | 为调用“xlink_sdk_start()”时的结构体参数 |
| *notify_t | 消息内容，具体参考“xlink_sdk_notify_t”定义 |

#### 返回值：

| 值 | 说明 |
| --- | --- |
| xlink_int32 | 返回处理实际长度 |

###3.13发送数据端点数据到服务器

```
xlink_int32 xlink_update_cloud_datapoint(struct xlink_sdk_cfg_t* sdk_cfg, xlink_int32 handle, xlink_int8* data, xlink_int32 datalength);
```

#### 功能：

通过外网发送数据端点数据到服务器。

当服务收到后会通过“xlink_event_cb”事件来通知，通知类型为“EVENT_TYPE_REQUEST_CB”。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | 为调用“xlink_sdk_start()”时的结构体参数 |
| handle | 句柄|
| *data | 发送的数据|
| datalen | 数据长度（SDK单次发送数据数据不能超过900字节）|

#### 返回值：

| 值 | 说明 |
| --- | --- |
| 0 | 转发成功| 
| 1 | 转发失败| 

###3.14发送数据端点数据到APP

```
xlink_int32 xlink_update_local_datapoint(struct xlink_sdk_cfg_t* sdk_cfg, xlink_int32 handle, xlink_int8* data, xlink_int32 datalength);
```

#### 功能：

通过内网发送数据端点数据到APP。

当服务收到后会通过“xlink_event_cb”事件来通知，通知类型为“EVENT_TYPE_REQUEST_CB”。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | 为调用“xlink_sdk_start()”时的结构体参数 |
| handle | 句柄|
| *data | 发送的数据|
| datalen | 数据长度（SDK单次发送数据数据不能超过900字节）|

#### 返回值：

| 值 | 说明 |
| --- | --- |
| 0 | 转发成功| 
| 1 | 转发失败| 

### 3.15向服务器请求数据

```
xlink_int32 xlink_request_event(struct xlink_sdk_cfg_t* sdk_cfg, xlink_int32 handle, struct xlink_sdk_event_t* event_t);
```

#### 功能：

通过外网请求服务器数据。

当服务收到后会通过“xlink_event_cb”事件来通知，通知类型为对应的请求类型。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | 为调用“xlink_sdk_start()”时的结构体参数 |
| handle | 句柄|
| *event_t | 请求的事件类型|

#### 返回值：

| 值 | 说明 |
| --- | --- |
| 0 | 转发成功| 
| 1 | 转发失败| 

### 3.16设备存入收到的外网数据

```
void xlink_push_cloud_data(struct xlink_sdk_cfg_t* sdk_cfg, xlink_int8* data, xlink_int32 datalength);

```

#### 功能：

当设备收到外网数据，即服务器发来的数据，通过此接口存放到SDK中。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | 为调用“xlink_sdk_start()”时的结构体参数 |
| *data | 存入的数据|
| datalen | 数据长度|

#### 返回值：

| 值 | 说明 |
| --- | --- |
| 0 | 转发成功| 
| 1 | 转发失败| 

### 3.17设备存入收到的内网数据

```
void xlink_push_local_data(struct xlink_sdk_cfg_t* sdk_cfg, xlink_int8* data, xlink_int32 datalength);

```

#### 功能：

当设备收到内网数据，即APP发来的数据，通过此接口存放到SDK中。

#### 参数：

| 参数 | 说明 |
| --- | --- |
| *sdk_cfg | 为调用“xlink_sdk_start()”时的结构体参数 |
| *data | 存入的数据|
| datalen | 数据长度|

#### 返回值：

| 值 | 说明 |
| --- | --- |
| 0 | 转发成功| 
| 1 | 转发失败| 

## 4.结构体定义

### 4.1 `SDK参数配置：xlink_sdk_cfg_t`

```
typedef struct xlink_sdk_cfg_t{
    xlink_int8* dev_pid;
    xlink_int8* dev_pkey;
    xlink_int8 dev_name[XLINK_DEV_NAME_MAX + 1];
    xlink_uint8 dev_mac[XLINK_DEV_MAC_LENGTH_MAX];
    xlink_int8 dev_mac_length;
    xlink_int16 dev_firmware_version;
    xlink_uint8 sys_para[XLINK_SYS_PARA_LENGTH];
} xlink_sdk_cfg_t;
```

####描述

| 成员 | 说明 |
| --- | --- |
| *dev_pid | 产品ID |
| *dev_pkey | 产品KEY|
| dev_name | 设备名称，最长为XLINK_DEV_NAME_MAX字节|
| dev_mac | 设备MAC地址|
| dev_mac_length | 设备MAC长度|
| dev_firmware_version | 固件版本|
| sys_para | 系统数据，用户不用关心|

### 4.2 `设备连接状态：xlink_connected_station_t`

```
typedef struct xlink_connect_station_t{
    xlink_int8 state;
} xlink_connect_station_t;
```

####描述

| 成员 | 说明 |
| --- | --- |
| state | 连接服务器状态，0：断开连接，1：连接 |

### 4.3 `日期事件：xlink_datetime_t`

```
typedef struct xlink_datetime_t{
    xlink_uint16 year;
    xlink_uint8  month;
    xlink_uint8  day;
    xlink_uint8  week;
    xlink_uint8  hour;
    xlink_uint8  min;
    xlink_uint8  second;
    xlink_int16  zone;
} xlink_datetime_t;
```

####描述

| 成员 | 说明 |
| --- | --- |
| year | 年 |
| month | 月|
| day | 日|
| week | 周|
| hour | 时|
| min | 分|
| second | 秒|
| zone | 时区|

### 4.4 `升级：xlink_upgrade_t`

```
typedef struct xlink_upgrade_t{
    xlink_int32 device_id;
    xlink_int8  flag;
    xlink_uint16 firmware_version;
    xlink_uint32 file_size;
    xlink_int8* down_url;
    xlink_int16 down_url_length;
    xlink_int8* check_md5;
    xlink_int16 check_md5_length;
} xlink_upgrade_t;
```

####描述

| 成员 | 说明 |
| --- | --- |
| device_id | 设备ID |
| flag | bit7:wifi升级，1有效，bit6:保留，bit5:0位非强制升级，1位强制升级，其他保留|
| firmware_version | 固件新版本|
| file_size | 文件大小|
| *down_url | 文件下载链接|
| down_url_length | 文件下载链接长度|
| *check_md5 | md5校验码|
| check_md5_length | md5校验码长度|

### 4.5 `升级完成：xlink_upgrade_complete_t`

```
typedef struct xlink_upgrade_complete_t{
    xlink_int8  flag;
    xlink_uint16 current_version;
    xlink_uint16 new_version;
} xlink_upgrade_complete_t;
```

####描述

| 成员 | 说明 |
| --- | --- |
| flag | bit7:wifi升级，1有效，其他保留 |
| current_version | 当前版本|
| new_version | 升级后新的固件版本|

### 4.6 `请求响应：xlink_request_cb_t`

```
typedef struct xlink_request_cb_t{
    xlink_int32  handle;
    xlink_uint32 value;
} xlink_request_cb_t;
```

####描述

| 成员 | 说明 |
| --- | --- |
| handle | 对应发送的句柄 |
| value | 0：成功，1：失败，其他保留|

### 4.7 `SDK事件：xlink_sdk_event_t`

```
typedef struct xlink_sdk_event_t{
    typedef xlink_enum_event_type_t enum_event_type_t;
    typedef xlink_union {
        struct xlink_connect_station_t connect_station_t;
        struct xlink_datetime_t datetime_t;
        struct xlink_upgrade_t upgrade_t;
        struct xlink_upgrade_complete_t upgrade_complete_t;
        struct xlink_request_cb_t request_cb_t;
    }event_struct_t;
} xlink_sdk_event_t;
```

####描述

| 成员 | 说明 |
| --- | --- |
| enum_event_type_t | 事件类型 |
| event_struct_t | 事件数据 |

### 4.8 `SDK消息：xlink_sdk_notify_t`

```
typedef struct xlink_sdk_notify_t{
    xlink_uint8 from_type;
    xlink_int32 from_id;
    xlink_int8* msg_value;
    xlink_int16 msg_value_length;
} xlink_sdk_notify_t;
```

####描述

| 成员 | 说明 |
| --- | --- |
| from_type | 消息来自哪个设备，1:server,2:devcie,3:app,其他保留 |
| from_id | 设备ID |
| *msg_value | 消息内容 |
| msg_value_length | 消息内容长度 |

## 5.枚举定义

### 5.1 `SDK 事件类型：xlink_enum_event_type_t`

```
typedef xlink_enum xlink_enum_event_type_t{
    EVENT_TYPE_CONNECT_STATION = 0,
    EVENT_TYPE_REQ_DATETIME,
    EVENT_TYPE_REQ_DATETIME_CB,
    EVENT_TYPE_UPGRADE_CB,
    EVENT_TYPE_UPGRADE_COMPLETE,
    EVENT_TYPE_REQUEST_CB
} xlink_enum_event_type_t;
```

####描述

| 成员 | 说明 |
| --- | --- |
| EVENT_TYPE_CONNECT_STATION | 设备连接服务器的连接状态事件类型，对应结构体xlink_connect_station_t |
| EVENT_TYPE_REQ_DATETIME | 设备请求获取服务器时间事件类型，事件内容为NULL |
| EVENT_TYPE_REQ_DATETIME_CB | 设备请求服务器时间返回事件类型，对应结构图为xlink_datetime_t |
| EVENT_TYPE_UPGRADE_CB | 设备收到服务器推送的升级事件类型，对应结构图为xlink_upgrade_t |
| EVENT_TYPE_UPGRADE_COMPLETE | 设备升级完成后上报给服务器事件类型，对应结构体为xlink_upgrade_complete_t |
| EVENT_TYPE_REQUEST_CB | 设备发送数据后收到的应答事件类型，对应结构体为xlink_request_cb_t |

©2017  **云智易**物联云平台（http://www.xlink.cn）
