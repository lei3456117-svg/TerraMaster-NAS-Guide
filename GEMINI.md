# 用户网络环境全局规则与调优规范 (User Network Rules & Guidelines)

> ⚠️ **强制约束**：在本工作区开启任何新对话时，**必须将阅读并遵守本网络规则作为第一先决条件**。禁止要求用户重复描述拓扑与工具链。
> 详尽手册见：`./网络调优实战日记与全环境沉淀档案.md` 与 `./网络环境档案与公司软路由实战手册.md`

---

## 1. 公司硬件拓扑与网络形态
* **物理链路**：`公司外网/交换机 ➡️ x86_64 软路由 (192.168.100.1) ➡️ 实体 Wi-Fi AP ➡️ Mac 电脑 / NAS`
* **淘汰设备**：原 Newifi 3 D2 硬件老化崩溃已废弃，现已全面切换为独立软路由。
* **业务网段**：`192.168.100.0/24`，软路由运行 PassWall 或 OpenClash。

## 2. Mac 核心业务矩阵与流量诉求
* **重度 AI 工具链**：`Antigravity`、`Codex`、`Cursor`、`Hermes`、(未来) `Claude Agent`。要求出境长连接稳定、低握手延迟。
* **国内协同/国产 AI**：`扣子 (Coze)`、`qoder 的 WorkBuddy`、`Kimi`。必须直连国内，严禁走海外代理。
* **高频海外基础设施**：`GitHub`、`Google`、`YouTube`。
* **办公 NAS**：铁威马 NAS，通过打洞穿透软件（Tailscale）实现远程访问。

## 3. 运行双轨制（排查网络第一步必查）
* **轨 1：软路由正常模式**
  * Mac 连 Wi-Fi / 软路由网线。
  * **Mac 本机规则**：彻底退出本地 Clash Verge 与 Antify，全交由软路由透明分流。
* **轨 2：软路由故障逃生模式**
  * Mac 跳过软路由，直连公司交换机上国内公网。
  * **Mac 本机规则**：打开 `Clash + Antify`（利用 Antify 的 macOS 扩展透明代理能力，将 Antigravity、Cursor、命令行等流量定向送入 Clash）。

## 4. 当前核心问题与调优铁律 (1000ms+ 延迟攻坚)
每次分析网络变慢或 1000ms+ 延迟时，按以下铁律依序排查：
1. **中转节点检查**：严禁境外直连慢速节点（电信 163 直连延迟 460ms+）。优先选用带国内中转或 BGP 入口的亚太节点。
2. **Tailscale DNS 检查**：Mac 执行 `/Applications/Tailscale.app/Contents/MacOS/Tailscale set --accept-dns=false`，严禁 MagicDNS 造成 1.5s~2s 阻塞。
3. **QUIC 状态**：软路由必须勾选「停用 QUIC (443/UDP)」，防止浏览器探测超时。
4. **NAS 端口放行**：软路由必须将 `3478,41641` 加入非代理 UDP 端口。

## 5. AI 工具链分流与风控隔离铁律（节点通用化规范）
* **核心定论（节点无贵贱，关键在规则与 DNS）**：便宜节点与贵节点在驱动 AI Agent 时无本质差异，决定速度与稳定性的核心是 **精准分流规则** 与 **DNS 干净解析**。
* **通用出口 4 级隔离规范**：
  1. **【高风控隔离出口·美国】**：专供 `Claude Code`、`Claude Agent`、`OpenAI / Codex`。严苛防封号与防异地漂移，必须锁定固定美国出口。
  2. **【亚太低延迟出口·台新】**：专供 `Antigravity`、`Google Gemini API`、`GitHub`。追求 100~200ms 极速响应，保障高频 gRPC/WebSocket 长连接。
  3. **【通用日常出口】**：通用外网兜底（YouTube 视频、普通海外网页）。
  4. **【国内直连白名单】**：`扣子`、`WorkBuddy`、`Kimi` 等国内工具内核级 0ms 放行。
