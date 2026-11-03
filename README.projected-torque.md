# 投影力矩分支

默认产品为 `User/xrobot.yaml`，配置来自 luyan2 的完整机器人配置。
舵轮控制包括正逆运动学、静止锁角、cos³ 补偿、底盘速度闭环和力矩前馈。
Helm 使用文档中的多边形裁剪、功率椭圆切点及边界交点方法求解加速度约束。
只约束四个驱动轮，电机功率带符号求和，制动负功率不截 0；舵向电机不计入预算。
PowerControl 保留原有在线辨识，增加只读模型快照接口供求解器使用。
打滑模型使用 μ=0.35；普通模式归一化权重为平动:旋转=1:1，ROTOR 为 1:4。
裁判离线使用 HELM_CHASSIS_MAX_POWER，超电沿用在线 BOOST 的 40/70/100 W 分档。
额外功率估算及 Ozone 采样变量已删除。
随动模式使用 2° 角度死区；独立和旋转模式的角速度由摇杆 z 指令控制。

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
`Modules/modules.yaml` 指定它们的“投影力矩”分支；PowerControl 的 Fork 保留 QDU
上游辨识与限流实现，并增加约束求解所需的只读模型接口。`xrobot.lock` 记录精确提交。
新克隆无需手动覆盖模块文件。

Motor 的 Fork 名为 `Motor-projected-torque`，保留账户中原有的 `Motor` 仓库。
其逻辑 ID 仍为 `QDU-Robomaster/Motor`，因此既有模块的依赖声明保持有效。

## 上板检查

已通过代码生成、依赖锁定检查和 Debug 编译；尚未验证实车。
先在无弹、轮子架空的条件下核对轮序、零位、方向、拨杆切换和失控放松，再做低速地面测试。
当前默认配置左拨杆从上到下为 RELAX、FOLLOW、ROTOR；右拨杆为云台 RELAX、COMMON、COMMON。
确认超电通信协议与功率计覆盖范围后，再使用在线功率模型辨识。
模块内的 `HelmMigrationRecord.md` 记录了参数、控制约定和详细验证步骤。

## 摩擦系数测量

当前轮地摩擦系数实测约为 0.4，控制约束按用户选择保守使用 0.35。抓地模型假定四轮等载。
实车仍需验证模型误差、制动回馈能力及最坏求解耗时是否满足 2 ms 周期。
