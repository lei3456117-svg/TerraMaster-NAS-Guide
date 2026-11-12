# 家庭软路由 PassWall 规则同步与配置实战指南

> **更新日期**：2026-11-12  
> **适用场景**：下班回家后，将公司已实测加固的「AI 隔离与零邮件告警」分流规则一键同步至家庭软路由。  
> **目标效果**：彻底解决在家打开 ChatGPT / Codex 时频繁收到 `New sign-in` 告警邮件及反复重新登录的问题。

---

## 一、 快速排查：为什么家庭软路由需要同步这 3 条规则？

今天在公司软路由排查出的根本原因是：
1. `geosite:openai` 仅包含 OpenAI 核心主站，缺少人机验证（Arkose）、Cloudflare 5秒盾（Turnstile）以及 Google 联合登录鉴权域名。
2. 这 3 类流量在家里会默认跌落至「默认出口」（例如香港/台湾），导致**前端风控环境在亚太、后端请求在美国**，引发 OpenAI 风控警报并触发告警邮件。
3. 只要把这 3 条规则同步进家庭软路由的 `AI_US` 规则组，整条认证链即可 100% 锁死在美国。

---

## 二、 1 分钟后台同步操作步骤

### 步骤 1：登录家庭软路由后台
* 打开浏览器，访问家庭软路由 IP（通常为 `http://192.168.100.1` 或 `http://192.168.1.1`）。
* 进入导航菜单：**服务** ➡️ **PassWall**（或 PassWall 2）。

### 步骤 2：编辑 `AI_US` 规则
1. 点击顶部标签页中的 **【规则管理】**（或【分流规则】）。
2. 在规则列表中找到 **`AI_US`**，点击其右侧的 **编辑** 按钮。
3. 滑动到 **【域名】** 输入框最底部，换行粘贴以下 3 行黄金加固域名：
   ```text
   domain:arkoselabs.com
   domain:challenges.cloudflare.com
   domain:accounts.google.com
   ```
4. 点击右下角 **保存并应用**。

---

## 三、 附：家庭软路由 `AI_US` 完整标准配置清单（可直接全量覆盖）

如果你想确保家庭软路由与公司完全对齐，可直接将 `AI_US` 域名输入框全部替换为以下标准清单：

```text
geosite:anthropic
domain:claude.ai
domain:claude.com
domain:anthropic.com
domain:claudeusercontent.com
geosite:openai
domain:openai.com
domain:chatgpt.com
domain:oaistatic.com
domain:oaiusercontent.com
domain:grok.com
domain:x.ai
domain:grok.x.com
domain:arkoselabs.com
domain:challenges.cloudflare.com
domain:accounts.google.com
```

---

## 四、 节点绑定确认与验证

### 1. 检查节点分配
在 **【分流规则】** 页面，确认各规则组绑定的落地节点：
* **`AI_US`**：必须固定绑定一个**美国节点**（如 `[IPLC·US] 美国` 或 `🇺🇲 美国 01`）。
* **`AI_APAC`**：绑定**台湾 / 新加坡**低延迟节点（专供 Antigravity、Gemini、GitHub）。
* **`默认`**：绑定通用日常出口。

### 2. 回家后 Mac 终端一键验证命令
回到家连上 Wi-Fi / 有线网后，在 Mac 终端运行以下命令：

```bash
# 验证 OpenAI API 连通性与首包
curl -I -s --connect-timeout 4 https://api.openai.com | head -n 3

# 验证 ChatGPT 网页连通性
curl -I -s --connect-timeout 4 https://chatgpt.com | head -n 3
```

返回 `HTTP/2 200` 或 `HTTP/2 403`（未鉴权状态），且无需重新输入账号密码登录网页，即代表全套环境与公司完全一致，完美运行！
