---
search:
    exclude: true
---

OPiKVM 终端执行 `kvmd-edidconf` 查看当前 EDID 信息：

```console
[root@opi-kvm ~]# kvmd-edidconf
Manufacturer ID: LNX
Product ID:      0x7773 (30579)
Serial number:   0x01010101 (16843009)
Monitor name:    PiKVM V4 Plus
Monitor serial:  CAFEBABE
Audio:           yes
```

这些字段有明显的名称和用途。注意 `Serial number` 和 `Monitor serial` 两个相似的字段。
第一个是数字值，第二个是 ASCII 字符串。如果您使用来自真实显示器的自定义 EDID，某些字段可能会缺失。

使用 `kvmd-edidconf` 配命令配合参数修改（完整参数列表请执行 `kvmd-edidconf --help` ）：

```console
[root@opi-kvm ~]#
[root@opi-kvm ~]# kvmd-edidconf --set-mfc-id=TTP --set-product-id=0x5B81 --set-serial=0x8DE11B79 --set-monitor-name=TOSHIBA --set-monitor-serial=ABCD1234 --apply
Manufacturer ID: TTP
Product ID:      0x5B81 (23425)
Serial number:   0x8DE11B79 (2380340089)
Monitor name:    TOSHIBA
Monitor serial:  ABCD1234
Audio:           yes
...
[root@pi-kvm ~]# echo 1 > /sys/bus/i2c/devices/1-002b/reset
```

完整制造商ID列表请访问：[这里](https://uefi.org/pnp_id_list).