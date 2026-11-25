# GL-BE3600 触摸屏中文界面（ipk 安装包）

本仓库只发布 OpenWrt 安装包 `.ipk`，**不包含源码**。

适用于 GL.iNet GL-BE3600（OpenWrt 23.05）。安装后触摸屏显示中文自定义界面，开机自启；原厂屏幕程序 `/usr/bin/gl_screen` 不会被修改。

## 文件

- `gl-be3600-screen-cn_1.0.0-1_all.ipk`（29,709 字节）
  - MD5：`dcfe231e5976cffe2d1a887d6ff45a66`
  - SHA-256：`b2f84054dbe425787560f4cf80fbc66da347c82293a9346923be1a6702b8fa3d`
  - 包内文件：`/usr/bin/skyris_screen_clients`（主程序，含中文字库）、`/etc/init.d/skyris_screen_clients`（自启服务）

## 安装与卸载

SSH 登录路由器后，把 ipk 上传到 `/tmp`，然后：

```sh
opkg install /tmp/gl-be3600-screen-cn_1.0.0-1_all.ipk
```

卸载并恢复原厂英文屏幕：

```sh
opkg remove gl-be3600-screen-cn
```

安装时自动启用并启动中文屏；卸载时自动停止中文屏并让原厂 `gl_screen` 接管。已实测闭环：安装 → 中文屏运行；卸载 → 原厂屏幕恢复；再次安装 → 中文屏恢复。

## 界面说明

按 iOS 风格设计：深色背景、圆角卡片、细分隔线，顶部一排 8 个页面标签（主页 / 网速 / 流量 / 系统 / 风扇 / 网络 / 设备 / 菜单），当前页白色高亮带蓝色下划线，点击标签可直接跳转；左右滑动也可翻页，底部圆点显示当前页；时钟在右下角。屏幕两侧箭头按钮已移除。

页面内容：

1. 主页 — 在线设备总数与 2.4G / 5G / 有线分组卡片
2. 网速 — 下行 / 上行卡片与趋势线
3. 流量 — 按实时速率排序的设备列表
4. 系统 — 处理器 / 内存 / 温度 / 风扇 / 闪存 / 运行时间
5. 风扇 — 转速卡片与档位按钮
6. 网络 — 连接状态与地址 / 网关 / 域名
7. 设备 — 全部在线设备，可翻页
8. 菜单 — 三个彩色按钮横排，按钮内文字上下两行：原厂60秒 / 恢复60秒、刷新 / 重新加载、息屏 / 关闭屏幕

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
