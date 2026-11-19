# 03 Topic 通信：用命令和 Python 收发消息，让海龟自己转圈

> 运行环境：VMware 虚拟机 + Ubuntu 24.04 LTS（有图形桌面）+ ROS 2 Jazzy（已按第 1 节装好 `ros-jazzy-desktop`）
> 命令来源：ROS 2 官方教程 [Understanding topics](https://docs.ros.org/en/jazzy/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html) 和 [Writing a simple publisher and subscriber (Python)](https://docs.ros.org/en/jazzy/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Publisher-And-Subscriber.html)，有疑问以官方文档为准。

## 本节目标

Topic（话题）是 ROS 2 里节点之间传数据的"广播频道"。往频道里发消息的叫 **publisher**（发布者），从频道里收消息的叫 **subscriber**（订阅者）。发的人不用管谁在听，听的人也不用管是谁发的。

第 2 节你按方向键控制海龟，背后就是键盘节点往 `/turtle1/cmd_vel` 这个 Topic 发速度。真机器人做导航时也是一样：导航程序往 `/cmd_vel` 发速度，机器人底盘收到后转动轮子。

本节分三步，从"不写代码"到"自己写代码"：

1. 用命令行偷看和冒充 Topic
2. 用 Python 写一个发布者，让海龟自己转圈
3. 用 Python 写一个订阅者，读出海龟的实时位置

**验收标准**：

- 第 1 步：`ros2 topic echo` 能看到海龟位置在刷新；一条 `ros2 topic pub` 命令让海龟转圈
- 第 2 步：运行 `circle_pub.py`，海龟画圆，终端每 0.5 秒打印一行"发布速度"
- 第 3 步：运行 `pose_sub.py`，终端每隔约半秒打印一行"乌龟位置"，数字一直在变

> 怎么操作：按 `Ctrl + Alt + T` 打开终端，**一行一行**复制粘贴进去，回车。终端里粘贴用 `Ctrl + Shift + V`（不是 `Ctrl + V`）。粘贴后先看一眼：开头不能有 `^[[200~`，空格不能少。

## 第 1 步：用命令行看清 Topic

### 运行命令

**终端 1**：启动海龟

```bash
source /opt/ros/jazzy/setup.bash
```

```bash
ros2 run turtlesim turtlesim_node
```

**终端 2**：列出所有 Topic，再偷看海龟位置（看完按 `Ctrl + C` 停掉）

```bash
source /opt/ros/jazzy/setup.bash
```

```bash
ros2 topic list
```

```bash
ros2 topic echo /turtle1/pose
```

**终端 3**：冒充键盘，每秒发一次速度

```bash
source /opt/ros/jazzy/setup.bash
```

```bash
ros2 topic pub --rate 1 /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"
```

### 预期输出

`ros2 topic list`（与第 2 节实测一致）：

```text
/parameter_events
/rosout
/turtle1/cmd_vel
/turtle1/color_sensor
/turtle1/pose
```

终端 3 开始发布后，海龟开始画圈。这时再在终端 2 运行 `ros2 topic echo /turtle1/pose`，会不停刷出（2026-11 实测）：

```text
x: 6.0764851570129395
y: 7.622073173522949
theta: 2.611405849456787
linear_velocity: 2.0
angular_velocity: 1.7999999523162842
---
```

`linear_velocity: 2.0` 和 `angular_velocity: 1.79999...` 正是终端 3 发出去的两个数。1.8 变成 1.7999... 是小数在电脑里存储的正常误差。

### 逐段讲解

- **`ros2 topic list`**：列出当前所有 Topic。`/turtle1/cmd_vel` 是海龟收速度指令的频道，`/turtle1/pose` 是海龟往外报自己位置的频道。
- **`ros2 topic echo /turtle1/pose`**：把某个 Topic 里的消息原样打印出来，用来"偷看"频道里在传什么。`x`、`y` 是坐标，`theta` 是朝向角（单位是弧度），后两行是当前速度。
- **`ros2 topic pub`**：往某个 Topic 发消息，相当于"冒充"一个发布者。
  - `--rate 1`：每秒发 1 次
  - `/turtle1/cmd_vel`：发到哪个频道
  - `geometry_msgs/msg/Twist`：消息类型。**Twist** 是 ROS 里表示"线速度 + 角速度"的标准格式，真机器人的 `/cmd_vel` 也用它
  - 引号里的内容：具体数值。`linear.x: 2.0` 是往前走，`angular.z: 1.8` 是绕竖直方向转。又走又转，就画成了圆

## 第 2 步：用 Python 写一个发布者

先在终端 3 按 `Ctrl + C` 停掉上一步的 `ros2 topic pub`（不然两个发布者同时在发）。终端 1 的海龟窗口保持开着。

### 完整代码

建文件夹，并用 Ubuntu 自带的文本编辑器新建文件：

```bash
mkdir -p ~/ros2_demo
```

```bash
gnome-text-editor ~/ros2_demo/circle_pub.py
```

把下面的代码粘进编辑器，`Ctrl + S` 保存后关掉：

```python
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import Twist


class CirclePublisher(Node):
    def __init__(self):
        super().__init__('circle_publisher')
        self.publisher_ = self.create_publisher(Twist, '/turtle1/cmd_vel', 10)
        self.timer = self.create_timer(0.5, self.timer_callback)

    def timer_callback(self):
        msg = Twist()
        msg.linear.x = 2.0
        msg.angular.z = 1.8
        self.publisher_.publish(msg)
        self.get_logger().info(f'发布速度: 前进 {msg.linear.x}, 转向 {msg.angular.z}')


def main():
    rclpy.init()
    node = CirclePublisher()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.try_shutdown()


if __name__ == '__main__':
    main()
```

> Python 靠**缩进**（行首的空格）分层级，粘贴后不要改动行首空格。

### 运行命令

```bash
ls ~/ros2_demo
```

先确认文件存在（应打印 `circle_pub.py`），再运行：

```bash
python3 ~/ros2_demo/circle_pub.py
```

### 预期输出

海龟开始画圆，终端每 0.5 秒打印一行（2026-11 实测）：

```text
[INFO] [1795102481.346005418] [circle_publisher]: 发布速度: 前进 2.0, 转向 1.8
[INFO] [1795102481.845978806] [circle_publisher]: 发布速度: 前进 2.0, 转向 1.8
[INFO] [1795102482.346259934] [circle_publisher]: 发布速度: 前进 2.0, 转向 1.8
```

中括号里的长数字是时间戳，相邻两行正好差 0.5 秒。按 `Ctrl + C` 程序退出，海龟过一会儿停下。

### 逐段讲解

- **开头三行 `import`**：把要用的工具拿进来。`rclpy` 是 ROS 2 的 Python 库（rcl = ROS Client Library，py = Python）；`Node` 是节点的模板；`Twist` 就是第 1 步命令里的 `geometry_msgs/msg/Twist` 消息类型。
- **`class CirclePublisher(Node)`**：定义我们自己的节点。括号里的 `Node` 表示"在 ROS 节点模板的基础上改"。
  - `super().__init__('circle_publisher')`：给节点起名叫 `circle_publisher`，日志里中括号显示的就是它。
  - `create_publisher(Twist, '/turtle1/cmd_vel', 10)`：申请一个发布者，三个参数依次是**消息类型、Topic 名字、队列长度**。队列长度 10 的意思是"来不及发的消息最多先排 10 条"，入门阶段照写 10 即可。
  - `create_timer(0.5, self.timer_callback)`：建一个定时器，每 0.5 秒自动调用一次 `timer_callback`。**callback**（回调函数）就是"你先写好，等某件事发生时由系统替你调用"的函数。
- **`timer_callback`**：每次被调用做三件事：新建一条空的 `Twist` 消息，填上前进速度 2.0 和转向速度 1.8，然后 `publish` 发出去，并用 `get_logger().info` 打印一行日志。
- **`main()`**：程序入口。
  - `rclpy.init()`：启动 ROS 2 的通信功能，必须最先调用。
  - `rclpy.spin(node)`：让节点一直运行，不停检查定时器到没到点。没有这一行，程序会直接结束。
  - `try / except KeyboardInterrupt`：按 `Ctrl + C` 时会产生 `KeyboardInterrupt`，这里接住它，让程序安静退出，不打印一大串报错。
  - `finally` 里两行是收尾：销毁节点，关闭 ROS 2。
- **`if __name__ == '__main__':`**：Python 的固定写法，意思是"直接运行这个文件时，执行 `main()`"。

> 说明：正规做法是把代码放进 ROS 2 的 package（功能包）里，用 `ros2 run` 启动。那需要先学工作空间和编译，后面单独一节讲。本节先用 `python3` 直接运行，专心理解 Topic。

## 第 3 步：用 Python 写一个订阅者

保持终端 1（海龟）和终端 2（`circle_pub.py`）都在运行，让海龟继续转圈。再开一个终端（`Ctrl + Shift + T` 可以在同一窗口新开标签页）。

### 完整代码

```bash
gnome-text-editor ~/ros2_demo/pose_sub.py
```

```python
import rclpy
from rclpy.node import Node
from turtlesim.msg import Pose


class PoseSubscriber(Node):
    def __init__(self):
        super().__init__('pose_subscriber')
        self.subscription = self.create_subscription(
            Pose, '/turtle1/pose', self.pose_callback, 10)
        self.count = 0

    def pose_callback(self, msg):
        self.count += 1
        if self.count % 30 == 0:
            self.get_logger().info(
                f'乌龟位置: x={msg.x:.2f}, y={msg.y:.2f}, 朝向={msg.theta:.2f}')


def main():
    rclpy.init()
    node = PoseSubscriber()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.try_shutdown()


if __name__ == '__main__':
    main()
```

### 运行命令

```bash
source /opt/ros/jazzy/setup.bash
```

```bash
python3 ~/ros2_demo/pose_sub.py
```

### 预期输出

2026-11 实测：

```text
[INFO] [1795102699.612044601] [pose_subscriber]: 乌龟位置: x=6.64, y=6.72, 朝向=1.61
[INFO] [1795102700.091211219] [pose_subscriber]: 乌龟位置: x=6.20, y=7.54, 朝向=2.48
[INFO] [1795102700.572299568] [pose_subscriber]: 乌龟位置: x=5.29, y=7.74, 朝向=-2.94
[INFO] [1795102701.050844584] [pose_subscriber]: 乌龟位置: x=4.55, y=7.18, 朝向=-2.08
```

大约每 0.48 秒打印一行，x、y、朝向一直在变。

两个值得注意的现象：

- **朝向从 2.48 跳到了 -2.94**：不是出错。`theta` 的范围是 -π 到 π（约 -3.14 到 3.14），转过 3.14 后就从 -3.14 接着算，就像钟表过了 12 点又从 1 点开始。
- **每 30 条才打印 1 条，却约 0.48 秒就打印一次**：说明海龟每秒大约发 60 多次位置。所以代码里要"抽样"打印，不然会刷屏。

### 逐段讲解（只讲和发布者不一样的地方）

- **`from turtlesim.msg import Pose`**：`Pose`（位姿，即"位置 + 朝向"）是 turtlesim 自己定义的消息类型，字段就是第 1 步 `echo` 看到的 `x`、`y`、`theta` 等。
- **`create_subscription(Pose, '/turtle1/pose', self.pose_callback, 10)`**：申请一个订阅者，四个参数依次是**消息类型、Topic 名字、收到消息时调用的函数、队列长度**。发布者是"定时器到点就发"，订阅者是"**消息来了**系统就自动调用 `pose_callback`"，并把消息作为 `msg` 传进来。
- **`count` 和 `%`**：`count` 记录收到了多少条。`%` 是取余数，`count % 30 == 0` 表示"count 是 30 的倍数时"才打印。
- **`{msg.x:.2f}`**：把小数保留 2 位再打印。
- **`main()`**：和发布者一模一样。

### 小实验

在运行 `circle_pub.py` 的终端按 `Ctrl + C` 停掉发布者，观察订阅者打印的数字：海龟停下后，x、y、朝向不再变化，但订阅者照样在收消息。这说明海龟一直在报位置，不管有没有人给它发速度；发布者和订阅者互不依赖。

## 常见报错与排查

| 现象 | 原因 | 解决 |
|---|---|---|
| `ros2: command not found` | 当前终端没 source | `source /opt/ros/jazzy/setup.bash` |
| `ModuleNotFoundError: No module named 'rclpy'` | 运行 Python 的那个终端没 source | 先 source，再 `python3 ...` |
| `ros2 topic pub` 报格式错误，或海龟不动 | 粘贴时丢了空格（冒号后面必须有空格），或开头多了 `^[[200~` | 用 `Ctrl + Shift + V` 粘贴，只粘一行，检查 `x: 2.0` 这样的冒号后空格 |
| `IndentationError` | 代码缩进被改乱了 | 重新完整粘贴代码，不要手动改行首空格 |
| `python3: can't open file ... No such file or directory` | 文件没保存，或路径/文件名打错了 | `ls ~/ros2_demo` 看文件在不在、名字对不对 |
| 程序在跑、日志也在打印，海龟却不动 | Topic 名字写错（如漏了 `/turtle1`），或海龟窗口已关闭 | 对照 `ros2 topic list` 检查名字；确认终端 1 还开着 |
| 海龟走得乱七八糟 | 上一步的 `ros2 topic pub` 没停，两个发布者同时在发 | 到对应终端按 `Ctrl + C` |
| 运行 `gnome-text-editor` 时刷出一堆 `libEGL warning`、`MESA error`、`Gtk-WARNING` | 虚拟机没有真显卡，编辑器在提示图形加速用不了（2026-11 实测） | 不影响保存文件，可以忽略 |
| 订阅者没有任何输出 | 海龟没启动，或 Topic 名字写错 | 确认终端 1 海龟窗口在；`ros2 topic echo /turtle1/pose` 能看到数据再跑 |

## 一句话速查

```bash
source /opt/ros/jazzy/setup.bash
ros2 topic list                    # 列出所有话题
ros2 topic echo /turtle1/pose      # 偷看话题里的消息
ros2 topic pub --rate 1 /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"   # 往话题发消息
python3 ~/ros2_demo/circle_pub.py  # 自己写的发布者：create_publisher + timer + publish
python3 ~/ros2_demo/pose_sub.py    # 自己写的订阅者：create_subscription + callback
```
