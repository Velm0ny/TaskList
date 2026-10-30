# TaskList
因为找不到自己想要功能的app，所以用dsh做了个任务列表应用。
以下全为dsh编写🥲。
# 清单 · TaskList

一个极简的 Android 任务清单 + 长期打卡应用。无广告、无联网、无第三方依赖，数据只存在本机。

- 纯 Java + 原生 View，界面全部自绘，APK 约 100 KB
- 不用 Gradle / Android Studio：`aapt2 + javac + d8 + apksigner`，一条脚本出包
- minSdk 26（Android 8.0+）· targetSdk 34

## 功能

### 任务清单

- 添加、编辑（长按）、删除（左滑）、完成次数（1–99 次，带进度弧）
- 截止时间：今天 / 明天 / 3 天后，临近截止变琥珀色，逾期变红，行首出现 ⚠
- 子任务：详情里每行一个，列表页可展开勾选，全部勾完算完成
- 备注：作业要求、链接等长文本
- 清单分类：学业 / 生活 …，颜色标签；任务文字颜色可单独自定义
- 智能输入：输入「明天交英语」「周三 18:30 开会」自动拆出时间
- 重要标记、清单筛选、深色模式、桌面小组件

### 长期打卡

- 周视图：今天 + 前 3 天四列，完成型显示 ✓ / ✕，可量化显示数值 + 单位
- 两种类型：完成型（勾选）、可量化（公里 / 小时 / 次数…）
- 统计页（长按名字进入）：周 / 月 / 季度 / 年目标、近 8 周成绩曲线、近 14 天历史柱状图、16 周日历热力图、最佳连续天数、星期频率
- 长按名字可拖动排序

### 提醒与设置

- 每条任务、每个习惯都能单独设提醒时间（截止时 / 提前 15 分钟 / 1 小时 / 1 天 / 自定义）
- 提醒方式：通知 / 响铃，铃声可在设置里更换
- 设置：默认警告时间（30 分钟–3 天）、任务完成震动与强度、测试通知 / 测试震动
- 完成动画：勾选弹跳、进度弧补间、整行淡色闪光

## 安装

从 [Releases](../../releases) 下载 `TaskList.apk`，用文件管理器点开安装；或：

```bash
adb install -r TaskList.apk
```

## 构建

依赖：

| 组件 | 说明 |
| --- | --- |
| JDK 17 | 编译与签名 |
| Android SDK | platform 34 + build-tools 34 |
| aarch64 aapt2 | 官方 build-tools 只提供 x86_64 的 aapt2/zipalign；在 arm64 机器上可用 Termux 打包的 aapt2 |

```bash
export JAVA_HOME=/path/to/jdk17
export ANDROID_SDK=/path/to/android-sdk
export AAPT2=/path/to/aapt2            # 需要时
export AAPT2_LIBS=/path/to/aapt2/libs  # 需要时
./build.sh
```

流程：`aapt2 compile/link` → `javac --release 8` → `d8` → 4 字节对齐打包（`tools/pack.py`，替代 zipalign）→ `apksigner` 签名。
`keystore.jks` 不会入库，首次构建会自动生成一个调试签名。

## 项目结构

```
TaskList/
├── AndroidManifest.xml
├── build.sh                 # 一键构建（不需要 Gradle）
├── tools/pack.py            # dex 注入 + zip 4 字节对齐
├── src/com/dsha/tasks/
│   ├── MainActivity.java    # 任务列表、左滑删除、智能输入
│   ├── TaskDialog.java      # 任务详情
│   ├── HabitActivity.java   # 习惯列表（周视图、拖动排序）
│   ├── HabitDetailActivity.java / HabitDialog.java
│   ├── SettingsActivity.java / Settings.java
│   ├── TaskStore.java / HabitStore.java / Task.java / Habit.java / Category.java
│   ├── SmartDate.java / TimeFmt.java
│   ├── Alarms.java / HabitAlarms.java / ReminderReceiver.java / HabitReminderReceiver.java / BootReceiver.java
│   ├── Reminders.java / Notifications.java / Haptics.java
│   ├── CheckCircle.java / ProgressLine.java / ChartView.java / LineChartView.java / HeatmapView.java
│   └── TaskWidgetProvider.java
└── res/                     # 布局、矢量图标、浅色 / 深色配色
```

## 权限

| 权限 | 用途 |
| --- | --- |
| `POST_NOTIFICATIONS` | 到点发通知（Android 13+ 运行时申请） |
| `SCHEDULE_EXACT_ALARM` | 准点触发；未授权时自动退化为非精确定时 |
| `RECEIVE_BOOT_COMPLETED` | 重启后重排提醒 |
| `VIBRATE` | 完成反馈震动 |

## 常见问题

**提醒不响？** 设置 → 测试通知，能收到说明通道正常；收不到就去系统里打开本应用的通知权限。小米等机型还需要允许「自启动」、省电策略设为「无限制」。

**震动感觉不到？** 设置 → 测试震动；再检查系统的振动强度、触感反馈开关。部分线性马达不支持振幅控制，三档强度差别会变小。

**数据存在哪？** 全部在应用私有目录的 SharedPreferences 里，不上传、不联网，卸载即清除。

## 许可

MIT License，版权归 子岚Velm0ny。联合 DSH 制作。
