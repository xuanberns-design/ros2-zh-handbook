# 02 turtlesim：用键盘控制小海龟

> 运行环境：VMware 虚拟机 + Ubuntu 24.04 LTS（有图形桌面）+ ROS 2 Jazzy（已按第 1 节装好 `ros-jazzy-desktop`）
> 命令来源：ROS 2 官方教程 [Using turtlesim, ros2, and rqt](https://docs.ros.org/en/jazzy/Tutorials/Beginner-CLI-Tools/Introducing-Turtlesim/Introducing-Turtlesim.html)，有疑问以官方文档为准。

## 本节目标

打开 turtlesim（小海龟仿真器，ROS 2 官方的入门小程序），再开一个键盘控制程序，用方向键让海龟在窗口里跑。

turtlesim 是一个极简的"假机器人"：它没有真实的电机和传感器，但和真机器人一样，是靠 ROS 2 的程序之间互相发消息来动的。后面学 Topic、Service、参数都会拿它做实验。

**验收标准**：按键盘方向键，海龟会移动，并在身后画出一条线。

## 完整步骤与运行命令

> 怎么操作：按 `Ctrl + Alt + T` 打开终端，**一行一行**复制粘贴进去，回车。终端里粘贴用 `Ctrl + Shift + V`（不是 `Ctrl + V`）。
> 本节不用写代码，用的是 ROS 2 自带的程序。

### 第 1 步：打开小海龟窗口（第一个终端）

```bash
source /opt/ros/jazzy/setup.bash
```

```bash
ros2 pkg executables turtlesim
```

先确认 turtlesim 装好了，应该列出 4 个程序（见下方预期输出）。如果提示找不到包，用 `sudo apt install ros-jazzy-turtlesim` 安装。

```bash
ros2 run turtlesim turtlesim_node
```

会弹出一个蓝色背景、中间有只小海龟的窗口。**这个终端不要关**，关掉海龟窗口也会消失。

### 第 2 步：用键盘控制（第二个终端）

再按 `Ctrl + Alt + T` 打开**一个新终端**。新终端不认识 ROS 2，要重新 source：

```bash
source /opt/ros/jazzy/setup.bash
```

```bash
ros2 run turtlesim turtle_teleop_key
```

然后**用鼠标点一下这个终端**（让它成为当前窗口），按方向键：

| 按键 | 作用 |
|---|---|
| ↑ / ↓ | 前进 / 后退 |
| ← / → | 原地左转 / 右转 |
| `q` | 退出键盘控制程序 |

建议把海龟窗口和这个终端拖开并排放，边按边看。

### 结束

在两个终端里分别按 `Ctrl + C`（或在键盘控制终端按 `q`）停止程序。

## 预期输出

`ros2 pkg executables turtlesim`：

```text
turtlesim draw_square
turtlesim mimic
turtlesim turtle_teleop_key
turtlesim turtlesim_node
```

第一个终端（`turtlesim_node`）：

```text
[INFO] [...] [turtlesim]: Starting turtlesim with node name /turtlesim
[INFO] [...] [turtlesim]: Spawning turtle [turtle1] at x=[5.544445], y=[5.544445], theta=[0.000000]
```

第二个终端（`turtle_teleop_key`）：

```text
Reading from keyboard
---------------------------
Use arrow keys to move the turtle.
Use g|b|v|c|d|e|r|t keys to rotate to absolute orientations. 'f' to cancel a rotation.
'q' to quit.
```

海龟窗口：按方向键时海龟移动，身后留下一条白线。每按一次只走一小段就停下，这是正常的（见下方讲解）。

## 逐段讲解

- **`source /opt/ros/jazzy/setup.bash`**：告诉**当前终端** ROS 2 装在哪。每个新终端都要执行一次，第 1 节讲过。
- **`ros2 pkg executables turtlesim`**：列出 turtlesim 这个 package（包）里有哪些可执行程序。`pkg` 是 package 的缩写。这是检查"某个包装没装好"的常用命令。
- **`ros2 run turtlesim turtlesim_node`**：运行 turtlesim 包里的 `turtlesim_node` 程序，它负责画窗口、画海龟。ROS 2 里每个这样运行起来的程序叫一个 node（节点），可以理解为机器人系统里分工干活的一个"小程序"。输出里的 `/turtlesim` 就是这个节点的名字，`turtle1` 是海龟的名字，`x`、`y` 是它出生的坐标，`theta` 是朝向角度。
- **`ros2 run turtlesim turtle_teleop_key`**：运行键盘控制节点。teleop 是 teleoperation（遥控）的缩写。它读取你的按键，换算成"往前走多快、转多快"的速度指令，发给 `turtlesim_node`。
- **为什么要点一下终端**：按键是被"当前选中的窗口"接收的。键盘控制程序在终端里读按键，所以必须选中这个终端，而不是海龟窗口。
- **为什么按一下只走一小段**：官方解释是出于安全考虑。真实机器人如果和遥控断开了连接，不应该一直按最后一条指令跑下去。
- **两个程序是怎么配合的**：键盘节点不断把速度指令"发布"出去，海龟节点"订阅"这些指令并让海龟动。这种一发一收的通信方式就是 Topic（话题），和第 1 节的 talker/listener 是同一回事，第 3 节会详细拆开看。

## 小加餐：看看海龟背后有什么在运行

保持上面两个终端都在运行，再开**第三个终端**：

```bash
source /opt/ros/jazzy/setup.bash
```

```bash
ros2 node list
```

预期输出（2026-11 实测）：

```text
/teleop_turtle
/turtlesim
```

`ros2 node list` 列出当前所有正在运行的节点。`/turtlesim` 是画海龟的节点，`/teleop_turtle` 是读按键的节点。

```bash
ros2 topic list
```

预期输出（2026-11 实测，与官方教程一致）：

```text
/parameter_events
/rosout
/turtle1/cmd_vel
/turtle1/color_sensor
/turtle1/pose
```

`ros2 topic list` 列出当前所有的话题（Topic，节点之间传消息的通道，像一个有名字的广播频道）。重点是 `/turtle1/cmd_vel`：cmd_vel 是 command velocity（速度指令）的缩写，你每按一次方向键，`/teleop_turtle` 就往这里发一条速度指令，`/turtlesim` 收到后让海龟移动。其余几个话题第 3 节再讲。

## 常见报错与排查

| 现象 | 原因 | 解决 |
|---|---|---|
| `ros2: command not found` | 当前终端没 source | 执行 `source /opt/ros/jazzy/setup.bash` |
| `Package 'turtlesim' not found` | 没装 turtlesim（装的不是 desktop 版时会出现） | `sudo apt install ros-jazzy-turtlesim`，装完重新 source |
| 启动时打印 `QSocketNotifier: Can only be used with threads started with QThread` | 图形界面库（Qt）打印的警告，2026-11 实测在 VMware + Ubuntu 24.04 上会出现 | 不影响使用，只要海龟窗口正常出现就可以忽略 |
| 按方向键海龟不动 | 选中的是海龟窗口或别的窗口，不是键盘控制的终端 | 用鼠标点一下运行 `turtle_teleop_key` 的终端再按 |
| 按方向键海龟不动，终端也没反应 | 键盘控制终端没 source，或 `turtlesim_node` 那个终端被关掉了 | 确认两个终端都 source 了、海龟窗口还开着 |
| 终端里出现 `^[[A`、`^[[B` 之类字符 | 方向键被发给了普通命令行，而不是正在运行的键盘控制程序（程序已退出或没启动） | 重新运行 `ros2 run turtlesim turtle_teleop_key` |
| 海龟窗口找不到 | 被终端窗口挡住了 | 把终端拖开，或在左侧任务栏找 TurtleSim 窗口 |

## 一句话速查

```bash
# 终端 1：打开小海龟
source /opt/ros/jazzy/setup.bash
ros2 run turtlesim turtlesim_node
# 终端 2：键盘控制（选中这个终端再按方向键，q 退出）
source /opt/ros/jazzy/setup.bash
ros2 run turtlesim turtle_teleop_key
# 终端 3：查看节点和话题
ros2 node list
ros2 topic list
```
