# OPiKVM 设备标识

本页解释了 OPiKVM 在目标主机操作系统中的呈现方式及其修改方法。
适用于需要 OPiKVM 模拟特定USB设备或显示器的开发者、测试人员和系统管理员。

!!! info

    在阅读本页之前，建议先阅读 [OPiKVM 配置指南](config.md)，
    以便您了解术语和以下参数的具体更改方式。

-----

## 基础知识

OPiKVM 是一个复合式人机交互设备(Composite Device)模拟器。
简而言之，目标主机将连接的 OPiKVM 识别为一组设备而非单一设备。

在开箱即用的最默认情况下，包括以下设备：

* HDMI 视频显示器；
* USB 键盘；
* USB 鼠标；
* USB 大容量存储驱动器（可弹出）；

因此，OPiKVM 模拟了两类设备：HDMI 和 USB。每类设备均有特定标识符。

例如在主机显示器设置中，您会看到类似 `OPiKVM` 的标识。USB设备同理。

-----

## HDMI 标识符

EDID（Extended Display Identification Data）负责设置显示器信息，同时向主机提供 OPiKVM 支持的分辨率信息。

{!_edidconf_options.md!}

-----

## USB 标识符

USB 是更复杂的子系统，其标识配置需谨慎操作，错误设置可能导致 USB 功能失效。

有关如何控制模拟设备的信息，请参阅 [这里](usb.md)。
标识描述如下。

根据 [OPiKVM 配置指南](config.md)（若未阅读，请立即查阅），您可以使用 `kvmd -m` 命令获取所有配置参数。

以下仅列出与USB标识符相关的关键参数（无关参数已过滤）：

```yaml
[root@opi-kvm ~]# kvmd -m
otg:
    vendor_id: 7531
    product_id: 260
    manufacturer: PiKVM
    product: PiKVM Composite Device
    serial: CAFEBABE
    device_version: -1
    max_power: 250

    devices:
        drives:
            default:
                inquiry_string:
                    cdrom:
                        vendor: PiKVM
                        product: Optical Drive
                        revision: '1.00'

                    flash:
                        vendor: PiKVM
                        product: Flash Drive
                        revision: '1.00'

        msd:
            default:
                inquiry_string:
                    cdrom:
                        vendor: PiKVM
                        product: Optical Drive
                        revision: '1.00'

                    flash:
                        vendor: PiKVM
                        product: Flash Drive
                        revision: '1.00'
```

注意参数层级结构。所有数值以十进制显示，但配置文件中可使用十六进制。
USB规范通用名称对照如下：

| 参数             | USB 规范          | 描述                                         |
|------------------|-------------------|----------------------------------------------|
| `vendor_id`      | `idVendor`        | USB.org 分配的唯一 [厂商 ID](https://usb.org/sites/default/files/vendor_ids051920_0.pdf)。   |
| `product_id`     | `idProduct`       | 该厂商为产品分配的产品 ID。                 |
| `manufacturer`   | `iManufacturer`   | 厂商的 ASCII 名称。                          |
| `product`        | `iProduct`        | 产品的 ASCII 名称。                          |
| `serial`         | `iSerialNumber`   | 产品的 ASCII 序列号。                       |
| `device_version` | `bcdDevice`       | 设备版本。自动分配，可更改为 256、257、258 等。 |

OPiKVM 上的 [麦克风](audio.md#microphone-outgoing-audio) 功能也使用这些ID。

`otg/devices/drives` 和 `otg/devices/msd` 下的参数对应虚拟存储设备的SCSI查询字符串：

* `vendor`、`product` 和 `revision` 分别定义CD/DVD或Flash设备的标识信息。

* `msd` 指 Web UI 可访问的虚拟驱动，而 `drives` 描述了所有附加驱动（默认禁用）。

* 请注意，大容量存储驱动器可通过 [禁用大容量存储](msd.md#disabling-mass-storage) 完全禁用。

### 配置修改示例

在 /etc/kvmd/override.yaml 中添加以下内容：

```yaml
otg:
    vendor_id: 0x6940     # 十六进制形式（0x前缀）
    product_id: 0x6973
    manufacturer: Corsair
    product: Gaming RGB    # 模拟为"Corsair Gaming RGB"设备
    serial: 1000           # 自定义序列号

    devices:
        msd:
            default:
                inquiry_string:
                    cdrom:
                        vendor: Corsair
                        product: DVD        # 光盘显示为"Corsair DVD"
                        revision: '1.00'
                    flash:
                        vendor: Corsair
                        product: STICK      # U盘显示为"Corsair STICK"
                        revision: '1.00'
```

### 生效步骤

1. 执行 kvmd -m 验证配置：

    * 显示完整配置（含修改值）表示成功
    * 报错则需检查YAML语法

2. 如配置无误后执行，执行软重启 reboot。
