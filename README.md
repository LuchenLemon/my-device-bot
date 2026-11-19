# My Device Bot

## Personal Cross-Device Automation and Intelligent Control System

**基于 Python、FastAPI、Docker 与公开通信协议的个人跨设备自动化控制系统**

**Project Status:** Planning and Early Development  
**Primary Language:** Python 3.11+  
**Development Environment:** macOS / Linux / Docker  
**Project Type:** Open-Source Experimental Project  
**AI Development Tools:** Devin, AI-assisted Programming Tools

---

## 1. 项目简介（Project Overview）

My Device Bot 是一个面向个人设备管理、跨设备通信和自动化控制的实验性软件项目。

本项目计划构建一个运行于 Linux 服务器或 NAS 环境中的轻量级自动化控制中枢，通过统一的设备接口、任务调度服务和通信适配器，使不同类型的设备能够在同一套软件系统中接受管理。

传统的设备控制系统通常依赖各自独立的软件、网络协议和管理界面。当用户同时使用网络设备、智能家居设备、舞台灯光设备或其他自动化硬件时，不同控制接口之间的兼容性问题会增加系统维护成本。

本项目希望研究如何通过标准化的软件接口管理这些设备，并利用自动化任务减少重复操作。

项目将采用模块化设计，允许开发者为不同设备编写独立的通信适配器，而不需要大幅修改核心服务。

在开发过程中，将尝试使用 Devin 等 AI 编程工具辅助完成代码编写、测试、错误排查以及部署配置。

### 1.1 核心目标

本项目的主要研究和开发目标包括：

1. 构建统一的跨设备控制接口。
2. 实现基于 FastAPI 的设备管理服务。
3. 探索 OSC、Art-Net 等公开通信协议的使用方法。
4. 建立可扩展的设备适配器机制。
5. 支持自动化任务调度与执行状态记录。
6. 实现基础权限校验、错误处理与操作日志。
7. 通过 Docker 简化服务部署和环境管理。
8. 使用模拟设备验证控制指令的正确性。
9. 研究 AI 编程工具辅助开发复杂软件项目的可行性。

### 1.2 当前开发状态

项目目前处于规划和初期开发阶段。

本仓库首先用于记录系统设计、公开技术研究、开发任务和后续实验结果。

本文档中的系统架构、接口和功能均为拟采用的设计方案。

在相应代码、测试和运行记录正式提交之前，不应将这些规划视为已经实现的功能。

---

## 2. 系统架构（System Architecture）

### 2.1 总体设计

系统计划采用分层的软件架构，将用户交互、接口服务、权限控制、任务管理和设备通信分别处理。

主要组成部分如下：

**用户交互层（User Interaction Layer）**

负责接收来自 Web 界面、Bot 或其他授权客户端的控制请求。

**API 服务层（API Service Layer）**

基于 FastAPI 提供统一的 HTTP API，并计划使用 WebSocket 实现实时状态通知。

**权限管理层（Authorization Layer）**

负责验证用户身份与设备操作权限，防止未经授权的控制请求被执行。

**任务调度层（Task Scheduling Layer）**

负责管理设备控制任务、执行顺序、超时设置及任务状态。

**设备适配层（Device Adapter Layer）**

通过统一接口封装不同设备的通信逻辑，使核心业务服务不必直接处理每种设备的协议细节。

**设备通信层（Communication Layer）**

负责与模拟设备或通过公开协议接入的真实硬件进行通信。

### 2.2 计划中的控制流程

用户通过 Web API 或 Bot 发送设备控制请求。

请求首先进入 FastAPI 服务，由权限模块判断当前用户是否具有执行该操作的权限。

通过验证后，任务调度服务根据目标设备选择对应的 DeviceAdapter。

适配器将统一格式的操作命令转换为目标设备使用的通信数据，并将发送结果和可获取的设备状态返回给上层服务。

基本流程：

User / Bot / Web Client

↓

FastAPI API Service

↓

Authentication & Authorization

↓

Command Scheduler

↓

DeviceAdapter Interface

↓

Protocol Adapter

↓

Device / Simulator

系统还将通过日志服务记录控制请求和执行过程，便于后续排查问题。

---

## 3. 技术选型（Technology Stack）

本项目计划采用以下公开技术：

| 技术 | 计划用途 |
|---|---|
| Python 3.11+ | 核心业务逻辑与通信程序 |
| FastAPI | HTTP API 服务 |
| Pydantic | 数据验证与请求模型 |
| WebSocket | 实时状态通知 |
| SQLite | 本地设备信息、任务与日志记录 |
| SQLAlchemy | 数据库访问 |
| Docker | 服务容器化 |
| Docker Compose | 服务部署与管理 |
| Python asyncio | 异步任务与通信 |
| OSC | 网络控制指令通信实验 |
| Art-Net | 舞台灯光网络控制实验 |
| DMX512 | 灯光控制协议研究 |
| Pytest | 自动化测试 |
| Git / GitHub | 版本管理与公开开发记录 |
| Devin | AI 辅助开发与测试 |

以上为初步选型，具体依赖版本将在实现和测试后确定。

---

## 4. 核心功能设计（Core Features）

### 4.1 统一设备管理

系统计划提供统一的设备注册、管理与查询服务。

每个设备将分配独立标识，并通过设备类型选择相应的通信适配器。

计划记录以下基础属性：

- 设备唯一标识
- 设备名称及类型
- 设备通信协议
- 适配器名称
- 连接参数
- 最近一次通信时间
- 当前已知状态
- 支持的控制操作
- 错误信息与运行记录

设备配置将通过独立配置文件或数据库管理，避免在代码中直接写入设备连接信息。

### 4.2 DeviceAdapter 设备适配器

DeviceAdapter 是本项目计划实现的核心扩展接口。

其主要作用是隔离设备通信协议与上层业务逻辑。

计划定义以下方法：

| 接口方法 | 功能 |
|---|---|
| `connect()` | 初始化设备连接 |
| `disconnect()` | 关闭连接并释放资源 |
| `send_command()` | 发送控制命令 |
| `get_status()` | 获取可用状态信息 |
| `validate_command()` | 检查命令参数是否合法 |
| `handle_error()` | 处理通信或执行异常 |

不同协议的适配器将采用统一的数据结构返回执行结果。

**第一阶段实施方案：**

首先实现 `MockDeviceAdapter`，模拟设备连接、状态变化与控制命令。

随后在模拟环境中验证接口稳定性，再根据实际需求扩展 OSC 和 Art-Net 适配器。

### 4.3 自动化任务调度

系统计划支持以下任务类型：

- 单次控制任务
- 定时控制任务
- 条件触发任务
- 多设备顺序执行任务
- 可取消的等待中任务

每个任务将具有独立任务编号及状态。

初步状态包括：

| 状态 | 含义 |
|---|---|
| Pending | 等待执行 |
| Running | 正在执行 |
| Succeeded | 已确认成功 |
| Failed | 执行失败 |
| TimedOut | 执行超时 |
| Cancelled | 已取消 |
| Unknown | 结果无法确认 |

对于无法提供执行确认的通信协议，系统应区分“命令已发送”和“设备已执行”。

不能仅根据数据包发送成功就认定真实硬件完成了对应操作。

### 4.4 实时监控与日志

系统计划建立基础事件记录机制，用于记录：

1. 用户提交控制命令的时间。
2. 命令对应的目标设备。
3. 发起操作的用户身份。
4. 权限检查结果。
5. 调度服务的处理结果。
6. 通信适配器的返回信息。
7. 发生的异常与错误类型。

WebSocket 将用于向授权客户端推送任务状态变化。

日志中不应直接保存用户密码、访问令牌或其他敏感凭据。

---

## 5. OSC 与 Art-Net 通信实验

### 5.1 OSC 通信

Open Sound Control（OSC）是一种用于在计算机及多媒体设备之间传递控制信息的开放通信协议。

本项目计划研究 OSC 在设备控制系统中的应用。

初步实验将使用 Python 实现 OSC 消息接收，并将收到的控制消息转换为内部统一命令。

例如，规划使用以下 OSC 地址：

`/stage/light/1/brightness`

该地址用于表示对一号灯光亮度的控制请求。

系统接收命令后，需要验证目标设备、操作权限及参数范围，再决定是否交由相应的通信适配器执行。

OSC 地址和内部控制命令之间的对应关系将通过配置管理。

### 5.2 Art-Net 通信

Art-Net 是一种通过以太网传输舞台灯光控制数据的协议。

在公开实验环境中，本项目计划实现基础 Art-Net 数据发送功能，并使用模拟接收端验证数据格式。

第一阶段重点研究：

- Art-Net 数据包结构
- Universe 地址配置
- DMX 通道映射
- 通道值范围检查
- UDP 数据发送
- 模拟接收端的数据验证

控制参数应受到严格限制，避免产生超出协议允许范围的数值。

### 5.3 DMX512 控制

DMX512 是舞台灯光设备常用的数字控制协议。

本项目不计划直接通过普通计算机网络接口产生 DMX512 电气信号。

在需要连接真实灯光设备时，计划采用符合要求的 Art-Net/DMX 硬件网关完成协议转换。

初步控制流程：

OSC Client

↓

OSC Receiver

↓

Command Validation

↓

Art-Net Adapter

↓

Art-Net/DMX Gateway

↓

DMX512 Device

在实际硬件型号确认前，所有测试均以软件模拟环境为主。

### 5.4 通信限制

需要注意，Art-Net 数据发送成功不等于灯具已经正确执行。

部分设备和协议无法直接返回实际执行状态。

因此，系统将区分以下信息：

- 服务是否收到控制请求
- 控制请求是否通过验证
- 数据包是否成功提交发送
- 是否存在独立设备状态反馈
- 实际控制结果是否经过验证

这一设计用于避免系统报告未经证实的执行结果。

---

## 6. 异常处理与恢复机制

跨设备通信可能受到网络连接、设备响应时间和协议限制的影响。

因此，本项目计划针对常见异常设计独立的处理流程。

### 6.1 通信超时

当设备在指定时间内未能返回预期结果时，记录超时事件。

对于具有明确响应机制的设备，可以进一步检查设备状态。

对于无法确认执行结果的单向控制协议，不应自动假设命令未执行。

### 6.2 设备离线

设备离线检测将根据其实际通信能力分别实现。

支持状态查询的设备可以通过周期性检查判断连接情况。

不支持状态反馈的设备只能记录通信服务自身的运行状态，不应直接宣称远端设备在线。

### 6.3 自动重连

对于支持重新建立连接的协议，计划实现有限次数的重连操作。

重连过程将设置最大尝试次数与等待时间，避免持续产生无效连接请求。

### 6.4 重复命令处理

系统将为控制请求设置独立标识，用于追踪请求状态。

对于存在重复执行风险的操作，优先检查之前的执行结果，不能确认时应转入人工检查或安全处理流程。

### 6.5 安全停止

在真实设备控制场景中，安全停止策略必须根据设备实际行为制定。

对于灯光设备，需要明确断开连接、控制服务崩溃或网关失联时的输出行为。

这些行为需要通过真实硬件测试验证，不能仅依赖软件中的默认设置。

---

## 7. 安全设计（Security Design）

本项目将安全性作为基础设计要求。

### 7.1 身份认证

计划为管理 API 添加身份认证机制，限制未经授权的客户端访问。

### 7.2 权限管理

根据用户角色及设备类别，限制可执行操作。

例如，普通用户可以查询设备状态，但不能直接修改系统配置。

### 7.3 输入验证

所有外部输入必须进行类型、范围和格式检查。

对于设备地址、控制通道和命令参数，需要建立明确的有效范围。

### 7.4 网络访问

控制服务默认不直接暴露于公网。

实际部署时优先使用可信局域网及必要的访问控制措施。

### 7.5 凭据管理

项目不会将密码、密钥和访问令牌提交到公开仓库。

本地配置通过环境变量或其他适当方式管理。

公开仓库仅提供不包含真实凭据的示例配置文件。

---

## 8. API 设计（Planned API）

第一阶段计划提供以下 HTTP API。

| Method | Endpoint | Description |
|---|---|---|
| GET | `/health` | 查询服务运行状态 |
| GET | `/api/devices` | 获取设备列表 |
| GET | `/api/devices/{id}` | 获取指定设备信息 |
| GET | `/api/devices/{id}/status` | 查询设备已知状态 |
| POST | `/api/devices/{id}/commands` | 提交设备控制命令 |
| GET | `/api/tasks` | 查询任务列表 |
| GET | `/api/tasks/{id}` | 查询任务执行状态 |
| POST | `/api/tasks/{id}/cancel` | 请求取消可取消的任务 |

以上接口为计划设计，尚不代表已经实现或可以访问。

具体请求结构、错误代码及权限要求将在代码提交后通过 OpenAPI 文档说明。

---

## 9. Docker 部署方案

### 9.1 部署目标

计划将应用部署在支持 Docker 的 Linux 主机或 NAS 上。

基础部署包括：

- FastAPI 应用服务
- 本地 SQLite 数据文件
- 设备适配器组件
- 配置文件
- 日志存储目录

### 9.2 容器化要求

开发过程中将逐步实现：

1. 创建应用 Dockerfile。
2. 配置 Python 依赖。
3. 编写 Docker Compose 文件。
4. 使用环境变量管理基础配置。
5. 配置数据库持久化。
6. 添加服务健康检查。
7. 验证容器重新启动后的数据保留情况。

### 9.3 网络与设备接入

OSC 和 Art-Net 实验可能涉及 UDP 通信。

容器网络配置将根据实际部署环境选择，并经过独立测试。

不会假设不同操作系统中的 Docker 网络行为完全相同。

### 9.4 部署说明

当前尚未提供正式部署命令。

完成第一阶段可运行版本后，将补充实际验证过的安装、启动、停止和故障排查说明。

---

## 10. 使用 Devin 进行辅助开发

### 10.1 开发目标

本项目计划尝试通过 Devin 辅助完成明确的软件开发任务，而不是让 AI 在缺少测试条件的情况下直接生成整个系统。

每个开发任务都需要具有清楚的输入、工作范围与验收要求。

### 10.2 第一阶段 Devin 任务

**Task: Implement a Minimal Cross-Device Automation Backend**

任务目标：

基于公开的 Python 开发工具，创建一个能够运行和测试的最小跨设备自动化后端。

主要要求：

1. 初始化 Python 项目及依赖管理。
2. 构建 FastAPI 服务。
3. 创建统一 DeviceAdapter 接口。
4. 实现不依赖真实硬件的 MockDeviceAdapter。
5. 提供基础设备注册与查询能力。
6. 实现命令提交与任务状态查询。
7. 使用 SQLite 保存必要的任务记录。
8. 为主要 API 编写自动化测试。
9. 提供 Dockerfile 和 Docker Compose 配置。
10. 编写项目部署与测试说明。

约束要求：

- 不依赖任何非公开代码。
- 不使用真实设备凭据。
- 不假设尚未实现的接口已经存在。
- 不将模拟测试结果表述为真实硬件测试结果。
- 不在未经测试的情况下宣称功能已经完成。

### 10.3 第一阶段验收要求

第一阶段的完成标准包括：

**功能验证**

能够注册模拟设备、提交有效控制命令，并查询相应的任务记录。

**异常验证**

能够正确处理不存在的设备、非法控制参数和模拟执行失败。

**运行验证**

可以在指定的 Docker 环境中成功启动服务，且健康检查能够得到预期响应。

**测试验证**

自动化测试能够在明确的测试环境中运行，并提供通过或失败的结果。

**文档验证**

仓库提供真实可用的运行说明、依赖信息和测试步骤。

### 10.4 后续 Devin 任务

在基础服务通过测试后，计划继续研究：

- OSC 消息接收
- Art-Net 数据包构造
- 模拟灯光控制
- 实时事件通知
- 任务执行记录
- 通信故障模拟
- API 权限管理

后续任务将根据上一阶段实际结果调整。

---

## 11. 测试方案（Testing Strategy）

### 11.1 单元测试

使用 Pytest 对独立功能进行验证。

主要包括：

- 设备参数验证
- 设备注册与查询
- 命令格式校验
- 适配器返回结果
- 任务状态变化
- 异常处理

### 11.2 API 测试

测试 HTTP 请求参数、响应格式、错误状态及访问权限。

### 11.3 通信测试

计划通过本地模拟发送端和接收端，验证 OSC 和 Art-Net 数据处理。

重点确认消息内容、地址、通道值及错误处理是否符合预期。

### 11.4 故障测试

使用模拟环境测试：

- 设备无响应
- 非法命令
- 连接中断
- 服务重新启动
- 重复请求
- 任务执行超时

### 11.5 测试报告

未来每个主要开发阶段将记录：

- 测试环境
- 测试日期
- 代码版本
- 测试用例
- 实际结果
- 已知问题
- 未完成事项

测试结果将以实际执行记录为准。

---

## 12. 开发计划（Development Roadmap）

### Phase 1: 基础后端

- [ ] 建立项目目录
- [ ] 创建 Python 开发环境
- [ ] 初始化 FastAPI 服务
- [ ] 定义 DeviceAdapter 接口
- [ ] 实现 MockDeviceAdapter
- [ ] 创建基础设备管理 API
- [ ] 实现命令提交功能
- [ ] 添加 SQLite 数据存储
- [ ] 编写单元测试
- [ ] 完成基础 Docker 部署

### Phase 2: 通信协议实验

- [ ] 实现 OSC 消息接收
- [ ] 建立 OSC 命令映射规则
- [ ] 研究 Art-Net 数据包格式
- [ ] 实现模拟 Art-Net 发送端
- [ ] 实现模拟接收端
- [ ] 添加 DMX 通道参数验证
- [ ] 编写通信协议测试

### Phase 3: 自动化与状态管理

- [ ] 完善任务调度机制
- [ ] 增加任务执行记录
- [ ] 添加 WebSocket 状态通知
- [ ] 实现基础权限校验
- [ ] 添加异常处理
- [ ] 验证重复请求处理
- [ ] 编写故障测试

### Phase 4: 系统集成

- [ ] 进行模块联合测试
- [ ] 完善容器部署
- [ ] 测试服务重新启动
- [ ] 完善日志管理
- [ ] 整理 API 文档
- [ ] 发布独立演示版本

### Phase 5: 后续研究

- [ ] 评估真实硬件适配条件
- [ ] 研究其他公开设备协议
- [ ] 评估长期运行稳定性
- [ ] 优化任务管理与设备配置
- [ ] 总结 AI 辅助开发的实际效果

---

## 13. 预期项目结构（Planned Repository Structure）

后续计划采用以下目录组织方式。

| 路径 | 作用 |
|---|---|
| `app/main.py` | FastAPI 应用入口 |
| `app/api/` | HTTP API |
| `app/core/` | 核心配置及权限组件 |
| `app/adapters/` | 设备适配器 |
| `app/models/` | 数据模型 |
| `app/services/` | 任务及设备管理服务 |
| `app/protocols/` | 通信协议实现 |
| `tests/` | 自动化测试 |
| `docs/` | 技术文档 |
| `examples/` | 独立演示程序 |
| `docker/` | 容器配置 |
| `README.md` | 项目说明 |
| `.gitignore` | Git 忽略规则 |
| `pyproject.toml` | Python 项目配置 |
| `compose.yaml` | Docker Compose 配置 |

此结构仅为规划，实际目录可能随开发调整。

---

## 14. 项目限制（Known Limitations）

目前项目存在以下限制：

1. 尚未发布完整的可运行版本。
2. 尚未完成真实设备的通信验证。
3. OSC 与 Art-Net 模块处于设计阶段。
4. 设备自动发现能力尚未确定。
5. 各类设备的可用控制接口需要单独研究。
6. Docker 部署仍需实际测试。
7. 自动化任务的长期稳定性尚未验证。

本项目将优先保证每个阶段的软件功能可以实际测试，再逐步增加新功能。

---

## 15. 公开范围与保密要求（Security and Confidentiality）

本仓库仅包含基于公开资料独立编写的实验代码、说明文档和测试内容。

项目不包含：

- 国家秘密或依法限制公开的技术资料
- 未经授权的第三方源代码
- 私人项目的受限制实现
- 内部系统配置信息
- 私人访问密钥
- 真实设备登录凭据
- 未经授权的硬件控制资料

所有公开内容均应符合相关法律、知识产权和保密规定。

如果某项研究内容不允许公开，则不会将其纳入本仓库。

---

## 16. 开发记录（Development Notes）

本仓库将持续记录实际开发进展。

计划记录的内容包括：

- 功能开发计划
- 代码提交
- 主要技术选择
- 自动化测试结果
- 已解决的问题
- 尚未解决的问题
- 版本更新说明

在提交第一版可运行代码后，将补充真实的运行截图、测试结果和使用示例。

---

## 17. 开源与贡献（Contributing）

目前项目主要作为个人学习与实验用途。

欢迎基于公开技术展开讨论，并提出合理的软件改进建议。

如果后续开放外部贡献，将补充贡献指南、代码规范与问题反馈流程。

在确定适当的开源许可证之前，本仓库不应被视为已经授予任意复制、修改或再分发代码的许可。

---

## 18. 免责声明（Disclaimer）

My Device Bot 是一个独立的软件开发实验项目。

本仓库所列功能、硬件接入方式和开发阶段均为计划内容，除非对应代码与测试记录已经公开，否则不代表相关功能已经实现。

本文档仅用于说明公开实验项目的设计目标和开发计划。

项目不保证与任何特定商业设备兼容，也不代表任何硬件制造商、协议维护方或 AI 开发工具提供商的官方立场。

本项目中的 AI 辅助开发任务仍需经过人工检查与实际测试。

---

## 19. English Summary

My Device Bot is an experimental personal automation project intended to explore unified device management, asynchronous task execution and communication through publicly documented protocols.

The planned system uses Python and FastAPI as its application foundation, with a modular DeviceAdapter interface designed to support simulated devices and future protocol implementations.

Initial research focuses on API development, mock device control, OSC messaging, Art-Net communication experiments, containerized deployment and automated testing.

Devin may be used as an AI-assisted development tool for selected programming, testing and documentation tasks.

The repository is currently in its planning and early development stage. Features described in this document are proposals unless supported by actual implementation and test results.

Only independently developed, publicly shareable and non-sensitive materials are intended to be included.

---

**My Device Bot**

*An experimental project exploring personal automation, unified device interfaces and AI-assisted software development.*
