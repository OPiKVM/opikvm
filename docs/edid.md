---
title: EDID
description: 如何在您的 PiKVM 上操作 EDID 信息
---

!!! info

    这适用于及基于 CSI 桥接的版本。
    无法更改 HDMI-USB 采集卡的 EDID。

EDID 提供了有关视频采集设备所支持的视频模式的信息。
在 PiKVM 的情况下，这是一个 HDMI CSI 桥接芯片。
通常情况下，您不需要更改它，因为默认配置已经非常灵活。
但在某些情况下，例如针对某些特殊的 UEFI/BIOS，可能需要进行更改（[参考案例](https://github.com/pikvm/pikvm/issues/78)）。

-----

## 基础知识

EDID 以 HEX（十六进制）格式存储在 PiKVM 的 `/etc/kvmd/tc358743-edid.hex` 文件中。
在启动 PiKVM OS 时，该文件会被 `kvmd-tc358743.service` 调用并加载到视频采集芯片中。
如果您替换了该文件中的 EDID，可以使用命令 `kvmd-edidconf --device=/dev/kvmd-subdev --apply` 手动应用，无需重启系统。

如果您只是想更改显示器的标识信息（如名称、序列号等），我们不建议您更改整个 EDID。
只需使用 `kvmd-edidconf` 及其内置的 EDID 更改选项即可。

!!! note

    Windows 会缓存驱动程序和注册表设置，因此仅更改显示器名称是不够的。
    您还需要同时更改产品 ID 和/或序列号：
    ```console
    [root@pikvm ~]# kvmd-edidconf --set-monitor-name=TOSHIBA --set-mfc-id=TTP --set-product-id=34953 --set-serial=2290649089 --device=/dev/kvmd-subdev --apply
    ```

{!_edidconf_options.md!}

下面将介绍处理 EDID 的典型示例以及使用自定义 EDID 的完整生命周期。

-----

<!-- ## 在 V4 Plus 上采用真实显示器标识

PiKVM V4 Plus 提供了一种简单的方法，可以从连接到 `OUT2` 端口（该端口也用于 [HDMI 环出](pass.md)）的物理显示器中读取并采用型号和序列号等显示器标识。
这样，目标主机就会将 PiKVM 识别为您的物理显示器。

要采用显示器标识，请将显示器连接 to `OUT2` 端口并运行以下命令：
```console
[root@pikvm ~]# rw
[root@pikvm ~]# kvmd-edidconf --import-display-ids --apply
[root@pikvm ~]# ro
```

现在可以拔掉显示器了，PiKVM 会记住这些新设置。

----- -->

## 恢复默认 EDID

如果您需要恢复默认的 EDID，可以使用 `kvmd-edidconf` 轻松实现，例如：
```console
[root@pikvm ~]# kvmd-edidconf --import-preset=v4plus --device=/dev/kvmd-subdev --apply
```

可用预设选项：`v0`、`v1`、`v2`、`v3`、`v4mini` 和 `v4plus`。
此外，默认的 EDID 也可以在您的 PiKVM 本地找到：`/usr/share/kvmd/configs.default/kvmd/edid`，或者在 [kvmd 仓库](https://github.com/pikvm/kvmd/blob/master/configs/kvmd/edid)中找到。

-----

## 在 PiKVM V0-V3 上默认强制 1080p

PiKVM V3（或 DIY V0-V2）在 1080p 模式下有 50Hz 的硬件限制，这比 60Hz 的常见频率要低。
因此，在 V3 上，默认模式为 720p。某些操作系统（如 Proxmox）在 720p 下可能工作不佳，因此您可以默认强制使用 1080p 分辨率：
```console
[root@pikvm ~]# rw
[root@pikvm ~]# kvmd-edidconf --import-preset=v3.1080p-by-default --device=/dev/kvmd-subdev --apply  # 或者使用 v1.1080p-by-default
[root@pikvm ~]# ro
```

-----

## 在 PiKVM V4 上禁用 1920x1200

PiKVM V4 支持 1920x1200 的高级采集模式。如果这给您带来困扰，您可以轻松禁用它并仅使用 1920x1080：
```console
[root@pikvm ~]# rw
[root@pikvm ~]# kvmd-edidconf --import-preset=v4plus.no-1920x1200 --device=/dev/kvmd-subdev --apply  # 或者 v4mini.no-1920x1200
[root@pikvm ~]# ro
```

-----

## 应用自定义 EDID

PiKVM 能够模拟具有特定 EDID 的物理显示器。
您可以在 [社区数据库](https://github.com/linuxhw/EDID) 中找到 EDID 示例，并在 PiKVM 上使用它。

同时，您应该注意 PiKVM 的硬件性能以及您所使用的 EDID 的兼容范围。例如，如果 EDID 声明支持 8K 分辨率，这显然是行不通的：您的主机会尝试发送 8K 信号，而 PiKVM 最大只能处理 1080p。

* OPiKVM CM4：最大分辨率为 1920x1200 @ 60Hz。

#### EDID 示例

??? example "Acer B246WL，1920x1200，支持音频"

    获取自 [此处](https://github.com/linuxhw/EDID/blob/master/Digital/Acer/ACR0565/CCF78B30FE61)，如上所述。
    ```
    00FFFFFFFFFFFF00047265058A3F6101
    101E0104A53420783FC125A8554EA026
    0D5054BFEF80714F8140818081C08100
    8B009500B300283C80A070B023403020
    360006442100001A000000FD00304C57
    5716010A202020202020000000FC0042
    323436574C0A202020202020000000FF
    0054384E4545303033383532320A01F8
    02031CF14F9002030405060701111213
    1415161F2309070783010000011D8018
    711C1620582C250006442100009E011D
    007251D01E206E28550006442100001E
    8C0AD08A20E02D10103E960006442100
    0018C344806E70B028401720A8040644
    2100001E000000000000000000000000
    00000000000000000000000000000096
    ```

??? example "ASUS PA248QV，1920x1200，支持音频"

    获取自 [此处](https://github.com/linuxhw/EDID/blob/master/Digital/ASUS/AUS2487/2B473481CAE6)，如上所述。
    ```
    00FFFFFFFFFFFF0006B3872401010101
    021F010380342078EA6DB5A7564EA025
    0D5054BF6F00714F8180814081C0A940
    9500B300D1C0283C80A070B023403020
    360006442100001A000000FD00314B1E
    5F19000A202020202020000000FC0050
    4132343851560A2020202020000000FF
    004D314C4D51533035323135370A014D
    02032AF14B900504030201111213141F
    230907078301000065030C001000681A
    00000101314BE6E2006A023A80187138
    2D40582C450006442100001ECD5F80B0
    72B0374088D0360006442100001C011D
    007251D01E206E28550006442100001E
    8C0AD08A20E02D10103E960006442100
    001800000000000000000000000000DC
    ```

??? example "DELL D2721H（用于避免某些 HDMI 分配器黑屏），1920x1080，无音频"

    获取自 [此处](https://github.com/linuxhw/EDID/blob/master/Digital/Dell/DEL2013/EEE824E681BF)，如上所述。
    ```
    00FFFFFFFFFFFF0010AC132045393639
    201E0103803C22782ACD25A3574B9F27
    0D5054A54B00714F8180A9C0D1C00101
    010101010101023A801871382D40582C
    450056502100001E000000FF00335335
    475132330A2020202020000000FC0044
    454C4C204432373231480A20000000FD
    00384C1E5311000A2020202020200181
    02031AB14F9005040302071601061112
    1513141F65030C001000023A80187138
    2D40582C450056502100001E011D8018
    711C1620582C250056502100009E011D
    007251D01E206E28550056502100001E
    8C0AD08A20E02D10103E960056502100
    00180000000000000000000000000000
    0000000000000000000000000000004F
    ```

#### 应用所选的自定义 EDID

要应用所选的 EDID，请执行以下步骤：

1. 使用任意文本编辑器打开 `/etc/kvmd/tc358743-edid.hex` 文件，例如使用 Nano：
    ```console
    [root@pikvm ~]# nano /etc/kvmd/tc358743-edid.hex
    ```

2. 将旧的 HEX 数据替换为新的内容，保存并关闭编辑器。

3. 应用 EDID 配置：
    ```console
    [root@pikvm ~]# kvmd-edidconf --device=/dev/kvmd-subdev --apply
    ```

4. 有时可能需要重启目标主机。请检查主机上的操作系统或 UEFI/BIOS 状态。
    如果一切正常，说明您的目标已达成，请继续最后一步。
    如果出现问题，您可以随时撤销这些更改并 [恢复默认 EDID](#edid)。

-----

## 编辑 EDID

要编辑 EDID，最好使用第三方工具，例如推荐的适用于 Windows 的高级工具 [AW EDID Editor](https://www.analogway.com/emea/products/software-tools/aw-edid-editor)（在 Wine 下运行非常完美）或 [wxEDID](https://sourceforge.net/projects/wxedid)。这两款编辑器均处理二进制 EDID 格式，但您可以使用 `kvmd-edidconf` 工具轻松地将其导入/导出到 PiKVM。

因此，要在 PiKVM 上调整 EDID，请遵循以下步骤：

1. 将系统当前的 EDID 导出为二进制文件 `myedid.bin`：
    ```console
    # kvmd-edidconf --export-bin=/root/myedid.bin
    ```

2. 使用 SCP、PuTTY 或类似工具将该文件复制到您的个人电脑（PC）上。
    在 EDID 编辑器中打开该二进制文件并修改所需参数。
    保存您的修改并把该二进制文件复制回 PiKVM。

3. 将二进制文件转换为 HEX 格式并进行测试：
    ```console
    [root@pikvm ~]# kvmd-edidconf --import=/root/myedid.bin --device=/dev/kvmd-subdev --apply
    ```

4. 有时可能需要重启目标主机。请检查主机上的操作系统或 UEFI/BIOS 状态。
    如果一切正常，说明您的目标已达成，请继续最后一步。
    如果出现问题，您可以随时撤销这些更改并 [恢复默认 EDID](#edid)。
