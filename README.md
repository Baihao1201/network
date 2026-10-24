# 校园网自动登录（Windows · 通用驱动式）

自动完成校园网 Web Portal 认证：**开机 / 插网线 / 连 WiFi / 掉网** 时自动登录，
**无线弹出窗口、无定时轮询**。有线与 WiFi 都支持且不限定 SSID，笔记本同样适用。

> ✅ 已在 **陕西师范大学（SNNU）** 实测通过。
> 🔧 其它学校：往 `drivers/` 放一个"驱动"文件即可适配，**不用改代码**。

[English](README_EN.md) | 中文

---

## 一、特性

| 能力 | 说明 |
|---|---|
| **触发时机** | 登录 Windows · 网络已连接（网线/WiFi）· 连上 WiFi · 掉网 |
| **够快** | 用原始 TCP 探测判断网络状态（**不做 DNS 解析**），离线判定约 25 ms 级；从网络就绪到认证成功约 1～3 秒 |
| **够静** | 通过 VBS 隐藏窗口启动，不弹 PowerShell 窗口 |
| **零后台** | 无常驻进程 / 服务，空闲时 CPU 与内存占用均为 0（见第八节实测数据） |
| **够稳** | 以"**能否真的上外网**"作为唯一成功标准，而不是看网页返回值 |
| **好迁移** | 账号密码等放在独立配置文件；一键安装 / 一键卸载；无需管理员权限 |
| **可扩展** | 门户的"地址 / 字段 / 运营商映射"全部写在数据文件（驱动）里 |
| **有界面** | 自带图形配置工具，可视化填账号、测试、安装 |

纯 PowerShell 5.1 实现，依赖仅系统自带的 `curl.exe`，**不需要 Python 或任何第三方库**。

---

## 二、快速开始（3 步）

1. 解压后双击 **`gui.cmd`**。
2. 填写 账号 / 密码 / 运营商，勾选「**记住账号密码（仅存本机）**」，
   点「测试登录」确认能连上，再点「**安装 / 更新任务**」。
3. 完成。之后登录系统、插网线、连 WiFi、掉网时都会自动认证。

> - 勾选「记住账号密码」后，下次打开会自动带出，而且**自动登录必须勾选才能生效**（凭据只存本机）。
> - 无需管理员权限。给同学用：把整个文件夹拷过去，同样在 GUI 里填一次、点安装即可。
> - 也可以不用图形界面：把 `config.example.psd1` 复制为 `config.psd1` 手工填写，再双击 `install.cmd`。

---

## 三、图形界面

双击 `gui.cmd`，可以：

- 填写并保存账号、密码、运营商、学校（驱动）、可选的门户地址/端口；
- 勾选「**记住账号密码（仅存本机）**」——下次打开自动带出，是自动登录的前提；
- **测试登录**（立即跑一次，输出显示在下方）；
- **安装 / 更新计划任务**、**卸载任务**；
- 一键打开日志、打开文件夹。

> 未勾选「记住账号密码」时，程序不会把凭据写入磁盘，因此无法自动登录（点安装/测试会提示先勾选）。

---

## 四、适配别的学校（开源重点）

校园网门户五花八门，但"请求长什么样"是可以数据化描述的。本项目把这件事抽成了**驱动**：

### 4.1 驱动是什么

`drivers/<名称>.psd1`，就是一个纯数据文件：

```powershell
@{
    name        = 'snnu'
    displayName = '陕西师范大学（SNNU Portal）'
    probeHost   = '202.117.144.205'      # 门户地址
    probePorts  = @(8603, 8602, 8601)    # 门户端口（按优先级）
    carriers    = @{ unicom='unicom'; mobile='mobile'; telecom='telecom'; campus='' }
    login       = @{
        urlTemplate = 'http://{host}:{port}/snnuportal/login'
        method      = 'POST'
        fields      = @{
            sourceurl = 'null'
            account   = '{account}'
            password  = '{password}'
            yys       = '{carrier}'
            issave    = ''
        }
    }
    logoff      = @{ urlTemplate = 'http://{host}:{port}/snnuportal/logoff'; method='POST' }
}
```

占位符 `{host} {port} {account} {password} {carrier}` 会在运行时被替换。

### 4.2 怎么给自己的学校写一个

1. 复制 `drivers/generic.psd1` → `drivers/我的学校.psd1`；
2. 浏览器打开校园网登录页，按 `F12` → **网络(Network)**，正常登录一次，
   找到那个登录请求，把它的 **地址、方法、表单字段** 抄进驱动；
3. `config.psd1` 里把 `driver` 改成 `'我的学校'`，安装即可。

因为脚本是**用"能不能上网"来判定成功**的，所以只要你抄的请求本身是对的，就能工作；
即使门户返回什么状态码、跳转到哪个页面都不影响。

> 目前 `snnu` 驱动是作者实测有效的；其余常见门户类型（深澜 Srun、Dr.COM、锐捷等）
> 欢迎按上面的方法做成驱动并 PR 回本项目。

---

## 五、配置说明（config.psd1）

```powershell
account  = '你的学号'
password = '你的密码'
carrier  = 'unicom'          # unicom=联通  mobile=移动  telecom=电信  campus=校园网
driver   = 'snnu'            # 用哪个驱动

portalHost  = ''             # 可选覆盖，留空用驱动默认
portalPorts = @()            # 可选覆盖，留空用驱动默认

internetProbeIps = @('223.5.5.5', '223.6.6.6')                    # TCP 快速探测（不依赖 DNS）
internetTargets  = @('https://www.baidu.com', 'https://www.taobao.com')  # HTTP 精确确认

waitReadySec       = 45      # 网络就绪后持续尝试的总时间窗口（秒）
tcpTimeoutMs       = 500     # 单次 TCP 探测超时（毫秒）
internetTimeoutSec = 4       # HTTP 外网确认超时（秒）
portalTimeoutSec   = 6       # 门户登录请求超时（秒）
loginRetryDelayMs  = 1500    # 两次登录尝试的最小间隔（毫秒）
```

> 改完保存即可，**无需重新安装**（计划任务每次运行都会重新读取配置）。

---

## 六、什么时候会自动触发（事件驱动，无定时轮询）

| 触发时机 | 说明 |
|---|---|
| **登录 Windows 时** | 进入桌面后自动认证 |
| **网络已连接时** | 插网线 / 连上任意 WiFi（系统 `NetworkProfile` 事件） |
| **连上 WiFi 时** | 不限定 SSID，`SNNU`、`SNNU-5G` 等都能触发 |
| **掉网时** | 网线/WiFi 还连着但外网断了（如认证过期），系统 `NCSI` 检测到即触发 |

---

## 七、为什么 WiFi 也能很快（技术说明）

早期版本在 WiFi 上要等十几秒甚至几十秒，根因有三：

1. **DNS 阻塞**：刚连上 WiFi 时 DNS 还没就绪，走域名的检测会卡在解析上（可达数秒），
   再乘以多个目标、多轮轮询，就是几十秒。
2. **外部进程开销**：每次检测都启动一个 `curl.exe`。
3. **轮询粒度粗**：1 秒一轮，网络就绪后要等下一轮才发现。

现在改为：

- **TCP 直达探测**：用 .NET `TcpClient` 连公网 IP 的 443 端口，**完全不碰 DNS**；
  在线时约 25 ms，离线时也只是一个短超时。
- **不做无谓判断**：一旦门户 TCP 可达，就**立即**发登录请求，不再先花时间探测外网。
- **细粒度循环**：250 ms 一轮，网络一就绪马上认证。
- **精确确认**：只有当快速探测显示"在线"时，才用一次 HTTP 请求做权威确认。

---

## 八、后台与资源占用（实测）

**结论：本程序没有任何常驻进程或后台服务**，是纯事件驱动的"跑一次就退出"模式。

| 状态 | CPU | 内存 |
|---|---|---|
| 空闲（网络正常、无事件） | **0** | **0**（进程根本不存在） |
| 触发一次（已在线，最常见） | 约 0.5 秒 CPU 时间（几乎全是 PowerShell 自身启动开销） | 峰值约 85 MB，存在不到 1 秒 |
| 触发一次（需要登录） | 同上，总耗时约 3 秒 | 同上，存在约 3 秒 |
| 最坏情况（触发后网络始终不可用） | 约 0.5 秒（其余时间都在 sleep，几乎不耗 CPU） | 峰值约 85 MB，最长持续 `waitReadySec`（默认 45 秒）后自动退出 |

补充说明：

- **触发频率完全由事件决定**：登录系统 / 插网线 / 连 WiFi / 掉网各一次；
  网络稳定时整天都不会运行，**不存在定时轮询**。
- 计划任务已设置 **多实例策略 = IgnoreNew**（上次没跑完不会重复启动）和
  **超时上限 = 5 分钟**（万一异常卡住也会被强制结束，绝不会永久驻留）。
- 那 ~85 MB 是 Windows PowerShell 自身的启动开销（脚本本身只有几 KB），且只存在一两秒。
- 脚本还做了优化：若几秒内检测到**根本没有已连接的物理网卡**，会立即退出，不空转。
- 想随时自己确认：**双击 `status.cmd`**，会直接告诉你有没有进程在跑、任务状态、触发次数。

---

## 九、目录结构

```
campus-login.ps1         主脚本（通用，驱动式）
config.psd1              你的配置（已被 .gitignore 忽略，不会误传）
config.example.psd1      配置模板
首次使用说明.txt          给新手的 3 步上手说明
drivers/snnu.psd1        陕西师范大学驱动（实测）
drivers/generic.psd1     通用表单门户模板
gui.ps1 / gui.cmd        图形配置工具
gui-run.ps1              图形工具的调用包装器（统一 UTF-8 输出，防止日志乱码）
install.ps1 / .cmd       一键安装（注册计划任务）
uninstall.ps1 / .cmd     一键卸载
status.ps1 / status.cmd  状态自检（查看有无后台占用、触发次数）
run-hidden.vbs           隐藏窗口启动器（不弹窗）
run-login.cmd            手动登录一次
README.md / README_EN.md 中英文说明
logs/                    运行日志（已被忽略，超 512KB 自动截断）
LICENSE                  MIT 协议
```

---

## 十、常见问题

**Q：连了 WiFi 但上不了网？**
双击 `run-login.cmd` 手动跑一次，再看 `logs\campus-login.log`：
- "外网正常，已在线" → 其实已经通了；
- "正在登录…登录成功" → 正常；
- 一直没成功 → 检查 `config.psd1` 账号/密码/运营商，或驱动里的字段是否与门户一致。

**Q：怎么强制重连一次？**

```cmd
powershell -NoProfile -ExecutionPolicy Bypass -File "campus-login.ps1" -ForceLogin
```

**Q：怎么确认计划任务在跑？**

```powershell
Get-ScheduledTask -TaskName 'CampusAutoLogin' | Select State
Get-ScheduledTaskInfo -TaskName 'CampusAutoLogin' | Select LastRunTime, LastTaskResult
```

**Q：会不会泄露我的密码？**
密码明文存在本地 `config.psd1`（个人电脑常规做法）。该文件已在 `.gitignore` 中，
**不会**被提交到 Git 仓库；对外分发时请只发 `config.example.psd1`。

---

## 十一、卸载

双击 `uninstall.cmd`，或：

```powershell
Unregister-ScheduledTask -TaskName 'CampusAutoLogin' -Confirm:$false
```

---

## 十二、开源协议

[MIT](LICENSE)　欢迎提交其它学校的驱动（`drivers/*.psd1`）与改进。
