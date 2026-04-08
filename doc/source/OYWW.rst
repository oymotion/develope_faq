OYWW1000 开发文档
=====================

概述
------

支持的数据类型
~~~~~~~~~~~~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 20 25 10 15 30

   * - 数据类型
     - DataType 枚举
     - 单位
     - 通道数
     - 说明
   * - 加速度（ACC）
     - ``DataType.NTF_ACC``
     - g
     - 3（X/Y/Z）
     - 三轴加速度
   * - 陀螺仪（GYRO）
     - ``DataType.NTF_GYRO``
     - °/s
     - 3（X/Y/Z）
     - 三轴角速度
   * - 四元数（Quaternion）
     - ``DataType.NTF_QUATERNION``
     - 
     - 4（W/X/Y/Z）
     - 姿态四元数
   * - 肌电（EMG）
     - ``DataType.NTF_EMG``
     - µV
     - 8-CH
     - 肌电信号
   * - 阻抗（Impedance）
     - ``DataType.NTF_IMPEDANCE``
     - Ω
     - 8-CH
     - 电极阻抗

系统要求与安装
~~~~~~~~~~~~~~~~~~~~~

- Python >= 3.9
- 操作系统：Windows / macOS / Linux
- 依赖库：``bleak``（BLE通信）、``numpy``（数据处理）

.. code-block:: bash

    pip install sensor-sdk

API 参考
----------

SensorController
~~~~~~~~~~~~~~~~

SensorController 是单例类，负责蓝牙设备扫描和 SensorProfile 生命周期管理。

属性
^^^^^^

.. list-table::
   :header-rows: 1
   :widths: 30 15 55

   * - 属性
     - 类型
     - 说明
   * - isScanning
     - bool
     - 只读，当前是否正在扫描
   * - isEnable
     - bool
     - 只读，蓝牙是否可用（始终返回 True）
   * - hasDeviceFoundCallback
     - bool
     - 只读，是否已设置设备发现回调

回调设置
^^^^^^^^^^^^^^^^

**onDeviceFoundCallback**

设置设备发现回调函数，在 startScan() 扫描期间周期性触发。

参数：
  - ``device_list`` (list[BLEDevice]) — 当前未连接的设备列表

.. code-block:: python

    def on_device_found(device_list):
        for device in device_list:
            print(f"{device.Name} - {device.Address} - RSSI: {device.RSSI}")

    controller.onDeviceFoundCallback = on_device_found

方法
^^^^

**startScan(periodInMs: int) -> bool**

启动周期性扫描，每隔 periodInMs 毫秒触发一次 onDeviceFoundCallback。

参数：
  - ``periodInMs`` — 回调周期（毫秒）

返回：True 表示启动成功；重复调用会复用已启动的扫描

.. code-block:: python

    success = controller.startScan(5000)

----

**stopScan() -> None**

停止周期性扫描。

.. code-block:: python

    controller.stopScan()

----

**scan(period: int) -> list[BLEDevice]**

阻塞式一次性扫描，扫描 period 毫秒后返回设备列表。

参数：
  - ``period`` — 扫描时长（毫秒）

返回：list[BLEDevice] — 发现的设备列表

.. code-block:: python

    devices = controller.scan(5000)

----

**requireSensor(device: BLEDevice) -> SensorProfile | None**

根据 BLEDevice 创建或获取对应的 SensorProfile。

参数：
  - ``device`` — 蓝牙设备对象

返回：SensorProfile 实例，失败返回 None；同一设备多次调用返回同一实例

.. code-block:: python

    sensor = controller.requireSensor(ble_device)

----

**getSensor(deviceMac: str) -> SensorProfile | None**

根据 MAC 地址获取已创建的 SensorProfile。

参数：
  - ``deviceMac`` — 设备 MAC 地址（格式：XX:XX:XX:XX:XX:XX）

返回：SensorProfile 实例，不存在返回 None

.. code-block:: python

    sensor = controller.getSensor("AA:BB:CC:DD:EE:FF")

----

**getConnectedSensors() -> list[SensorProfile]**

获取所有已连接的 SensorProfile 列表。

返回：list[SensorProfile] — 已连接的传感器列表

.. code-block:: python

    sensors = controller.getConnectedSensors()

----

**getConnectedDevices() -> list[BLEDevice]**

获取所有已连接的 BLEDevice 列表。

返回：list[BLEDevice] — 已连接的蓝牙设备列表

.. code-block:: python

    devices = controller.getConnectedDevices()

----

**terminate() -> None**

终止 SDK，释放所有资源。程序退出前必须调用。

前置条件：应在程序退出前调用，会自动断开所有已连接设备，释放后台线程

.. code-block:: python

    controller.terminate()

SensorProfile
~~~~~~~~~~~~~

SensorProfile 代表单个 OYWW1000 设备的连接和数据流管理。

属性
^^^^

.. list-table::
   :header-rows: 1
   :widths: 30 20 50

   * - 属性
     - 类型
     - 说明
   * - deviceState
     - DeviceStateEx
     - 只读，当前设备状态
   * - hasInited
     - bool
     - 只读，是否已完成 init() 初始化
   * - isDataTransfering
     - bool
     - 只读，是否正在传输数据
   * - BLEDevice
     - BLEDevice
     - 只读，关联的蓝牙设备对象

回调设置
^^^^^^^^

**onStateChanged**

设备状态变化回调。

参数：
  - ``sensor`` (SensorProfile) — 触发回调的传感器实例
  - ``state`` (DeviceStateEx) — 新的设备状态

.. code-block:: python

    def on_state_changed(sensor, state):
        print(f"设备状态: {state}")

    sensor.onStateChanged = on_state_changed

----

**onErrorCallback**

错误回调。

参数：
  - ``sensor`` (SensorProfile) — 触发回调的传感器实例
  - ``reason`` (str) — 错误原因

.. code-block:: python

    def on_error(sensor, reason):
        print(f"错误: {reason}")

    sensor.onErrorCallback = on_error

----

**onPowerChanged**

电量变化回调。

参数：
  - ``sensor`` (SensorProfile) — 触发回调的传感器实例
  - ``power`` (int) — 电量百分比 (0-100)，-1 表示无效

.. code-block:: python

    def on_power_changed(sensor, power):
        print(f"电量: {power}%")

    sensor.onPowerChanged = on_power_changed

----

**onDataCallback**

数据回调，接收传感器数据。

参数：
  - ``sensor`` (SensorProfile) — 触发回调的传感器实例
  - ``data`` (SensorData) — 包含数据类型、采样率、通道数据等

.. code-block:: python

    def on_data(sensor, data):
        if data.dataType == DataType.NTF_EMG:
            pass  # 处理 EMG 数据

    sensor.onDataCallback = on_data

连接管理
^^^^^^^^^^^^^^^^

**connect() -> bool**

连接设备，阻塞直到进入 Ready 状态或超时。

返回：True 表示连接成功并进入 Ready 状态

前置条件：必须在 Ready 状态后才能调用 init() 等命令

.. code-block:: python

    success = sensor.connect()

----

**disconnect() -> bool**

断开设备连接。

返回：True 表示断开成功

.. code-block:: python

    success = sensor.disconnect()

数据流初始化
^^^^^^^^^^^^^^^^^^^^^^

**init(packageSampleCount: int, powerRefreshInterval: int) -> bool**

初始化数据流，配置采样参数和电量回调周期。

参数：
  - ``packageSampleCount`` — 每次 onDataCallback 中每通道包含的采样数（IMU/四元数建议 1，EMG 建议 32）
  - ``powerRefreshInterval`` — onPowerChanged 回调间隔（毫秒）

返回：True 表示初始化成功

前置条件：deviceState == DeviceStateEx.Ready；必须在 startDataNotification() 之前调用

.. code-block:: python

    success = sensor.init(
        packageSampleCount=1,
        powerRefreshInterval=60000
    )

----

**getDeviceInfo() -> DeviceInfo | None**

获取设备信息。

返回：DeviceInfo 对象，失败返回 None

前置条件：deviceState == DeviceStateEx.Ready

.. code-block:: python

    info = sensor.getDeviceInfo()
    if info:
        print(f"设备: {info.DeviceName}")
        print(f"固件: {info.FirmwareVersion}")
        print(f"EMG: {info.EmgChannelCount} 通道 @ {info.EmgSampleRate} Hz")

数据流控制
^^^^^^^^^^^^^^^^^^^

**startDataNotification() -> bool**

启动数据流，开始接收传感器数据。

返回：True 表示启动成功

前置条件：hasInited == True；启动后 onDataCallback 开始触发

.. code-block:: python

    success = sensor.startDataNotification()

----

**stopDataNotification() -> bool**

停止数据流。

返回：True 表示停止成功

.. code-block:: python

    success = sensor.stopDataNotification()

参数配置
^^^^^^^^^^^^

**setParam(key: str, value: str) -> str**

设置设备参数，用于启用数据流和配置滤波器。

参数：
  - ``key`` — 参数名（见下表）
  - ``value`` — 参数值（通常为 ``"ON"`` 或 ``"OFF"``）

返回：``"OK"`` 表示设置成功，其他值表示失败

前置条件：``deviceState == DeviceStateEx.Ready``

.. code-block:: python

    result = sensor.setParam("NTF_EMG", "ON")
    if result == "OK":
        print("EMG 数据流已启用")

**支持的参数（OYWW1000 设备）**

.. list-table::
   :header-rows: 1
   :widths: 20 15 35 30

   * - 参数名
     - 值
     - 说明
     - 调用时机
   * - ``NTF_EMG``
     - ``ON`` / ``OFF``
     - 启用/禁用 EMG 数据流
     - ``init()`` 之前
   * - ``NTF_IMU``
     - ``ON`` / ``OFF``
     - 启用/禁用 IMU 数据流（ACC + GYRO，若设备支持则自动包含四元数）
     - ``init()`` 之前
   * - ``FILTER_HPF``
     - ``ON`` / ``OFF``
     - 10 Hz 高通滤波器
     - ``startDataNotification()`` 之后
   * - ``FILTER_LPF``
     - ``ON`` / ``OFF``
     - 200 Hz 低通滤波器
     - ``startDataNotification()`` 之后
   * - ``FILTER_50HZ``
     - ``ON`` / ``OFF``
     - 50 Hz 陷波滤波器（工频干扰）
     - ``startDataNotification()`` 之后
   * - ``FILTER_60HZ``
     - ``ON`` / ``OFF``
     - 60 Hz 陷波滤波器（工频干扰）
     - ``startDataNotification()`` 之后

电量查询
^^^^^^^^^^^^

**getBatteryLevel() -> int**

立即查询电量。

返回：电量百分比（0-100），失败返回 -1

前置条件：``deviceState == DeviceStateEx.Ready``

.. code-block:: python

    power = sensor.getBatteryLevel()
    print(f"当前电量: {power}%")

数据结构
------------

BLEDevice
~~~~~~~~~

BLEDevice 表示扫描发现的蓝牙设备。

.. list-table::
   :header-rows: 1
   :widths: 25 15 60

   * - 属性
     - 类型
     - 说明
   * - Name
     - str
     - 设备名称
   * - Address
     - str
     - MAC 地址（格式：XX:XX:XX:XX:XX:XX）
   * - RSSI
     - int
     - 信号强度（dBm）

DeviceInfo
~~~~~~~~~~

DeviceInfo 包含设备硬件和能力信息，通过 getDeviceInfo() 获取。

.. list-table::
   :header-rows: 1
   :widths: 30 15 55

   * - 属性
     - 类型
     - 说明
   * - DeviceName
     - str
     - 设备名称
   * - ModelName
     - str
     - 型号
   * - FirmwareVersion
     - str
     - 固件版本
   * - HardwareVersion
     - str
     - 硬件版本
   * - EmgChannelCount
     - int
     - EMG 通道数
   * - EmgSampleRate
     - int
     - EMG 采样率（Hz）
   * - AccChannelCount
     - int
     - 加速度通道数
   * - AccSampleRate
     - int
     - 加速度采样率（Hz）
   * - GyroChannelCount
     - int
     - 陀螺仪通道数
   * - GyroSampleRate
     - int
     - 陀螺仪采样率（Hz）
   * - QuatChannelCount
     - int
     - 四元数通道数（0 表示不支持）
   * - QuatSampleRate
     - int
     - 四元数采样率（Hz）
   * - MTUSize
     - int
     - BLE MTU 大小

SensorData
~~~~~~~~~~

SensorData 是 onDataCallback 中接收到的传感器数据包。

.. list-table::
   :header-rows: 1
   :widths: 30 25 45

   * - 属性
     - 类型
     - 说明
   * - deviceMac
     - str
     - 设备 MAC 地址
   * - dataType
     - DataType
     - 数据类型枚举
   * - sampleRate
     - int
     - 采样率（Hz）
   * - channelCount
     - int
     - 通道数
   * - packageSampleCount
     - int
     - 本包每通道采样数
   * - channelSamples
     - list[list[Sample]]
     - 二维数组：[通道索引][采样索引]

**channelSamples 访问模式**

通过 channelSamples[channel_index][sample_index] 访问 Sample 对象：

.. code-block:: python

    for ch_idx, ch_samples in enumerate(data.channelSamples):
        for sample in ch_samples:
            print(f"CH{ch_idx}: {sample.data}")

Sample
~~~~~~

Sample 表示单个采样点数据。

.. list-table::
   :header-rows: 1
   :widths: 25 15 60

   * - 属性
     - 类型
     - 说明
   * - data
     - float
     - 物理量值（单位见数据类型表）
   * - rawData
     - int
     - 原始 ADC 值
   * - impedance
     - int
     - 阻抗值（Ω）
   * - channelIndex
     - int
     - 通道索引
   * - sampleIndex
     - int
     - 采样序号（用于检测丢包）
   * - isLost
     - bool
     - True 表示该包丢失
   * - timeStampInMs
     - int
     - 时间戳（毫秒）

DataType
~~~~~~~~

DataType 枚举定义传感器数据类型。

.. list-table::
   :header-rows: 1
   :widths: 35 15 50

   * - 枚举值
     - 十六进制
     - 说明
   * - DataType.NTF_ACC
     - 0x01
     - 加速度
   * - DataType.NTF_GYRO
     - 0x02
     - 陀螺仪
   * - DataType.NTF_QUATERNION
     - 0x05
     - 四元数
   * - DataType.NTF_EMG
     - 0x08
     - 肌电
   * - DataType.NTF_IMPEDANCE
     - 0x12
     - 阻抗
   * - DataType.NTF_IMU
     - 0x13
     - IMU 复合包（内部拆分为 ACC + GYRO，若设备支持则同时包含四元数）

DeviceStateEx
~~~~~~~~~~~~~

DeviceStateEx 枚举表示设备连接状态。

.. list-table::
   :header-rows: 1
   :widths: 25 10 65

   * - 状态
     - 值
     - 说明
   * - Disconnected
     - 0
     - 未连接
   * - Connecting
     - 1
     - 连接中
   * - Connected
     - 2
     - 已连接（BLE 层）
   * - Ready
     - 3
     - 就绪（可发送命令）
   * - Disconnecting
     - 4
     - 断开中
   * - Invalid
     - 5
     - 无效状态

快速开始
------------

导入 SDK
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    from sensor import *
    import signal

初始化控制器并扫描设备
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    controller = SensorController()

    def on_device_found(device_list):
        controller.stopScan()
        for d in device_list:
            print(f"发现设备: {d.Name}  {d.Address}  RSSI={d.RSSI}")

    controller.onDeviceFoundCallback = on_device_found
    controller.startScan(5000)  # 每 5 秒回调一次

    signal.signal(signal.SIGINT, lambda s, f: (controller.terminate(), exit()))

创建 SensorProfile 并注册回调
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    sensor = controller.requireSensor(ble_device)

    sensor.onStateChanged  = lambda s, st: print(f"状态变化: {st}")
    sensor.onErrorCallback = lambda s, r:  print(f"错误: {r}")
    sensor.onPowerChanged  = lambda s, p:  print(f"电量: {p}%")
    sensor.onDataCallback  = on_data  # 见数据处理示例

连接设备
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    success = sensor.connect()
    if not success:
        print("连接失败")
        return
    # connect() 会阻塞直到设备进入 Ready 状态或超时

启用数据流并初始化
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

调用顺序：setParam（数据流启用）→ init() → startDataNotification() → setParam（滤波器）

.. code-block:: python

    # 步骤 1：启用数据流（必须在 init() 之前）
    sensor.setParam("NTF_EMG", "ON")
    sensor.setParam("NTF_IMU", "ON")  # 若设备支持四元数，init() 时会自动一并初始化

    # 步骤 2：初始化
    success = sensor.init(packageSampleCount=1, powerRefreshInterval=60_000)
    if not success:
        print("初始化失败")
        return

    # 步骤 3：查询设备信息
    info = sensor.getDeviceInfo()
    print(f"设备: {info.DeviceName}  固件: {info.FirmwareVersion}")

启动数据流
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    success = sensor.startDataNotification()
    if not success:
        print("启动数据流失败")
        return

    # 步骤 4：启用滤波器（数据流启动后）
    sensor.setParam("FILTER_HPF", "ON")
    sensor.setParam("FILTER_50HZ", "ON")

停止与断开
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    sensor.stopDataNotification()
    sensor.disconnect()
    controller.terminate()  # 程序退出前必须调用

数据处理示例
------------------

IMU（加速度 + 陀螺仪 + 四元数）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

IMU 数据通过 NTF_IMU 复合包下发，onDataCallback 会分别以 DataType.NTF_ACC、DataType.NTF_GYRO 触发。若设备硬件支持 QAT6，四元数（DataType.NTF_QUATERNION）也会随 IMU 一同初始化并下发，无需额外配置。通道顺序固定为 X / Y / Z（索引 0 / 1 / 2）。

启用方式（在 init() 之前调用）：

.. code-block:: python

    sensor.setParam("NTF_IMU", "ON")

.. note::

    四元数是否下发取决于设备硬件是否支持 QAT6。可通过 ``DeviceInfo.QuatChannelCount > 0`` 判断。

数据处理示例：

.. code-block:: python

    def on_data(sensor, data):
        if data.dataType == DataType.NTF_ACC:
            # 加速度数据，单位 g
            for sample in data.channelSamples[0]:   # X 轴
                print(f"ACC X: {sample.data:.4f} g")
            for sample in data.channelSamples[1]:   # Y 轴
                print(f"ACC Y: {sample.data:.4f} g")
            for sample in data.channelSamples[2]:   # Z 轴
                print(f"ACC Z: {sample.data:.4f} g")
        elif data.dataType == DataType.NTF_GYRO:
            # 陀螺仪数据，单位 °/s
            for sample in data.channelSamples[0]:
                print(f"GYRO X: {sample.data:.4f} °/s")
            for sample in data.channelSamples[1]:
                print(f"GYRO Y: {sample.data:.4f} °/s")
            for sample in data.channelSamples[2]:
                print(f"GYRO Z: {sample.data:.4f} °/s")
        elif data.dataType == DataType.NTF_QUATERNION:
            # 四元数数据（设备支持时随 IMU 自动下发），通道顺序 W/X/Y/Z
            w = data.channelSamples[0][0].data
            x = data.channelSamples[1][0].data
            y = data.channelSamples[2][0].data
            z = data.channelSamples[3][0].data
            print(f"Quat  W={w:+.4f}  X={x:+.4f}  Y={y:+.4f}  Z={z:+.4f}")

EMG（肌电）
~~~~~~~~~~~~~~~~~~

EMG 数据单位为 µV，通道数由设备决定。

启用方式（在 init() 之前调用）：

.. code-block:: python

    sensor.setParam("NTF_EMG", "ON")

数据处理示例（含丢包检测）：

.. code-block:: python

    def on_data(sensor, data):
        if data.dataType == DataType.NTF_EMG:
            for ch_idx, ch_samples in enumerate(data.channelSamples):
                for sample in ch_samples:
                    if sample.isLost:
                        print(f"CH{ch_idx} 丢包: sampleIndex={sample.sampleIndex}")
                    else:
                        print(f"CH{ch_idx} EMG: {sample.data:.2f} µV  阻抗: {sample.impedance} Ω")

阻抗（Impedance）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

阻抗数据有两种获取方式：

1. 从 EMG 数据包中读取（推荐）：每个 Sample 的 impedance 字段实时包含阻抗值
2. 独立阻抗数据包：DataType.NTF_IMPEDANCE

.. code-block:: python

    def on_data(sensor, data):
        if data.dataType == DataType.NTF_IMPEDANCE:
            for ch_idx, ch_samples in enumerate(data.channelSamples):
                for sample in ch_samples:
                    ohm = sample.impedance
                    kohm = ohm / 1000
                    print(f"CH{ch_idx} 阻抗: {kohm:.1f} KΩ")

.. note::

    阻抗数据通常在 startDataNotification() 后随 EMG 数据一同下发，无需单独订阅。

DSP 滤波器配置
~~~~~~~~~~~~~~~~~~~

在 startDataNotification() 之后可随时调用，对 EMG 信号生效：

.. code-block:: python

    # 10 Hz 高通滤波器
    sensor.setParam("FILTER_HPF", "ON")

    # 200 Hz 低通滤波器
    sensor.setParam("FILTER_LPF", "ON")

    # 50 Hz 陷波滤波器（工频干扰）
    sensor.setParam("FILTER_50HZ", "ON")

    # 60 Hz 陷波滤波器（工频干扰）
    sensor.setParam("FILTER_60HZ", "ON")

.. note::

    返回 "OK" 表示设置成功。

完整示例
------------

以下示例覆盖完整流程：扫描 → 连接 → 启用数据流 → 初始化 → 启动 → 接收所有数据类型 → 退出清理。

.. code-block:: python

    import signal
    import time
    from sensor import *

    SCAN_MS = 5000
    PACKAGE_COUNT = 1  # IMU/四元数每包 1 个样本；EMG 可设为 32

    controller = SensorController()
    sensor_ref = None


    def on_device_found(device_list):
        controller.stopScan()
        for d in device_list:
            if d.Name.startswith("OY"):  # 筛选 OYWW 设备
                setup_sensor(d)
                break


    def setup_sensor(device):
        global sensor_ref
        sensor = controller.requireSensor(device)
        if sensor is None:
            return

        # 注册回调
        sensor.onStateChanged  = lambda s, st: print(f"[状态] {st}")
        sensor.onErrorCallback = lambda s, r:  print(f"[错误] {r}")
        sensor.onPowerChanged  = lambda s, p:  print(f"[电量] {p}%")
        sensor.onDataCallback  = on_data

        # 连接设备
        if not sensor.connect():
            print("连接失败")
            return

        # 启用数据流（init 之前）
        sensor.setParam("NTF_IMU", "ON")  # 若设备支持四元数，init() 时会自动一并初始化
        sensor.setParam("NTF_EMG", "ON")

        # 初始化
        if not sensor.init(PACKAGE_COUNT, 60_000):
            print("初始化失败")
            return

        # 查询设备信息
        info = sensor.getDeviceInfo()
        print(f"设备: {info.DeviceName}  固件: {info.FirmwareVersion}")
        print(f"EMG  {info.EmgChannelCount}ch @ {info.EmgSampleRate}Hz")
        print(f"ACC  {info.AccChannelCount}ch @ {info.AccSampleRate}Hz")
        print(f"Quat {info.QuatChannelCount}ch @ {info.QuatSampleRate}Hz")

        # 启动数据流
        if not sensor.startDataNotification():
            print("启动数据流失败")
            return

        # 启用滤波器（数据流启动后）
        sensor.setParam("FILTER_HPF", "ON")
        sensor.setParam("FILTER_LPF", "ON")
        sensor.setParam("FILTER_50HZ", "ON")

        sensor_ref = sensor
        print("数据流已启动")


    def on_data(sensor, data):
        dt = data.dataType

        if dt == DataType.NTF_ACC:
            x = data.channelSamples[0][0].data
            y = data.channelSamples[1][0].data
            z = data.channelSamples[2][0].data
            print(f"ACC  X={x:+.3f}g  Y={y:+.3f}g  Z={z:+.3f}g")

        elif dt == DataType.NTF_GYRO:
            x = data.channelSamples[0][0].data
            y = data.channelSamples[1][0].data
            z = data.channelSamples[2][0].data
            print(f"GYRO X={x:+.2f}°/s  Y={y:+.2f}°/s  Z={z:+.2f}°/s")

        elif dt == DataType.NTF_QUATERNION:
            w = data.channelSamples[0][0].data
            x = data.channelSamples[1][0].data
            y = data.channelSamples[2][0].data
            z = data.channelSamples[3][0].data
            print(f"QUAT W={w:+.4f}  X={x:+.4f}  Y={y:+.4f}  Z={z:+.4f}")

        elif dt == DataType.NTF_EMG:
            for ch_idx, ch_samples in enumerate(data.channelSamples):
                for s in ch_samples:
                    if not s.isLost:
                        print(f"EMG CH{ch_idx}: {s.data:.1f}µV  阻抗={s.impedance}Ω")

        elif dt == DataType.NTF_IMPEDANCE:
            for ch_idx, ch_samples in enumerate(data.channelSamples):
                for s in ch_samples:
                    print(f"阻抗 CH{ch_idx}: {s.impedance / 1000:.1f}KΩ")


    def terminate():
        if sensor_ref:
            sensor_ref.stopDataNotification()
            sensor_ref.disconnect()
        controller.terminate()
        exit()


    signal.signal(signal.SIGINT, lambda s, f: terminate())

    controller.onDeviceFoundCallback = on_device_found
    controller.startScan(SCAN_MS)

    time.sleep(60)
    terminate()

注意事项
------------

1. **命令调用顺序** — 所有命令（init、setParam、startDataNotification）必须在 ``deviceState == DeviceStateEx.Ready`` 后调用；数据流启用（``setParam("NTF_IMU", "ON")`` 等）需在 ``sensor.init()`` 之前设置；滤波器配置需在 ``startDataNotification()`` 之后调用。

2. **资源释放** — 程序退出时必须调用 ``controller.terminate()``，否则 BLE 后台线程不会释放。

3. **线程安全** — ``onDataCallback`` 在内部线程触发，如需更新 UI 请通过信号/队列转发到主线程。

4. **丢包处理** — ``sample.isLost == True`` 时 ``sample.data`` 无效，应跳过该样本；可通过 ``sample.sampleIndex`` 检测连续性。

5. **滤波器参数（OYWW1000 设备）** — 高通滤波器：10 Hz；低通滤波器：200 Hz；陷波滤波器：50 Hz / 60 Hz（工频干扰）。

6. **异步方法** — SDK 提供异步版本：``asyncConnect()``、``asyncInit()``、``asyncStartDataNotification()`` 等；异步方法需在 async 函数中使用 ``await`` 调用。

常见问题
------------

**Q1: 如何判断设备是否支持四元数？**

四元数随 IMU 数据流自动下发，无需单独启用。通过 ``DeviceInfo.QuatChannelCount`` 判断设备是否支持：

.. code-block:: python

    info = sensor.getDeviceInfo()
    if info.QuatChannelCount > 0:
        print(f"支持四元数: {info.QuatChannelCount} 通道 @ {info.QuatSampleRate} Hz")
    else:
        print("设备不支持四元数")

----

**Q2: 滤波器设置不生效？**

确认调用时机：数据流启用（``NTF_EMG`` 等）需在 ``init()`` 之前；滤波器配置需在 ``startDataNotification()`` 之后。

----

**Q3: 如何检测丢包？**

检查 ``sample.isLost`` 或通过 ``sample.sampleIndex`` 连续性判断：

.. code-block:: python

    last_index = -1
    for sample in ch_samples:
        if sample.isLost or (last_index >= 0 and sample.sampleIndex != last_index + 1):
            print(f"丢包检测: {last_index} -> {sample.sampleIndex}")
        last_index = sample.sampleIndex

----

**Q5: 程序退出时报错？**

确保调用了 ``controller.terminate()``：

.. code-block:: python

    import signal

    def cleanup():
        controller.terminate()
        exit()

    signal.signal(signal.SIGINT, lambda s, f: cleanup())
