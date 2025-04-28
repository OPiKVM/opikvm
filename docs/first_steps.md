# 首次启动

!!! note "准备工作"

    * TF卡一张，建议容量16G以上Class10的内存卡

!!! warning "注意! OPiKVM CM4发货时已经预先刷好镜像,您不需要再烧录,如果需要重新刷镜像按照下列方法烧录镜像"

---

## 接通电源

!!! warning "切记 **不要!** 插入电压输出大于5V的电源适配器，会烧坏开发板。"

    * 连接一个 5V/3A 的 USB Type-C 接口的高品质的电源适配器
    * Type-C电源接口不支持PD协商

---

## 等待系统初始化完成

!!! 提示

    * 首次上电时系统会进行初始化，此时oled屏幕上会提示"系统正在初始化中，请勿中断"，请稍微等待至屏幕出现设备ip地址后，系统加载完成。

    * ![oled](img/login/oled.jpg){:width="150px" .off-glb}

---

## 连接网络

有以下三种方法可以将OPiKVM接入到网络中,你可以根据你的实际情况选择不同的方法,具体操作步骤请参考[如何接入网络](network.md)

* 有线网络连接
* Wi-Fi连接
* USB共享网络

## 登录方式

!!! 提示

    * Web UI登录用于访问远程主机
    * SSH用于登录PiKVM本机系统

{!_passwd.md!}

---

### Web UI登录

- 打开浏览器输入PiKVM的ip地址，ip地址会在oled屏幕上显示,输入[`PiKVM Web`](auth.md)用户名密码，如果开启了[2FA](auth.md#2fa-two-factor-authentication)功能，也请输入2FA code

    ![login](img/login/login.jpg){.off-glb}

- 登录成功后会跳转至网页控制台，分别为`KVM`、`Terminal终端`、`登出`按钮

    ![session](img/login/session.jpg){.off-glb}

---

### KVM

- 点击`KVM`按钮进入KVM功能界面，可以看到采集到远程主机的HDMI输出画面

    ![kvm_info](img/kvm_info.jpg){.off-glb}

- 点击左上角OPiKVM图标，提示离开此网站，点击离开，返回主控制台

    ![return](img/login/return.jpg){.off-glb}

---

### Web Terminal终端

有以下两种方式打开Web终端

1. 控制台点击`Terminal`按钮，进入Web Terminal终端

    ![web_terminal](img/login/web_terminal.jpg){.off-glb}

2. `KVM`界面下点击`系统` > `终端`

    ![kvm_terminal](img/login/kvm_terminal.jpg){.off-glb}

    Web Terminnal会以[`kvmd-webterm`](auth.md)账户默认登录，使用浏览器后退按钮返回控制台

    !!! example "也可以使用SSH终端软件通过root账号进行登录"
        ```console
        ssh root@PiKVM地址
        ```
