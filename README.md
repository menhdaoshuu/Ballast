# Ballast（压舱石）

Android 内存优化模块，适用于 **KernelSU / Magisk**。

纯 C 事件驱动引擎，零 fork 常驻，专注「**保后台 + 降占用**」——不是无脑杀进程，而是让后台应用活着但不占物理内存。

---

## 16G 与 12G 的区别

| | **16G 版** | **12G 版** |
|---|---|---|
| 文件名 | `Ballast_16G_PureC_Universal_v6.7.3.zip` | `Ballast_12G_PureC_Universal_v6.7.4.zip` |
| version | `v6.7.3-16G-PureC` | `v6.7.4-12G-PureC` |
| versionCode | `673` | `674` |
| ZRAM 比例 | 75% | 75% |
| 软顶阈值 | 按 16G 容量调校 | 按 12G 容量调校，更早介入 |
| 更新源 | `update_16g.json` | `update_12g.json` |

**两者模块 ID 相同（`ksu_sys_mem_opt`）**，互斥，不能共存。换版本直接刷入覆盖即可。

> 选哪个：按手机**物理内存**选。16G 机型装 16G 版，12G 及以下装 12G 版。

---

## 特性

- **静态纯 C 引擎** —— 零 fork，稳态下每轮只一次 `open()`
- **事件驱动唤醒** —— inotify 监听 top-app cgroup，切应用 0ms 响应
- **75% ZRAM** —— 高压缩比，物理内存释放更彻底
- **avail_buffers 水位解放** —— 更早、更平缓地回收，减少突发卡顿
- **ART 紧凑堆** —— 降低 ART 自身占用，加快冷启动
- **23 款流媒体防膨胀软顶** —— 超阈值温和回收冷页，播放不中断
- **通知栏实时内存条** —— 零 fork 提供数据，划掉即触发一键压缩
- **免重启热更新** —— 改配置下一轮自动重载

---

## 刷入方法

### 方式一：Root 管理器直接安装

1. 下载对应容量版本的 ZIP
2. 打开 **KernelSU / Magisk** 管理器
3. 进入「模块」→「从本地安装」
4. 选择 ZIP，等待完成
5. **重启手机**

### 方式二：在线更新（推荐）

模块已内置 `updateJson`，Root 管理器会自动检查更新：

```
KernelSU：模块页 → 右上角 ↻ → 检测到新版本 → 直接更新
Magisk：  模块页 → 检测更新 → 直接更新
```

更新源走 **jsDelivr CDN**，国内可直连，无需代理。

---

## 自检：`check.sh`

刷入后如果想确认模块是否正常工作：

```bash
su -c sh /data/adb/modules/ksu_sys_mem_opt/check.sh
```

会输出当前各项状态：ZRAM 是否挂载、守护进程是否在跑、水位参数是否写入成功、软顶对哪些应用生效等。

**如果某项显示失败**，那多半是当前 ROM 没有对应的内核节点，不是模块坏了。

---

## 跨系统适配：`ballast_probe.sh`

换了新机型、或者不确定当前 ROM 该用哪个节点？

```bash
su -c sh /storage/emulated/0/ballast_probe.sh
```

或者直接：

```bash
su -c sh /data/adb/modules/ksu_sys_mem_opt/ballast_probe.sh
```

脚本会扫描当前系统上所有可用的内存控制节点，输出一份报告：

- ZRAM 设备与压缩算法
- `/dev/memcg` 与 `/sys/fs/cgroup/memory` 哪个可用
- `force_swapout` / `force_reclaim_zram` 等厂商节点的实际命名
- 内核水位文件路径
- 建议的配置取值

**把报告贴到 Issue 里**，能快速定位适配问题。

---

## 配置文件

```
/data/adb/ksu_mem_opt.conf
```

修改后**无需重启**，守护进程下一轮巡检自动重载。

常用项：

```ini
# ZRAM 占用比例（%）
ZRAM_PERCENT=75

# 是否启用通知栏内存条
ENABLE_NOTIFY=1

# 软顶应用名单（逗号分隔包名）
SOFT_CAP_PKGS=com.bilibili.app.in,com.ss.android.ugc.aweme
```

---

## 兼容性

| 项目 | 支持情况 |
|---|---|
| ColorOS / realme UI | ✅ |
| 小米澎湃 OS / MIUI | ✅ |
| HyperOS | ✅ |
| Android 版本 | 12 ~ 17（GKI 内核） |
| Root 方案 | KernelSU / Magisk / APatch |
| 需要 LSPosed | ❌ 不需要 |

---

## 注意事项

- **首次使用建议先跑 `check.sh`**，确认节点适配
- 模块会调整内核内存水位，**极端低内存场景下**可能出现后台清理偏慢
- 与其它内存优化模块（Uperf、Fas-rs、各种「内存扩展」模块）**功能重叠，不建议叠加**
- 刷入前建议先备份当前系统的 `/data/adb/modules` 目录

---

## 更新地址

- 16G：`https://cdn.jsdelivr.net/gh/menhdaoshuu/Ballast@main/update_16g.json`
- 12G：`https://cdn.jsdelivr.net/gh/menhdaoshuu/Ballast@main/update_12g.json`

---

## License

仅供个人学习与设备优化使用。刷机有风险，操作前请自行备份，因使用本模块导致的任何设备问题由使用者自行承担。
