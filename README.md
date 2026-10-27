# Lab4 二维激光雷达 ROS 2 实验

课程源码来源：https://gitee.com/yapengshi/lidar_pkg_ros2.git ，基线提交 `eb12ec8`。

## 实验环境

香橙派 AI Pro 20T，Ubuntu 22.04.3 LTS，ROS 2 Humble，系统 Python `/usr/bin/python3`。
雷达串口 `/dev/ttyACM0`，消息坐标系 `laser`。当前账户需有串口访问权限。

## 编译与运行

将本仓库放在 `~/ros2_ws/src/lidar_pkg_ros2` 下，在使用系统 Python 的终端运行：

```bash
export ROS_DOMAIN_ID=11
source /opt/ros/humble/setup.bash
cd ~/ros2_ws
colcon build --symlink-install
source install/setup.bash
ros2 launch lidar_pkg monitor.launch.py
```

同一雷达同时只运行一个驱动。另一终端使用相同通信域与环境后观察：

```bash
ros2 node list --no-daemon
ros2 topic list --no-daemon
ros2 topic info /scan --no-daemon
timeout 12 ros2 topic hz /scan
```

## 配置与实测结果

`config/lidar_params.yaml` 中半径为 0.10 m，最少邻点数为 2（含自身）。
将 `filter.enabled` 依次设为 false、true，每次停止并重启驱动。
当前提交保存 B 组开启滤波的配置。相对课程基线，将 min_neighbors 从 3 调为 2。

| 条件 | 三帧有效槽数 | 平均有效槽数 | topic hz 最后读数 |
|---|---|---|---|
| A 关闭滤波 | 306、296、297 | 299.67 | 5.238 Hz |
| B 开启滤波 | 258、254、250 | 254.00 | 5.243 Hz |

每帧 ranges 有 720 个方向槽；无有效距离的槽为 inf。两组先后采样，环境及回波波动也会影响计数差异。
驱动中的 scan_time=1/7 是预设时间字段，不代表实际接收频率为 7 Hz。
