# USB 设备配置

OPiKVM 默认模拟一组核心 USB 设备以确保基本功能：一个键盘、鼠标 和 [大容量存储驱动器](msd.md)。此外，您还可以扩展更多设备： [USB 以太网](usb_ethernet.md) 或 [支持双向音频的麦克风](audio.md)。

极少数情况下，目标主机的 BIOS/UEFI 可能无法正确处理单 USB 端口模拟的多个设备，可能需要被禁用某些设备。

完整的 USB 配置更改（添加或移除设备）需要重新启动，但也可以暂时禁用现有的模拟设备，然后在预设中重新启用它们。

-----

## 基础概念：端点资源

每个模拟 USB 设备均消耗有限的硬件资源——**端点（Endpoints）**。

根据设备的不同，所需的端点数有所不同：

| 设备 | 端点数 |
|------|--------|
| 键盘、鼠标 | 每个 1 个 |
| 大容量存储驱动器 | 每个 2 个 |
| USB 麦克风 | 2 个 |
| USB 以太网、USB 串口 | 每个 3 个 |

**总的来说，OPiKVM 提供 9 个端点用于 USB 仿真**，其中一些端点默认已经使用：

* OPiKVM 模拟一个绝对鼠标和一个大容量存储驱动器，使用了 4 个端点中的 9 个。

* 您可以添加其他设备，使用剩余的端点，或者禁用现有的设备来释放一些端点，或者仅临时执行此操作。

* 您还可以在预设中配置大量设备（超过 OPiKVM 允许的端点数），然后动态启用仅需要的设备。

可添加以下设备（需确保总端点 ≤ 9）：

如果您配置的设备消耗的端点总数超过了 9 个，那么其中最不重要的设备将会被禁用。您可以使用动态配置来启用它们。

要配置附加设备，请参阅以下相应页面：

* [USB 麦克风](audio.md) - 用于目标主机的语音应用的双向音频通信。

* [USB 以太网](usb_ethernet.md) - 可以在 OPiKVM 上配置 FTP 或 Samba 服务器，目标主机将通过网络看到它。OPiKVM 还可以作为路由器，将主机连接到大型网络。

设备标识符自定义方法参见[这里](id.md)。

-----

## 预设配置管理

设备配置分两步：添加设备 → 设置启动状态

当您按上述页面所述添加设备时，它会在 OPiKVM 重启后自动启动，并在目标设备上可用。此行为可以更改：设备将被创建，但不会立即启动，直到您动态开启它。

`/etc/kvmd/override.yaml` 文件用于进行此类更改。在以下示例中，启用了 USB 串口和 [麦克风](audio.md)，但串口默认不启动：

```yaml
otg:
    devices:
        serial:             # USB 串口
            enabled: true   # 创建设备
            start: false    # 默认不启动（节省端点）
        audio:              # USB 麦克风
            enabled: true   # 创建并默认启动
```

* `enabled: true` 表示创建设备（需重启生效）
* `start: true/false` 控制设备默认启动状态
* 执行 `kvmd -m` 查看完整配置树

## 动态配置

### 命令行工具

`kvmd-otgconf` 工具允许您实时查看和修改 USB 配置。

更改配置需要 root 权限。

查看配置。每一行代表一个模拟的设备。

`+` 表示启用状态，`-` 表示禁用状态， `# [数字]` 显示设备消耗端点数量

```console
[root@opi-kvm ~]# kvmd-otgconf
# Endpoints used: 7 of 9
# Endpoints free: 2
- acm.usb0  # [3] 串口
+ hid.usb0  # [1] 键盘
+ hid.usb1  # [1] 绝对鼠标
+ hid.usb2  # [1] 相对鼠标
+ mass_storage.usb0  # [2] 大容量存储驱动器
+ uac2.usb0  # [2] 麦克风
```

当 BIOS 因 USB 兼容性问题无法识别存储设备时：

在这种情况下，您可以禁用除键盘和相对鼠标之外的所有设备，然后进入 BIOS：

```console
[root@opi-kvm ~]# kvmd-otgconf -d mass_storage.usb0 uac2.usb0 hid.usb1
# Endpoints used: 2 of 9
# Endpoints free: 7
- acm.usb0  # [3] 串口
+ hid.usb0  # [1] 键盘
- hid.usb1  # [1] 绝对鼠标
+ hid.usb2  # [1] 相对鼠标
- mass_storage.usb0  # [2] 大容量存储驱动器
- uac2.usb0  # [2] 麦克风
```

然后在 BIOS 中更改启动顺序，将 USB 启动盘设置为优先启动。

退出 BIOS 后，再次开启大容量存储驱动器。使用它从 OPiKVM 大容量存储启动镜像：

```console
[root@opi-kvm ~]# kvmd-otgconf -e mass_storage.usb0
# Endpoints used: 4 of 9
# Endpoints free: 5
- acm.usb0  # [3] 串口
+ hid.usb0  # [1] 键盘
- hid.usb1  # [1] 绝对鼠标
+ hid.usb2  # [1] 相对鼠标
+ mass_storage.usb0  # [2] 大容量存储驱动器
- uac2.usb0  # [2] 麦克风
```

您还可以再次启用 `uac2.usb0` 和 `hid.usb1`。

### Web UI 菜单

通过 GPIO 扩展实现 Web 界面动态控制。

要设置菜单，使用 `kvmd-otgconf --make-gpio-config` 生成配置，并将其与现有配置合并到 `/etc/kvmd/override.yaml` 文件中。

??? example "示例：`kvmd-otgconf --make-gpio-config` 输出"
    ```yaml
    # kvmd-otgconf --make-gpio-config
    kvmd:
        gpio:
            drivers:
                otgconf:
                    type: otgconf
            scheme:
                hid.usb0:
                    driver: otgconf
                    mode: output
                    pin: hid.usb0
                    pulse: false
                hid.usb1:
                    driver: otgconf
                    mode: output
                    pin: hid.usb1
                    pulse: false
                hid.usb2:
                    driver: otgconf
                    mode: output
                    pin: hid.usb2
                    pulse: false
                mass_storage.usb0:
                    driver: otgconf
                    mode: output
                    pin: mass_storage.usb0
                    pulse: false
            view:
                table:
                    - ["#键盘", "#hid.usb0", hid.usb0]
                    - ["#绝对鼠标", "#hid.usb1", hid.usb1]
                    - ["#相对鼠标", "#hid.usb2", hid.usb2]
                    - ["#大容量存储驱动器", "#mass_storage.usb0", mass_storage.usb0]
    ```

请注意，该菜单不是动态生成的，如果您添加或删除设备，则需要更新配置。
