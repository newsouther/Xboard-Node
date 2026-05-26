# 项目编码规范

## 项目文件结构（目录树）

本项目为 Xboard VPN 后端的节点端（Go 实现），负责对接面板下发的节点配置，启动并管理 sing-box / xray-core 代理内核，支持单节点、多节点、机器模式三种运行方式。

```
Xboard-Node/
├── CLAUDE.md                            # Claude 开发规范
├── Dockerfile                           # 容器镜像构建文件
├── Makefile                             # 本地构建、测试、安装、Docker 打包任务
├── README.md                            # 项目简介、特性、安装方式与配置说明
├── config.yml.example                   # 示例配置文件（单节点/多实例写法）
├── go.mod                               # Go 模块定义与依赖版本
├── go.sum                               # Go 依赖校验和
├── install.sh                           # 系统服务部署与初始化安装脚本
├── docs-custom-outbounds.md             # 自定义 outbound 配置说明文档
├── docs-custom-routes.md                # 自定义路由规则说明文档
├── docs-dns-providers.md                # ACME DNS-01 提供商说明文档
├── cmd/
│   ├── xboard-node/
│   │   └── main.go                      # 节点服务主程序入口：加载配置、启动服务、热重载、健康检查、多实例/机器模式调度
│   └── xbctl/
│       └── main.go                      # 运维命令行工具：状态查询、绑定管理、配置生成、升级、卸载、服务启停
└── internal/
    ├── cert/
    │   ├── cert.go                      # 证书管理核心：自动 TLS（ACME）、手动证书加载、热更新、PEM 重载
    │   ├── cert_test.go                 # 证书模块测试
    │   └── dnsproviders/                # ACME DNS-01 各云厂商适配（每文件对应一个 DNS 提供商）
    │       ├── registry.go              # DNS 提供商注册表与查找逻辑
    │       ├── registry_test.go         # 注册表测试
    │       ├── helpers.go               # DNS 提供商通用辅助函数
    │       ├── alidns.go                # 阿里云 DNS
    │       ├── azure.go                 # Azure DNS
    │       ├── bunny.go                 # Bunny DNS
    │       ├── cloudflare.go            # Cloudflare DNS
    │       ├── desec.go                 # deSEC DNS
    │       ├── digitalocean.go          # DigitalOcean DNS
    │       ├── duckdns.go               # DuckDNS
    │       ├── gandi.go                 # Gandi DNS
    │       ├── godaddy.go               # GoDaddy DNS
    │       ├── googleclouddns.go        # Google Cloud DNS
    │       ├── hetzner.go               # Hetzner DNS
    │       ├── huaweicloud.go           # 华为云 DNS
    │       ├── linode.go                # Linode DNS
    │       ├── namecheap.go             # Namecheap DNS
    │       ├── namesilo.go              # NameSilo DNS
    │       ├── netlify.go               # Netlify DNS
    │       ├── ovh.go                   # OVH DNS
    │       ├── porkbun.go               # Porkbun DNS
    │       ├── route53.go               # AWS Route53 DNS
    │       ├── tencentcloud.go          # 腾讯云 DNS
    │       └── vultr.go                 # Vultr DNS
    ├── config/
    │   ├── config.go                    # 配置加载、继承、默认值、校验、实例展开与环境变量解析（配置中枢）
    │   ├── config_test.go               # 配置模块测试
    │   ├── standalone.go                # standalone 模式配置适配（不依赖面板，本地静态配置）
    │   └── watcher.go                   # 配置文件监听与热重载
    ├── controlplane/
    │   ├── types.go                     # 控制平面通用接口与类型定义
    │   ├── local.go                     # 本地控制平面实现（standalone 模式）
    │   ├── local_test.go                # 本地控制平面测试
    │   ├── panel.go                     # 面板 API 控制平面：握手、拉取节点/用户配置、推送流量/状态
    │   ├── panel_test.go                # 面板控制平面测试
    │   ├── machine.go                   # 机器模式控制平面：适配面板机管理接口，动态发现节点
    │   ├── mailbox.go                   # 节点事件邮箱与状态缓存（WS 事件分发缓冲）
    │   └── mailbox_test.go              # 邮箱逻辑测试
    ├── kernel/
    │   ├── kernel.go                    # 内核抽象接口与公共逻辑（sing-box / xray 统一入口）
    │   ├── customcfg.go                 # 自定义内核配置合并逻辑
    │   ├── custom_route_compile.go      # 自定义路由规则编译为内核格式
    │   ├── geo.go                       # GeoIP / GeoSite 数据处理
    │   ├── geodata/
    │   │   └── geodata.go               # Geo 数据资源访问与管理
    │   ├── singbox/
    │   │   ├── singbox.go               # sing-box 内核生命周期与控制逻辑（启动/停止/热重载）
    │   │   ├── config.go                # sing-box 配置生成（将 NodeSpec/UserSpec 转为 sing-box JSON）
    │   │   ├── config_test.go           # sing-box 配置测试
    │   │   ├── conntracker.go           # sing-box 连接追踪（统计在线用户与流量）
    │   │   └── singbox_test.go          # sing-box 模块测试
    │   └── xray/
    │       ├── xray.go                  # xray-core 内核生命周期与控制逻辑
    │       ├── config.go                # xray-core 配置生成（将 NodeSpec/UserSpec 转为 xray JSON）
    │       ├── config_test.go           # xray 配置测试
    │       ├── dispatcher.go            # xray 流量分发/调度逻辑（用户流量统计钩子）
    │       ├── dispatcher_test.go       # xray 分发逻辑测试
    │       └── xray_test.go             # xray 模块测试
    ├── limiter/
    │   ├── limiter.go                   # 设备数限制与限速器（按用户限制并发设备与带宽）
    │   ├── limiter_test.go              # limiter 测试
    │   ├── speedtracker.go              # 用户速度统计与限速追踪（滑动窗口速率计算）
    │   └── speedtracker_test.go         # speedtracker 测试
    ├── machine/
    │   └── machine.go                   # 机器模式总编排器：动态发现面板节点、启动/停止 Service、共享 WS mux 分发事件
    ├── model/
    │   ├── types.go                     # 核心领域模型：NodeSpec、UserSpec、OutboundConfig、RouteRule、MultiplexConfig
    │   ├── panel.go                     # 面板 API 数据 ↔ NodeSpec/UserSpec 双向转换（含证书、路由、outbound、multiplex）
    │   ├── standalone.go                # standalone 配置 → NodeSpec/UserSpec 转换与校验入口
    │   ├── validate.go                  # 节点配置统一校验：内核类型归一化、outbound 冲突、路由规则、transport 兼容性
    │   ├── custom_config_tags.go        # 从自定义出站/配置文件中收集 outbound tag，检测 tag 冲突
    │   ├── custom_outbound_support.go   # 各内核对自定义 outbound 协议与特性的支持矩阵
    │   ├── custom_outbound_validate.go  # 自定义 outbound 配置校验（协议、tag、settings、proxy_tag、端口）
    │   ├── custom_route.go              # 自定义路由规则、匹配条件与动作结构定义
    │   ├── custom_route_support.go      # 各内核对路由匹配项和动作的支持矩阵
    │   └── custom_route_validate.go     # 自定义路由规则校验（匹配项、端口范围、网络类型、动作目标）
    ├── monitor/
    │   └── monitor.go                   # 系统指标采集：CPU、负载、内存、磁盘、网络速率、GC、goroutine 数（用于状态上报）
    ├── nlog/
    │   └── nlog.go                      # 结构化日志：彩色输出、节点前缀、启动摘要
    ├── panel/
    │   ├── client.go                    # 面板 REST API 客户端：握手、拉取配置/用户、上报流量/状态/机器信息、ETag 缓存
    │   ├── types.go                     # 面板 API 数据结构：节点配置、证书、路由规则、用户响应等类型定义
    │   └── ws.go                        # 面板 WebSocket 客户端：重连、心跳、配置/用户/设备/节点同步事件处理
    ├── service/
    │   └── service.go                   # 节点运行核心：控制平面对接、内核启动/热更新、用户/设备同步、流量上报、证书联动
    └── tracker/
        └── tracker.go                   # 用户流量增量追踪：在线 IP、连接数、速度统计、快照发布与恢复
```

---

## AI开发规范

- 当前文件只能调整项目文件结构，禁止新增其他任何内容
- 查找、阅读文件等操作，路径优先按目录树路径读取，不要只执行模糊搜索
- 使用中文回复和中文代码注释
- 执行任务，必须完整阅读当前文件中的项目文件结构，必须完整阅读所有和用户任务相关的文件（即使您觉得该文件不用修改），然后再决定用户的任务涉及哪些文件的新增和修改
- 在执行用户的任务前，需要先写一份实施计划和用户确认（不确定的也要向用户确认，获取更详细的需求信息）
- 不能运行、编译整个项目，不能执行pip安装依赖包命令等
- 新增的代码和修改务必遵循易维护、可读性高、函数职责明确、注释详细原则
