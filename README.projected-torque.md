# 投影力矩分支

默认产品为 `User/xrobot.yaml`，配置来自 luyan2 的完整机器人配置。
舵轮控制包括正逆运动学、静止锁角、cos³ 补偿、底盘速度闭环和力矩前馈。
功率估算与限幅只统计四个驱动轮，舵向保持 PID 输出及独立扭矩限幅。

## 获取和构建

```powershell
git clone --recurse-submodules --branch "投影力矩" https://github.com/KaMi-0932/bsp-dev-c.git
cd bsp-dev-c
python -m pip install "xrobot==1.0.1" "libxr==6.0.1"
xrobot setup --frozen
xrobot gen -c User/xrobot.yaml
cmake --preset debug
cmake --build --preset debug
```

构建需要 CMake、Ninja 和 starm-clang 已加入当前终端的 PATH。
输出为 `build/debug/DevC.elf`、`DevC.bin` 和 `DevC.hex`。

## 模块版本

`Modules/projected-torque-index.yaml` 将五个模块的逻辑 ID 映射到 KaMi-0932
账户下的 Fork，`Modules/sources.yaml` 为这份项目内索引设置更高优先级。
`Modules/modules.yaml` 指定它们的“投影力矩”分支，`xrobot.lock` 记录精确提交。
新克隆无需手动覆盖模块文件。

Motor 的 Fork 名为 `Motor-projected-torque`，保留账户中原有的 `Motor` 仓库。
其逻辑 ID 仍为 `QDU-Robomaster/Motor`，因此既有模块的依赖声明保持有效。

## 上板检查

当前验证覆盖代码生成、Debug 编译和驱动轮功率限幅的主机测试，尚未验证实车。
先在无弹、轮子架空的条件下核对轮序、零位、方向、拨杆切换和失控放松，再做低速地面测试。
左拨杆从上到下为 RELAX、FOLLOW、INDEPENDENT；右拨杆为云台 RELAX、COMMON、COMMON。
确认功率计覆盖驱动轮支路后，再使用在线功率模型辨识。
模块内的 `HelmMigrationRecord.md` 记录了参数、控制约定和详细验证步骤。
