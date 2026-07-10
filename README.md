# unirtos_network_demos

本仓库推荐通过 unirtos-cli 的 demo 工作流使用，以保证创建、环境拉取和编译流程一致。

## 功能描述

本 Demo 展示 UniRTOS 网络注册与数据拨号（Datacall）的基础开发流程，适合作为蜂窝网络应用的入门样例。

- 演示网络附着、信号状态获取与小区信息查询
- 提供 Datacall 建链、IP 获取与掉线重连示例
- 便于扩展 APN 配置、业务连接管理与异常恢复逻辑

## 快速上手

### 1. 安装 UniRTOS 工具链

- [开发准备](https://www.quectel.com.cn/unirtos/docs?docs_page=快速上手/开发准备/开发准备.html)
- [安装交叉编译工具链](https://www.quectel.com.cn/unirtos/docs?docs_page=快速上手/环境搭建/环境搭建.html)
- [安装 Python3](https://www.python.org/downloads/)
- [安装 git](https://git-scm.com)
- 安装 `unirtos-cli`：`pip install unirtos-cli`

以上工具安装完成后，确认以下命令可用：

```bash
python --version # Python3
git --version
unirtos --version # 1.0.5 及以上版本
unirtos-cli version # 1.0.11 及以上版本
```

### 2. 使用 unirtos-cli 拉取 demo

先查看可用 demo 与版本：

```bash
unirtos-cli ls-demos
```

创建本 demo 工程：

```bash
unirtos-cli new -r unirtos_network_demos
```

如需指定版本：

```bash
unirtos-cli new -r unirtos_network_demos -v 1.0.0
```

### 3. 进入工程并编译

```bash
cd unirtos_network_demos-1.0.0
unirtos-cli env-setup
unirtos-cli build
```

## 常用命令

```bash
# 打开 SDK 菜单配置
unirtos-cli menuconfig

# 清理构建产物
unirtos-cli clean
```

## 代码概览

#### 项目结构

```
unirtos_network_demos/
├── CMakeLists.txt          # CMake 构建配置
├── datacall_demo.c         # Datacall 拨号与重连示例
├── datacall_demo.h         # Datacall 示例头文件
├── env_config.json         # 外部依赖与环境配置
├── network_demo.c          # 网络注册与信号信息示例
├── network_demo.h          # 网络示例头文件
├── menuconfig/             # SDK 菜单配置目录
└── README.md               # 本文件
```

#### 示例工作流程

```
程序启动
	↓
调用 unir_network_demo_init()
	↓
创建名为 "QNWDEMO" 的任务
	↓
进入任务主函数 nw_demo_task()
	↓
注册网络事件回调 network_event_cb()
	↓
检查 SIM 状态、频段配置并等待附着
	↓
附着成功后周期输出 RAT、运营商、小区与信号信息

并行的 Datacall 示例流程：
调用 unir_datacall_demo_init()
	↓
创建名为 "QDATACALLDEMO" 的任务
	↓
进入 datacall_demo_task()
	↓
等待附着 -> 配置 PDP Context(APN/IP 类型)
	↓
执行 qosa_datacall_start() 建立数据连接
	↓
获取并输出 IPv4/IPv6 地址
	↓
接收网络去激活事件后自动重拨恢复连接
```

#### 主要 API 接口

##### unir_network_demo_init
网络 demo 初始化函数
- 创建网络任务 `QNWDEMO`
- 设置任务栈大小和优先级
- 启动网络状态监控主流程

##### nw_demo_task
网络任务处理函数
- 注册 PS 注册状态、信号质量、RRC、NITZ、NAS 事件回调
- 检查 SIM 状态、支持频段与当前频段配置
- 等待网络附着，失败时结合 reject 原因、锁频配置与小区信息排查
- 附着成功后周期获取运营商、小区和信号参数，并输出 CSQ 示例值

##### network_event_cb
网络事件回调函数
- 处理 PS 注册状态变化事件
- 处理信号质量变化事件
- 处理 RRC 连接状态变化事件
- 处理 NITZ 时间信息与 NAS reject 事件

##### unir_datacall_demo_init
Datacall demo 初始化函数
- 创建 Datacall 任务 `QDATACALLDEMO`
- 启动拨号建链与重连流程

##### datacall_demo_task
Datacall 任务处理函数
- 等待网络附着
- 配置 PDP Context（APN、IP 类型）
- 创建 datacall 连接并执行同步拨号
- 获取并打印 IP 信息
- 处理去激活事件并进行重试重连

#### 日志展示

初始化与运行阶段可看到类似输出：

```
[I/DEMO] create sem result=0
[I/DEMO] ret=0,status=1
[I/DEMO] ret=0x0,ps_status=1
[I/DEMO] registration success
[I/DEMO] current_rat:4
[I/DEMO] serving cell info output
[I/DEMO] rssi=-7000, rsrp=-9500, rsrq=-900, sinr=1200
[I/DEMO] datacall status=1
[I/DEMO] ipv4 addr:10.x.x.x
```

当发生网络去激活事件时，Datacall 示例会输出重试日志并自动重拨恢复。

## 配置说明

默认关键配置位于 `network_demo.c` 与 `datacall_demo.c`：

- `NW_DEMO_WAIT_ATTACH_MAX_WAIT_TIME`：网络附着最大等待时间，默认 300 秒
- `NW_DEMO_EXCUTE_CFUN_MAXTIME`：附着失败时 CFUN 恢复最大执行次数，默认 10 次
- `DATACALL_DEMO_WAIT_ATTACH_MAX_WAIT_TIME`：Datacall 前附着最大等待时间，默认 300 秒
- `DATACALL_DEMO_WAIT_DATACALL_MAX_WAIT_TIME`：Datacall 建链最大等待时间，默认 120 秒
- `apn_str`：默认 APN 为 `test`，请按运营商要求调整
- `profile_idx`：默认 PDP profile 为 1

不同平台或运营商配置可能存在差异，建议按实际网络环境调整 APN、PDP 类型、重试间隔与阈值参数。

## 技术社区

技术社区：https://forumschinese.quectel.com/c/66-category/66

## 贡献指南

欢迎参与共建，建议按以下方式提交：
- 提交前先执行一次基础验证：env-setup、build、clean。
- 使用清晰的提交说明，描述改动目的、影响范围和验证结果。
- 新增功能或行为变化时，同步更新 README 与相关文档。
- 通过 Issue 或 Pull Request 提交问题修复与功能改进。
