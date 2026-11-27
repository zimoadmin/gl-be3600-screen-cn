# GL-BE3600 触摸屏中文界面（ipk 安装包）

本仓库只发布 OpenWrt 安装包 `.ipk` 和升级元数据，**不包含源码**。

适用于 GL.iNet GL-BE3600（OpenWrt 23.05）。安装后触摸屏显示中文自定义界面，开机自启；原厂屏幕程序 `/usr/bin/gl_screen` 不会被修改。

## 文件

- `gl-be3600-screen-cn_1.0.4-1_all.ipk`（33,416 字节）
  - MD5：`c5732ce79a17c5e0cce8003bb4352cfe`
  - SHA-256：`02d23252c8841e23641b221f6ef1484860e448b1fa82a4c3cd8ea854581796c8`
  - 包内文件：`/usr/bin/skyris_screen_clients`（主程序，含中文字库）、`/etc/init.d/skyris_screen_clients`（自启服务）
- `version.txt`：在线升级元数据，第一行版本号、第二行安装包文件名

## 安装与卸载

SSH 登录路由器后，把 ipk 上传到 `/tmp`，然后：

```sh
opkg install /tmp/gl-be3600-screen-cn_1.0.4-1_all.ipk
```

卸载并恢复原厂英文屏幕：

```sh
opkg remove gl-be3600-screen-cn
```

安装时自动启用并启动中文屏；卸载时自动停止中文屏并让原厂 `gl_screen` 接管。已实测闭环：安装 → 中文屏运行；卸载 → 原厂屏幕恢复；再次安装 → 中文屏恢复。

## 在线升级

屏幕最后一个标签是「更新」，进入后有两个主按钮：

1. **检查更新**：设备从本仓库的 `version.txt` 读取最新版本号，与当前版本比较，结果显示在按钮下方（「已是最新」/「发现新版 x.x」/「检查失败」）。
2. **立即升级**：自动下载仓库里的最新 ipk 并执行 `opkg install --force-reinstall`，安装完成后服务自动重启，状态显示「升级成功」。下载过程中屏幕显示动画（进度条 + 扫光 + 跑马灯 + 当前下载源）。无论成功或失败，下载到 `/tmp` 的临时 ipk 都会在结束后自动删除；安装日志仅在失败时保留便于排查。

直连和五个代理**同时发起下载**，谁先通过 `tar tzf` 校验就安装谁，其余未完成的下载会被终止；六路都失败才提示「下载失败」。源列表顺序只用于编号显示，不再逐个等待：

1. GitHub 直连 `raw.githubusercontent.com`
2. `down.mxw.xx.kg`
3. `ghproxy.icu`
4. `github.mxw.qzz.io`
5. `xsadwsd.kdns.fr`
6. `v4.gh-proxy.org`

代理形式为 `https://<代理域名>/<原始 raw URL>`，下载的 ipk 会先用 `tar tzf` 校验完整性再安装。另外，如需走自定义代理，可在路由器上创建 `/etc/skyris_proxy`，内容为代理地址（如 `http://192.168.60.10:7890`），脚本会额外带上 `-x` 参数。

> 从 `1.0.4-1` 起，仓库发布的 ipk 也已包含并发竞速与自动清理逻辑，在线升级装回来的就是这一版。在线升级的 raw 地址使用版本标签 `v1.0.4`，避免 `main` 分支 raw 缓存导致拿到旧元数据。

「菜单」页仍保留原来的三个彩色按钮（原厂60秒 / 刷新 / 息屏），「更新」是它后面新增的独立页面。

> 注意：在线升级会把仓库 `version.txt` 中指向的 ipk 装到设备。发布新版本时，必须同步更新仓库里的 ipk、`version.txt` 和主程序内的 `APP_VERSION` 常量，三者保持一致。

## 界面说明

按 iOS 风格设计：深色背景、圆角卡片、细分隔线，顶部一排 9 个页面标签（主页 / 网速 / 流量 / 系统 / 风扇 / 网络 / 设备 / 菜单 / 更新），当前页白色高亮带蓝色下划线，点击标签可直接跳转；左右滑动也可翻页，底部圆点显示当前页；时钟在右下角。屏幕两侧箭头按钮已移除。

页面内容：

1. 主页 — 在线设备总数与 2.4G / 5G / 有线分组卡片
2. 网速 — 下行 / 上行卡片与趋势线
3. 流量 — 按实时速率排序的设备列表
4. 系统 — 处理器 / 内存 / 温度 / 风扇 / 闪存 / 运行时间
5. 风扇 — 转速卡片与档位按钮
6. 网络 — 连接状态与地址 / 网关 / 域名
7. 设备 — 全部在线设备，可翻页
8. 菜单 — 三个彩色按钮横排（原厂60秒 / 刷新 / 息屏），每个按钮内文字上下两行
9. 更新 — 独立的在线升级页：检查更新 / 立即升级，下方显示当前版本

## 常用命令

```sh
# 查看状态
/etc/init.d/skyris_screen_clients status

# 重启中文屏服务
/etc/init.d/skyris_screen_clients restart

# 临时显示原厂英文屏幕 60 秒，之后自动回到中文界面
/usr/bin/skyris_screen_clients oem60

# 停止中文屏，恢复原厂屏幕（保留开机自启）
/etc/init.d/skyris_screen_clients stop

# 彻底恢复原厂并取消开机自启
/etc/init.d/skyris_screen_clients disable
/etc/init.d/skyris_screen_clients stop

# 重新启用中文屏
/etc/init.d/skyris_screen_clients enable
/etc/init.d/skyris_screen_clients start
```

## 回滚校验

确认原厂屏幕程序未被改动：

```sh
md5sum /usr/bin/gl_screen
# 应为 808224bb6d93d074fcd35370e5bcb5da
```

## 实现说明

基于社区项目 `Lenoud/gl-be3600-screen` 改造：新增中文点阵字库（12×12 大字库与 10×10 小字库）并改写文本渲染以支持 UTF-8；渲染时按字库实际行数绘制，避免汉字截断、笔画叠在一起。固定文案全部汉化；字库外的设备名字符回退为 MAC 尾号，避免乱码。亮度、自动锁屏、定时开关屏仍沿用原厂设置。
