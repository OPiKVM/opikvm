# 修改 HID 设备的 PID/VID

* OPiKVM 支持修改 USB 键盘鼠标的 PID 与 VID 信息修改。

    默认情况下，设置如下：

    ```yaml
    otg:
        manufacturer: PiKVM
        product: Composite KVM Device
        vendor_id: 0x1D6B
        product_id: 0x0104
        serial: CAFEBABE
    ```

* 你可以通过以下示例更改显示内容，编辑 `/etc/kvmd/override.yaml` 文件：

    ```yaml
    otg:
        manufacturer: Corsair
        product: Corsair Gaming RGB
        vendor_id: 0x6940
        product_id: 0x6973
        serial: XXXXXXXX
    ```

    使用以下 USB 数据库获取所需的设备信息：[https://the-sz.com/products/usbid](https://the-sz.com/products/usbid) 或 [https://devicehunt.com](https://devicehunt.com)。
