# Rainyun-Qiandao-v3.0 (Selenium)

**🐳 容器化部署，内置定时任务**

**v3.0 版本更新！验证码识别切换纯算法方案，识别率高达 99%**

**雨云签到工具 容器化部署后可实现每日自动签到~**

众所周知，雨云为了防止白嫖加入了TCaptcha验证码，但主包对JS逆向一窍不通，纯请求的方法便走不通了。

因此只能曲线救国，使用 **Selenium+OpenCV** 来模拟真人操作。

经不严谨测试，目前的方案验证码识别率高达**~~48.3%~~ 99%**~~，不过多次重试最终也能通过验证，那么目的达成~~！

**本分支特色功能：**

1. ✅ Docker 一键部署 —— 提供 `Dockerfile` 与 `docker-compose`，开箱即用，无需配置环境
2. ✅ GitHub Actions —— 支持利用 GitHub Actions 免费资源进行每日自动签到，无需服务器
3. ✅ 宝塔面板 (BT Panel) / Linux 特殊虚拟主机运行 —— 提供 `script/run_bt.sh` 脚本，无需配置环境
4. ✅ 多账号支持 —— 支持配置无限个账号并发签到（使用 `|` 分隔），各账号随机浏览器指纹，并发执行
5. ✅ 多通道通知 —— 支持 PushPlus、WXPusher、钉钉、邮件等多种通知方式
6. ✅ 代理 IP 池 —— 支持配置 HTTP 代理，防止因 IP 封锁导致的签到失败
7. ✅ 智能截图 —— 签到成功/失败自动截图并压缩上传，不仅有图有真相，还节省流量
8. ✅ 拦截自动代理 —— 动态探测 `app.rainyun.com` 可达性，被拦截时自动抓取国内免费代理绕过（无需手动配置，直连可达时优先直连，覆盖海外 Actions、海外/国内 VPS、Docker 等所有环境）
9. ✅ 签到状态校验 —— 点击领取奖励后轮询检查按钮是否变为"已完成"，检测验证码加载框（三个点）防止网络慢导致误判

## 食用方法

> [!NOTE]
> **GitHub Actions 配置教程**：[https://www.leapya.com/article/2](https://www.leapya.com/article/2) —— Fork 后配置 Secrets 即可每日自动签到，无需服务器。
>
> 下方文档主要针对 Docker / 宝塔面板等自建部署方式。
> 青龙面板部署方式独立于下方步骤，请见 [青龙面板部署教程](https://www.leapya.com/article/22)。

### 1.拉取项目

```bash
git clone --depth 1 https://github.com/LeapYa/Rainyun-Qiandao.git
cd Rainyun-Qiandao
```

### 2. 配置环境变量

复制 `.env.example` 为 `.env` 文件，并填入你的账号信息：

Windows (PowerShell):

```powershell
copy .env.example .env
```

Linux/Mac:

```bash
cp .env.example .env
```

编辑 `.env` 文件，根据里面的提示填入你的雨云账号和密码，多个账号/密码之间请使用竖线 | 分隔

<details>
<summary>📋 <b>完整参数列表（点击展开）</b></summary>

#### 🔐 雨云登录凭据（必填）

| 变量名               | 说明                         | 示例                           |
| -------------------- | ---------------------------- | ------------------------------ |
| `RAINYUN_USERNAME` | 雨云账号，多账号用`\|` 分隔 | `user1@qq.com\|user2@163.com` |
| `RAINYUN_PASSWORD` | 对应密码，多账号用`\|` 分隔 | `pass1\|pass2`                |

#### 📢 通知渠道配置（可选，至少配一个才能收到推送）

| 变量名                    | 说明                                                     | 备注                           |
| ------------------------- | -------------------------------------------------------- | ------------------------------ |
| `PUSHPLUS_TOKEN`        | [PushPlus](http://www.pushplus.plus/) Token               | 实名用户 2 万字 / 会员 10 万字 |
| `WXPUSHER_APP_TOKEN`    | [WXPusher](http://wxpusher.zjiecode.com/admin/) App Token | 限制 4 万字                    |
| `WXPUSHER_UIDS`         | WXPusher 接收者 UID，多个用`,` 分隔                    | 个人标识                       |
| `WXPUSHER_TOPIC_IDS`    | WXPusher 主题 ID，多个用`,` 分隔                       | 群发标识                       |
| `DINGTALK_ACCESS_TOKEN` | 钉钉机器人 Access Token                                  | 限制约 2 万字                  |
| `DINGTALK_SECRET`       | 钉钉机器人加签密钥                                       | 可选                           |
| `SMTP_HOST`             | SMTP 服务器地址                                          | 如`smtp.qq.com`              |
| `SMTP_PORT`             | SMTP 端口                                                | `465`(SSL) 或 `587`(TLS)   |
| `SMTP_USER`             | SMTP 登录用户名                                          |                                |
| `SMTP_PASS`             | SMTP 授权码                                              | 不是登录密码                   |
| `SMTP_TO`               | 收件人邮箱                                               | 不填则默认发给第一个签到账号   |

> **关于推送内容超长**：当推送内容超过渠道字符限制时，程序会自动降级：完整报告 → 无截图报告 → 精简摘要，**无需手动处理**。PushPlus 还会先按 10 万字（会员）尝试，失败后自动降级到 2 万字（实名）重试。

#### ⚙️ 运行参数（可选）

| 变量名                  | 说明                             | 默认值    |
| ----------------------- | -------------------------------- | --------- |
| `SCHEDULE_TIME`       | 定时执行时间（仅 schedule 模式） | `08:00` |
| `DEBUG`               | 开启调试日志                     | `false` |
| `MAX_DELAY`           | 多账号错峰启动最大随机延时（秒） | `15`    |
| `MAX_WORKERS`         | 最大并发线程数                   | `3`     |
| `TIMEOUT`             | 请求超时时间（毫秒）             | `30000` |
| `CHECKIN_MAX_RETRIES` | 签到失败最大重试次数             | `2`     |

#### 🌐 代理 IP（可选）

| 变量名            | 说明             | 默认值           |
| ----------------- | ---------------- | ---------------- |
| `PROXY_API_URL` | 代理 IP 接口地址 | 不填则不使用代理 |

#### 📸 截图与压缩（可选）

| 变量名              | 说明                                                                  | 默认值          |
| ------------------- | --------------------------------------------------------------------- | --------------- |
| `SCREENSHOT_MODE` | 截图嵌入策略：`all` 全部 / `failed_only` 仅失败 / `none` 无截图 | `failed_only` |
| `SCREENSHOT_QUALITY` | 截图 JPEG 质量上限（10-100），实际只会更低：逐档下调取画质仍达标的最低档，设得很低时等同固定质量 | `35` |
| `TINYPNG_API_KEY` | [TinyPNG](https://tinypng.com/developers) API Key（每月免费 500 次）   | 不填则本地压缩  |

</details>

### 3. 启动服务（选择一种模式）

根据你的不同场景和使用需求，从以下三种模式中**选择一种**运行

#### 模式一：使用Docker定时运行（推荐）

适合长期部署，程序会持续运行，并在每天指定时间（默认08:00）自动执行签到。

```bash
# 启动定时服务
sudo docker compose up -d rainyun-schedule

# 查看实时日志
sudo docker compose logs -f rainyun-schedule

# 停止服务
sudo docker compose down
```

#### 模式二：使用Docker单次运行

适合测试账号配置是否正确，或者临时手动执行一次签到。运行结束后容器会自动退出。

```bash
# 立即执行一次签到（前台运行，可看到实时日志）
sudo docker compose --profile once up rainyun-once

# 或者后台运行
sudo docker compose --profile once up -d rainyun-once
```

#### 模式三：在宝塔面板 (BT Panel) / Linux 虚拟主机运行

适用于不方便使用 Docker，希望直接在 Linux 服务器（如宝塔面板环境）上运行本工具的用户，如果需要再虚拟主机上运行，请确保您的虚拟主机支持 Python 3.8+ 和 Chromium 浏览器，或者购买和使用**特殊虚拟主机**（任意使用所有函数/完全ROOT权限的虚拟主机）。

> **注意**：完整安装（Python环境 + Chromium浏览器）需要约 **200MB - 300MB** 的磁盘空间。如果您的主机空间不足 300MB，请勿尝试安装。

##### (1) 环境准备

确保您的服务器安装了 **Python 3.8+**。如果是宝塔面板：

1. 在“软件商店”搜索并安装 **“Python管理器”**。
2. 在 Python管理器 中安装 Python 3.9 或更高版本。

##### (2) 安装 Chromium 浏览器

如果您拥有 root 权限或特殊虚拟主机，请务必执行此步骤以安装系统级依赖和浏览器。
(如果无法安装Chromium，只能尝试跳过此步直接运行，但极大率会因为缺失系统库而报错)

```bash
# 给予脚本执行权限
chmod +x script/install_chromium.sh

# 运行安装脚本（需要 root 权限）
sudo ./script/install_chromium.sh
```

如果脚本执行成功，会显示 Chromium 和 ChromeDriver 的版本号。

##### (3) 安装 Python 依赖

建议使用虚拟环境（防止污染系统库）：

```bash
# 创建虚拟环境 (venv)
python3 -m venv venv

# 激活虚拟环境
source venv/bin/activate

# 安装依赖
pip install -r requirements.txt
```

##### (4) 配置定时任务（Crontab）

我们提供了一个专门用于配合 Crontab 的启动脚本 `script/run_bt.sh`。

**在宝塔面板中添加计划任务：**

- **任务类型**：Shell 脚本
- **任务名称**：雨云每日签到
- **执行周期**：每天 08:00 (或其他您想要的时间)
- **脚本内容**：

```bash
# 请修改为实际的Rainyun-Qiandao项目所在路径
bash /www/wwwroot/Rainyun-Qiandao/script/run_bt.sh
```

## 代理IP配置（可选）

项目支持两种代理方式：**拦截自动代理**（免配置）和 **自建代理接口**（可选）。

### 拦截自动代理（免配置）

雨云会动态拦截部分 IP（覆盖海外数据中心、部分国内云服务器等），表现为 `ERR_CONNECTION_REFUSED`。

脚本会动态探测 `app.rainyun.com` 可达性，被拦截时自动抓取国内免费代理绕过：

- 内置 5 个国内免费代理源（89IP、快代理、齐云IP、开心代理、ProxyScrape）并发抓取，无第三方依赖，无需从 GitHub 安装额外库（代理源选取参考了 [freeproxy](https://github.com/CharlesPikachu/freeproxy)，感谢原作者开源）
- 以 `app.rainyun.com` 为探针并发验证（状态 200 且响应 ≤3s），找到可用代理即停，确保代理能真正连上雨云
- 每个账号每次签到独立获取代理，失败自动重试
- **无需任何配置**，直连可达时优先直连，被拦截时自动启用代理

> 本地运行（国内网络）不受影响，直连即可。

### 自建代理接口（可选）

如果需要每个账号使用不同的代理IP，可以配置 `PROXY_API_URL` 环境变量。

> 由于签到任务时间比较长（大概需要三到五分钟），但免费代理的时效很短，所以如果要配置代理IP，建议购买按量付费的时间较长的代理IP，十几块钱就有一千个了，可以用很久了

### 配置方式

在 `.env` 文件中添加：

```bash
# 代理IP接口地址（不填则不使用代理）
PROXY_API_URL=http://your-proxy-api.com/get?token=xxx
```

### 支持的接口返回格式

程序支持多种常见的代理接口返回格式：

```
# 格式1：纯文本
192.168.1.1:8080

# 格式2：JSON
{"ip": "192.168.1.1", "port": 8080}

# 格式3：JSON（proxy字段）
{"proxy": "192.168.1.1:8080"}

# 格式4：嵌套JSON
{"code": 0, "data": {"ip": "192.168.1.1", "port": 8080}}

# 格式5：带协议前缀
http://192.168.1.1:8080
```

### 工作流程

1. 每个账号签到前，会单独请求一次代理接口获取新的代理IP
2. 获取代理后会自动验证连通性
3. 如果代理获取失败或验证不通过，会使用本地IP继续签到（降级策略）

## 其他注意事项

### 账号安全

- **请不要将账号密码硬编码在脚本中，而是通过环境变量传递**。
- 建议使用单独的账号进行签到，避免因为主账号异常而导致的影响。

## 更新日志

### 2026-09-01 (v3.0)

- 验证码识别切换为纯算法方案（ICR）：黑色区域阈值分割 + 多角度模板匹配 + 贪心冲突消解，一次识别全部目标，替代 ddddocr；`requirements.txt` 移除 ddddocr，解除 Python <3.13 上限与 onnxruntime 的 glibc 强绑定（Alpine 下可用 apk 安装 opencv/numpy 运行）
- 修复 Actions 偶发页面卡住：页面加载策略改为 `eager`（DOMContentLoaded 即返回），跨太平洋慢子资源挂起不再导致 `driver.get()` 卡满超时；新增 `safe_get` 容错跳转，超时后页面实际已就绪则继续流程，不再误判为连接失败
- 修复诊断函数缺失 `import json`/`import requests` 导致页面超时诊断实际失效
- 修复 ChromeDriver 版本错位：`/usr/bin/chromedriver` 固定路径仅限 Docker 使用，Actions 等环境改由 Selenium Manager 自动对齐版本
- 截图压缩优化：压缩实现从 Pillow 切换到项目已有依赖 OpenCV，不为压缩单独引入新依赖；新增 `SCREENSHOT_QUALITY` 配置（默认 35，语义为质量上限），自适应搜索取逐通道 SSIM ≥ 0.95 的最低档，体积不超过所设质量的固定编码；色度 4:4:4 不降采样，修复彩色状态文字渗色；启用 JPEG 优化霍夫曼表与渐进式编码

### 2026-08-10

- 修复登录按钮 `StaleElementReferenceException`：填账号密码可能触发 Vue 重渲染导致 `login_button` 引用失效，改为填完后重新获取按钮再点击；同时 `visibility_of_element_located` 改为 `element_to_be_clickable` 确保 `enabled` 状态
- 修复代理重试反复命中慢代理：删除 `_IN_ACTIONS` 硬编码，改为 `check_rainyun_blocked()` 实时探测 `app.rainyun.com` 可达性决定是否走代理（雨云拦截为动态策略，未拦截时直连）；`get_freeproxy_ip` 新增 `exclude_ips` 参数，重试时跳过本轮已失败代理；`check_rainyun_blocked` 异常分支细化（仅连接类异常判拦截，网络毛刺不误判）；拦截探测结果缓存 5 分钟避免重试反复吃 timeout
- 修复直连场景下 renderer 超时误判为代理失败：动态探测改为直连后，Actions runner 性能波动导致的 renderer 超时被误判为 `proxy_failed=True`。区分代理/直连场景，有代理时触发换代理，直连时走普通重试
- 新增页面加载超时诊断信息：记录浏览器状态（URL/title/readyState/page_source 长度）和服务端连通性（requests 探测响应时间），区分网络波动/服务端慢/页面资源卡住三种场景
- 新增 Chrome 性能日志：页面超时时 dump 卡住期间的关键网络请求（失败请求/慢响应 TTFB>1s/主文档请求），可直接看出是 DNS/TTFB/资源下载哪个环节卡住
- 移除改进版 freeproxy 依赖：拦截自动代理改用自建轻量级抓取模块（5 个国内代理源并发抓取 + 探针验证找到即停），`requirements.txt` 不再包含 `git+https` 的 GitHub 依赖，国内安装不受 DNS 污染影响
- 移除 ip2region 离线定位依赖：国内源（89IP/快代理/齐云/开心）代理本身即 CN，ProxyScrape 自带国家码过滤，无需 ip2region.xdb

### 2026-08-03 (v2.3)

- 新增海外 IP 自动代理：海外环境（GitHub Actions、海外 VPS、Docker 等）被雨云拦截时，自动检测并抓取国内免费代理绕过拦截（基于改进版 freeproxy，找到可用即停，免配置）
- 新增签到状态校验：点击领取奖励后轮询检查按钮是否变为"已完成"，检测验证码加载框（三个点）防止网络慢导致误判
- 修复密码错误检测遗漏：密码错误时 toast 弹出后仅存在约 5 秒，但代码先等验证码超时后才检测，导致 toast 早已消失。改为验证码等待期间每 0.5 秒同时轮询 toast 错误提示，使用精确 XPath 定位 Vue-Toastification toast 元素，密码错误时秒级捕获
- 修复签到按钮 XPath：去掉末尾 `/a`，签到完成后按钮变为"已完成"时不再抛 `NoSuchElementException`
- 修复慢代理登录超时：用 `WebDriverWait` 轮询 URL 跳转替换固定 `sleep(5)`，最长等待 30 秒，避免代理慢导致误判"账号密码错误"
- 修复重试代理复用：重试时复用上次代理 IP，避免换 IP 导致服务器 Cookie 失效进而被迫走密码登录
- 修复截图并发竞争：多账号同秒截图时临时 PNG 文件名带账号标识，避免互相覆盖导致压缩失败
- 修复截图压缩失败回退逻辑：压缩失败时不再回退原始 PNG（避免邮件体积过大），改为放弃截图
- 优化登录失败诊断：区分"未配置账号密码"/"账号密码错误"/"跳转异常"，提示检查环境变量/GitHub Secrets
- 修复签到失败时 Actions 仍显示绿色成功的问题（补 `sys.exit(1)`）
- 精确化领取奖励按钮 xpath，避免误匹配"关注雨云"旁的同名按钮
- 推送通知响应超时从 10 秒延长到 30 秒

<details>
<summary>📜 历史更新日志（点击展开）</summary>

### 2026-03-30

- CI环境下隐藏积分信息
- 修复通知内容超长被截断问题，自动降级报告格式
- 增加截图嵌入策略配置（all / failed_only / none）

### 2026-02-04

- 支持多账号并发执行
- 优化日志输出，增加用户标识，提升多账号管理的可读性
- 关闭无图模式
- 调整Action默认执行时间

### 2026-02-03

- 优化点击逻辑，避免重复签到时报错显示异常
- 支持截图发送到通知功能中
- 压缩图片，减少通知大小

### 2026-01-31

- 根据账号随机浏览器指纹，增加反爬虫机制。
- 增加Cookie持久化功能，避免重复登录。
- 无图模式，减少资源占用。
- 新增代理IP支持，每个账号可独立使用不同代理IP。

### 2026-01-30

- 增加通知功能，支持PushPlus、WXPusher、钉钉、邮件通知。

### 2026-01-29

- 修复因前端弹窗导致的签到失败问题，优化自动化交互逻辑。
- 增强安全性与易用性，支持通过 `.env` 配置账号密码及运行参数，并完善文档说明。

</details>

## 常见问题

### Q: GitHub Actions / 海外 VPS 环境下签到失败，提示"代理过慢导致登录超时"或浏览器显示 This site can't be reached？

雨云于 2026 年 8 月更新了海外 IP 拦截策略，海外环境（GitHub Actions、海外 VPS、Docker 等）访问 `app.rainyun.com` 会被拒绝连接。程序会自动检测拦截并抓取国内免费代理绕过。免费代理质量参差不齐，慢代理可能导致登录请求未在 30 秒内完成。程序会自动标记失败代理并换新代理重试，最多重试 3 次。如果所有代理都太慢，可以尝试：

- 重新触发一次运行（每次抓取的代理不同）
- 自行配置优质代理（设置环境变量 `HTTP_PROXY` / `HTTPS_PROXY`）

### Q: 提示"账号或密码错误"但我的密码没问题？

请先到 [雨云登录页](https://app.rainyun.com/auth/login) 手动登录确认账号密码是否正确。如果手动登录正常但签到仍报错，请提 [Issue](https://github.com/LeapYa/Rainyun-Qiandao/issues)。

### Q: 日志显示"代理过慢"但实际上是密码错误？

代理太慢时，登录 API 请求无法完成，页面既不跳转也不弹出错误提示，程序无法区分是代理问题还是密码问题。只有代理够快时，API 返回 400 后页面才会弹出 toast 错误提示，程序才能捕获并报告"账号或密码错误"。这种情况下多试几次（或换好代理）就能看到真正的错误原因。

### Q: 提示"未配置雨云账号密码"？

说明 `RAINYUN_USERNAME` 或 `RAINYUN_PASSWORD` 环境变量为空。请按 [食用方法](#食用方法) 中的步骤配置 GitHub Secrets。

### Q: 签到成功但 Actions 显示红色失败？

这通常是因为签到过程中出现了非致命异常（如截图保存失败），但签到本身已成功。查看日志中是否有"签到成功"字样即可确认。如果确实签到失败，日志会明确标注失败原因。

### Q: 报错 `NoSuchElementException` / `TimeoutException`，提示找不到元素或等待超时？

网页加载缓慢导致元素未及时渲染。可尝试延长超时等待时间，或更换连接性更好的国内主机。

### Q: Fork 后定时任务不执行？

GitHub 对 Fork 仓库的定时任务有限制，需要手动激活：

1. 进入 Fork 仓库的 **Actions** 页面
2. 点击 **I understand my workflows, go ahead and enable**
3. 首次需要手动触发一次运行，之后定时任务才会生效

## 致谢

本项目基于 [Rainyun-Qiandao](https://github.com/SerendipityR-2022/Rainyun-Qiandao) 开发，感谢原作者的开源贡献。

验证码识别模块（ICR）移植自上游仓库，其思路源自 [RainyunCheckIn@FalseHappiness](https://github.com/FalseHappiness/RainyunCheckIn)，在此一并感谢。

> [!NOTE]
> **免责声明与致谢**
>
> - ⚠️ 本项目仅供技术交流与学习参考，请严格遵守相关法律法规，切勿将其用于任何商业或非法用途。
> - 🚫 将本项目分享到任何雨云官方相关讨论社区/群组是极其不明智的行为，请不要这么做！
> - 💡 开源不易，在您进行分发、搬运或二次开源时，请务必保留原项目出处及致谢信息，感谢您的理解与尊重！

---

## 🚀 GitHub Actions 部署指南（安全加固版）

> 本节由安全审查后追加，与上游文档并列。**按本节操作即可完成部署，且已规避下列已知风险。**

### 一、部署前必读：三项风险与对应处置

| 风险 | 说明 | 本仓库已做的处置 |
| --- | --- | --- |
| **账号密码经免费代理转发** | Actions 位于海外，直连 `app.rainyun.com` 会被拒；脚本会**自动抓取公开免费代理**，登录密码将经过来源不明的第三方节点 | 新增 `ENABLE_FREE_PROXY` 开关。默认仍为 `true`（保证开箱即用）；**强烈建议**配置 `PROXY_API_URL` 后设为 `false`，见第 3.4 节 |
| **凭据落入仓库/日志** | 工作流输入框、日志、截图、验证码调试样本都可能含敏感信息 | 已移除 `workflow_dispatch` 的账号密码输入框；`.gitignore` 补充忽略 `logs/`；日志仅失败时上传且保留 3 天 |
| **第三方 Action 供应链投毒** | `uses: owner/repo@v4` 的 tag 可被移动指向恶意提交 | 所有 Action **固定到 commit SHA**，并加 `permissions: contents: read` 收窄令牌权限 |

### 二、Fork 并启用

1. 打开 <https://github.com/LeapYa/Rainyun-Qiandao>，点右上角 **Fork** → **Create fork**。
2. **强烈建议**取消勾选 "Copy the `main` branch only"，改为只保留默认分支即可；Fork 完成后进入自己仓库。
3. 把本仓库已加固的 `.github/workflows/` 与 `rainyun.py` 覆盖上去（或直接使用已加固版本作为你的仓库）。
4. 进入 **Actions** 标签页 → 点击 **I understand my workflows, go ahead and enable**。

> Fork 仓库的 `schedule` 默认不激活，**必须先手动跑一次**才会开始定时。

### 三、配置 Secrets

进入仓库 **Settings → Secrets and variables → Actions → New repository secret**，按下表逐个添加。

#### 3.1 必填

| Name | Value |
| --- | --- |
| `RAINYUN_USERNAME` | 雨云账号（邮箱）。多账号用 `|` 分隔 |
| `RAINYUN_PASSWORD` | 对应密码。多账号用 `|` 分隔，顺序需与账号一一对应 |

> ⚠️ **务必使用小号**。该密码以明文存放，一旦泄露，攻击者可登录雨云控制台操作你的服务器、域名与余额。

#### 3.2 通知渠道（可选，至少配一个才会收到推送）

任选其一即可，推荐 **PushPlus**（配置最简单）：

| Name | Value |
| --- | --- |
| `PUSHPLUS_TOKEN` | <http://www.pushplus.plus/> 的 token |

其他渠道按需添加：`WXPUSHER_APP_TOKEN` / `WXPUSHER_UIDS` / `WXPUSHER_TOPIC_IDS`、`DINGTALK_ACCESS_TOKEN` / `DINGTALK_SECRET`、`SMTP_HOST` / `SMTP_PORT` / `SMTP_USER` / `SMTP_PASS` / `SMTP_TO`。

#### 3.3 测试账号工作流（可选）

若你保留了 `test-checkin.yml`，需额外添加 `RAINYUN_TEST_USERNAME` / `RAINYUN_TEST_PASSWORD`；**不需要就删掉该文件**。

#### 3.4 可信代理（强烈建议）

| Name | Value |
| --- | --- |
| `PROXY_API_URL` | 你自己的代理接口地址，每次请求返回一个国内 IP，格式见 `.env.example` |

配置后在 `.github/workflows/daily-checkin.yml` 中把

```yaml
ENABLE_FREE_PROXY: 'true'
```

改为 `'false'`。此时脚本**只走你自己的代理**，不再从公开代理站抓 IP，密码不会经过陌生节点。

> 若暂时没有可信代理，保持 `'true'` 也能跑通（脚本会自动找可用免费代理并重试 3 次），但请知悉其代价。

### 四、触发与验证

1. 进入 **Actions → 雨云每日签到 → Run workflow → Run workflow**（分支选 `main`）。
2. 点进这次运行，展开 **执行签到** 步骤查看日志。
3. **成功标志**：日志出现 `签到任务执行成功！` 且进程退出码为 0。
4. **失败排查**：
   - `未配置雨云账号密码` → Secrets 名称写错或为空。
   - `账号或密码错误` → 先去 <https://app.rainyun.com/auth/login> 手动登录验证。
   - `代理过慢导致登录超时` → 免费代理质量差，重跑一次（每次抓到的代理不同），或配置 `PROXY_API_URL`。
   - 找不到元素 / `TimeoutException` → 页面渲染慢，可在工作流中把 `TIMEOUT` 调大。
5. 手动跑通后，定时任务即生效：**每天北京时间 06:45**（UTC 22:45）自动执行。

### 五、运行后的安全收尾

- **关闭无关工作流**：不用 `test-checkin.yml` 就删除，减少凭据暴露面。
- **定期轮换密码**：若怀疑泄露，立即在雨云后台改密，并同步更新 Secret。
- **不要外发 `logs/` 与 `temp/`**：调试包内含验证码样本与登录后页面截图。
- **确认 `.env` 未被提交**：`.gitignore` 已忽略，切勿使用 `git add -f` 强加。
- **降低留存**：`SCREENSHOT_MODE` 可设为 `none`，日志与推送中将不含截图。
- **遵守规则**：本工具自动破解验证码并规避风控，**存在账号被限制的风险**，且可能违反雨云用户协议，请自行评估。

### 六、本地改动清单（相对上游）

| 文件 | 改动 |
| --- | --- |
| `rainyun.py` | PushPlus 推送由 `http://` 改为 `https://`；新增 `ENABLE_FREE_PROXY` 开关控制是否自动使用公开免费代理；**新增 `BarkProvider` 推送渠道**（含 APNs 4096 字节上限自适应裁剪） |
| `.env.example` | 新增 Bark 配置段（`BARK_KEY` / `BARK_SERVER` / `BARK_GROUP` / `BARK_LEVEL` / `BARK_SOUND` / `BARK_ICON` / `BARK_URL`） |
| `.gitignore` | 新增忽略 `logs/`（验证码调试样本、日志） |
| `.github/workflows/daily-checkin.yml` | Action 固定 SHA；`permissions: contents: read`；移除 push 触发与账号密码输入框；新增 `concurrency`、`timeout-minutes`；`persist-credentials: false`；日志仅失败时上传 |
| `.github/workflows/test-checkin.yml` | 同上（Action 固定 SHA、收窄权限、移除输入框） |
| `.github/workflows/*.yml` | 新增 Bark 相关环境变量透传 |

---

## 📱 使用 Bark 推送（iOS）

Bark 走 APNs，不需要注册第三方账号，**复制一串 key 就能用**，是目前最省事的推送渠道。

### 一、拿到你的 BARK_KEY

1. App Store 搜索安装 **Bark**（免费，作者 Finb）
2. 打开 App，首页会显示一条测试推送的 URL，形如：
   ```
   https://api.day.app/AbCdEf123456/推送内容
   ```
3. 其中 `AbCdEf123456` 就是你的 **device key**（等同推送凭证，请勿公开）

### 二、配置 Secrets / 环境变量

**必填（1 个）：**

| Name | Value |
| --- | --- |
| `BARK_KEY` | 你的 device key，**也可以直接粘贴 App 里复制的完整 URL**（程序会自动解析出服务器与 key） |

**可选：**

| Name | 说明 | 示例 |
| --- | --- | --- |
| `BARK_SERVER` | 自建服务器地址，留空则用官方 | `https://bark.example.com` |
| `BARK_GROUP` | 推送分组，通知中心可按组查看 | `雨云签到` |
| `BARK_LEVEL` | `active`(默认) / `timeSensitive`(专注模式也提醒) / `passive`(静默) / `critical`(重要警告) | `timeSensitive` |
| `BARK_SOUND` | 自定义铃声 | `alarm` |
| `BARK_ICON` | 自定义图标 URL（仅 iOS 15+） | `https://day.app/assets/images/avatar.jpg` |
| `BARK_URL` | 点击推送后跳转的地址 | `https://app.rainyun.com/account/reward/earn` |

> GitHub Actions 用户：在 **Settings → Secrets and variables → Actions** 中添加同名 Secret 即可，工作流已自动透传这 7 个变量。

### 三、本机验证（可选）

改完配置想先确认推送通不通，可绕过签到直接测：

```bash
python -c "import os; from rainyun import BarkProvider, setup_logging; setup_logging(); \
BarkProvider(os.environ['BARK_KEY'], group=os.getenv('BARK_GROUP'), level='timeSensitive') \
.send('雨云签到测试', {'summary_markdown': '如果你收到这条，说明 Bark 配置成功 ✅'})"
```

### 四、与其它渠道的差异

| 项目 | 说明 |
| --- | --- |
| **消息格式** | 发送 `markdown_full`（完整报告），超长时自动降级 `markdown_lite` → `summary_markdown` |
| **长度上限** | **APNs 硬上限 4096 字节**（`bark-server` 中 `PayloadMaximum = 4096`）。程序按**整包体积**动态裁剪正文（2400/1800/1200/800/400 逐档收缩），宁可少发也不会整条被拒收 |
| **图片** | Bark 不支持在通知里嵌图，因此**截图不会推送**。需要看图请用 PushPlus / 邮件渠道 |
| **安全** | `BARK_KEY` 等同推送凭证，日志中只打印 `AbCd***56` 形式；若 `BARK_SERVER` 使用 `http://`，程序会告警提示明文风险 |

### 五、常见问题

**Q: 日志显示 `Bark notification failed: HTTP 400 device token is invalid`？**
`BARK_KEY` 填错了。注意别把 App 里那条完整 URL 的 `/推送内容` 部分一起复制进来——不过即使复制了完整 URL 也能正常工作，程序会自动取最后一段作为 key。请确认 key 来自「测试推送」那条 URL。

**Q: 收到推送但内容被截断了？**
APNs 单条上限 4096 字节，属正常降级。日志中会有 `Bark: 推送内容超出 APNs 上限，正文已压缩到 xxx 字节以内`。想看得更全可改用 PushPlus 或邮件。

**Q: 想连截图一起收到？**
Bark 不支持。可同时配置 PushPlus / 邮件渠道，两边并行推送。


