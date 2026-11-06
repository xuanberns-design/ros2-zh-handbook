# 01 环境安装：在 Ubuntu 24.04 上安装 ROS 2 Jazzy

> 运行环境：VMware 虚拟机 + Ubuntu 24.04 LTS（有图形桌面）+ ROS 2 Jazzy
> 命令来源：ROS 2 官方安装文档 [Ubuntu (deb packages)](https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html)，有疑问以官方文档为准。

## 本节目标

用 `apt`（Ubuntu 的软件安装工具，相当于手机的应用商店）装好 ROS 2 Jazzy，并跑通官方自带的"一个程序说话、另一个程序收听"的例子。

**验收标准**：开两个终端，一个不停打印 `Publishing: 'Hello World: 1'`、`2`、`3`……，另一个同步打印 `I heard: [Hello World: 1]`……。看到这个就算装好了。

## 完整步骤与运行命令

> 怎么操作：按 `Ctrl + Alt + T` 打开终端，**一行一行**复制粘贴进去，回车。终端里粘贴用 `Ctrl + Shift + V`（不是 `Ctrl + V`）。
> 遇到要输密码时，输入你登录 Ubuntu 的密码。输密码时屏幕上不显示任何字符，这是正常的，输完直接回车。

### 第 1 步：检查语言编码（locale）

```bash
locale
```

`locale` 是系统的"语言和编码设置"。看输出里 `LANG=` 那一行：

- 如果结尾是 `UTF-8`（比如 `LANG=zh_CN.UTF-8`），**这一步就完成了，直接跳到第 2 步**。官方文档说明任何支持 UTF-8 的设置都可以。
- 如果不是 UTF-8，先把输出发给我，我们再处理（官方给的修改命令会把系统语言改成英文，没必要的话不改）。

### 第 2 步：打开 Ubuntu 的 universe 软件仓库

```bash
sudo apt install software-properties-common
sudo add-apt-repository universe
```

- `sudo`：用管理员权限执行后面的命令，所以会让你输密码。
- 软件仓库（repository）：可以理解为一个"软件下载源"。universe 是 Ubuntu 官方维护的一个扩展仓库，ROS 2 依赖里面的一些软件。
- 如果问你 `Press [ENTER] to continue`，按回车。

### 第 3 步：添加 ROS 2 的软件仓库

```bash
sudo apt update && sudo apt install curl -y
```

- `apt update`：刷新"有哪些软件可以装"的清单。
- `curl`：一个从网上下载东西的命令行工具。`-y` 表示遇到"是否继续"自动回答 yes。

```bash
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F'"' '{print $4}')
echo $ROS_APT_SOURCE_VERSION
```

- 这一行去 GitHub 查"ROS 仓库配置包"的最新版本号，存进一个叫 `ROS_APT_SOURCE_VERSION` 的变量里。
- `echo` 把这个变量打印出来。**应该看到一个类似 `1.1.0` 的版本号。如果打印出来是空行，说明没连上 GitHub，先停下来把情况告诉我。**

```bash
curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})_all.deb"
sudo dpkg -i /tmp/ros2-apt-source.deb
```

- 第一行：按刚才查到的版本号，下载一个 `.deb` 安装包（Ubuntu 的安装包格式，类似 Windows 的 `.exe`）到 `/tmp` 文件夹。
- 第二行：`dpkg -i` 安装这个包。它的作用是告诉你的系统"以后可以从 ROS 官方仓库下载软件"，并装好用来验证下载内容没被篡改的密钥。

### 第 4 步：安装 ROS 2 Jazzy

```bash
sudo apt update
sudo apt upgrade
sudo apt install ros-jazzy-desktop
```

- `apt upgrade`：先把系统里已有的软件更新到最新，官方建议这么做，避免版本冲突。中途问 `[Y/n]` 时输入 `Y` 回车。
- `ros-jazzy-desktop`：ROS 2 Jazzy 的"桌面完整版"，包含核心功能、图形工具 RViz、示例程序和教程（turtlesim 小海龟也在里面）。
- 这一步要下载几百 MB 到一两 GB，**可能需要十几分钟到半小时**，耐心等它跑完、重新出现 `$` 提示符。

### 第 5 步：验收测试

打开**第一个终端**，输入：

```bash
source /opt/ros/jazzy/setup.bash
ros2 run demo_nodes_cpp talker
```

再按 `Ctrl + Alt + T` 打开**第二个终端**，输入：

```bash
source /opt/ros/jazzy/setup.bash
ros2 run demo_nodes_py listener
```

两个都跑起来后，在各自终端按 `Ctrl + C` 停止。

## 预期输出

第一个终端（talker）：

```text
[INFO] [...] [talker]: Publishing: 'Hello World: 1'
[INFO] [...] [talker]: Publishing: 'Hello World: 2'
...
```

第二个终端（listener）：

```text
[INFO] [...] [listener]: I heard: [Hello World: 1]
[INFO] [...] [listener]: I heard: [Hello World: 2]
...
```

方括号里的 `...` 是时间戳，每个人不一样。数字可能对不齐（listener 晚启动会错过前几条），没关系，只要 listener 在持续打印 `I heard` 就成功了。

## 逐段讲解

- **`source /opt/ros/jazzy/setup.bash`**：ROS 2 装在 `/opt/ros/jazzy` 这个文件夹里。终端默认不知道它在哪，`source` 这个文件就是告诉**当前这个终端**"ROS 2 的命令和程序都在这里"。**每打开一个新终端都要执行一次**，否则会报 `ros2: command not found`。
- **`ros2 run <包名> <程序名>`**：运行某个 package（包，ROS 里组织代码的单位，可以理解为一个文件夹，里面装着相关的程序）里的某个可执行程序。
- **talker / listener**：talker 每秒发一条消息，listener 接收并打印。它们一个用 C++ 写（`demo_nodes_cpp`），一个用 Python 写（`demo_nodes_py`），能互相通信，说明 ROS 2 的 C++ 和 Python 两套接口都装好了。这种"一个发、一个收"的通信方式叫 Topic（话题），是第 3 节的主角。

## 常见报错与排查

| 现象 | 原因 | 解决 |
|---|---|---|
| `echo $ROS_APT_SOURCE_VERSION` 打印空行 | 虚拟机连不上 GitHub（国内网络常见） | 用虚拟机里的 Firefox 打开 `https://github.com/ros-infrastructure/ros-apt-source/releases`，在最新版本的 Assets 里下载 `ros2-apt-source_<版本号>.noble_all.deb`，再运行 `sudo apt install ~/下载/ros2-apt-source_*.noble_all.deb`（`~/下载` 是中文系统的下载文件夹，英文系统是 `~/Downloads`），然后 `sudo apt update`，输出里有 `packages.ros.org` 即成功 |
| `curl -sI https://api.github.com` 返回 `HTTP/2 403` | 网络通了，但 GitHub 接口对未登录访问有次数限制，共用的 VPN 出口 IP 常被用光（推测，可用 `curl -s https://api.github.com/rate_limit` 确认） | 改用网页跳转取版本号：`export ROS_APT_SOURCE_VERSION=$(curl -sI https://github.com/ros-infrastructure/ros-apt-source/releases/latest \| grep -i '^location:' \| sed 's\|.*/tag/\|\|' \| tr -d '\r')`（2026-11 实测得到 `1.3.0`） |
| `dpkg: error: cannot access archive` | 上一步版本号为空或下载失败，`.deb` 文件不存在 | 回到第 3 步，确认版本号不是空的再下载 |
| `Unable to locate package ros-jazzy-desktop` | ROS 仓库没加成功，或者加完没 `apt update` | 重新执行 `sudo apt update`，看输出里有没有 `packages.ros.org` 字样 |
| `apt update` 很慢（只有几十 kB/s），或输出里出现 `noble-proposed` | 用的是默认官方源，国内慢；`proposed` 是"预发布测试"仓库，新手不该开 | 备份后改 `/etc/apt/sources.list.d/ubuntu.sources`：去掉 `noble-proposed`，把地址换成国内镜像（如清华 `https://mirrors.tuna.tsinghua.edu.cn/ubuntu`），见下方「附录：关掉 proposed 并换国内镜像源」 |
| `Could not get lock /var/lib/dpkg/lock` | 系统在后台自动更新，占着 apt | 等几分钟再试；或者重启虚拟机 |
| `ros2: command not found` | 当前终端没执行 `source` | 执行 `source /opt/ros/jazzy/setup.bash` |
| listener 什么都不打印 | 另一个终端的 talker 没在运行，或某个终端没 `source` | 确认两个终端都 source 了、talker 在持续打印 |

## 附录：关掉 proposed 并换国内镜像源

适用于 Ubuntu 24.04（它的软件源配置文件是 `/etc/apt/sources.list.d/ubuntu.sources`）。

```bash
# 1. 备份原文件，改坏了可以恢复
sudo cp /etc/apt/sources.list.d/ubuntu.sources /etc/apt/sources.list.d/ubuntu.sources.bak
# 2. 去掉 noble-proposed（预发布测试仓库）
sudo sed -i 's/ noble-proposed//' /etc/apt/sources.list.d/ubuntu.sources
# 3. 不下载源代码清单（deb-src），减少下载量
sudo sed -i 's/^Types: deb deb-src$/Types: deb/' /etc/apt/sources.list.d/ubuntu.sources
# 4. 把官方源地址换成清华镜像
sudo sed -i 's|http://cn.archive.ubuntu.com/ubuntu|https://mirrors.tuna.tsinghua.edu.cn/ubuntu|g' /etc/apt/sources.list.d/ubuntu.sources
# 5. 检查结果
cat /etc/apt/sources.list.d/ubuntu.sources
grep -rn proposed /etc/apt/sources.list /etc/apt/sources.list.d/
sudo apt update
```

- `sed -i 's/旧/新/'`：在文件里把"旧"文字替换成"新"文字，`-i` 表示直接改文件。
- 第 5 步的 `grep` 应该什么都不输出，表示 proposed 已经没有了。
- 恢复原样：`sudo cp /etc/apt/sources.list.d/ubuntu.sources.bak /etc/apt/sources.list.d/ubuntu.sources`
- 清华镜像的官方说明在 https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/ ，可以去那里核对。

## 一句话速查

```bash
# Ubuntu 24.04 安装 ROS 2 Jazzy 后，每个新终端先执行：
source /opt/ros/jazzy/setup.bash
# 验证：一个终端 ros2 run demo_nodes_cpp talker，另一个 ros2 run demo_nodes_py listener
```
