---
search:
    exclude: true
---


!!! danger "✮ ✮ ✮ 重要!修改密码! ✮ ✮ ✮"
    PiKVM 默认密码:

    * **Linux 操作系统级别的管理员**（SSH、控制台等）:
        * 用户名 `root` 和 `orangepi`
        * 密码 `orangepi`

    * **KVM 用户**（Web 界面，[API](https://docs.pikvm.org/api/)，[VNC](https://docs.pikvm.org/vnc/)等）:
        * 用户名 `admin`
        * 密码 `admin`
        * 默认未启用2FA

    **它们是两个独立的账户，具有独立的密码。**

    您需要通过 SSH 或 Web 终端更改密码。
    如果您使用的是 Web 终端，请输入 `su -` 命令以获取 `root` 访问权限（输入 `root` 用户密码）。

    ```console
    [kvmd-webterm@opi-kvm:~$] su -
    Password:
    [root@opi-kvm:~#] passwd                   #修改root密码
    New password:
    Retype new password:
    passwd: password updated successfully
    [root@opi-kvm:~#]
    [root@opi-kvm:~#] passwd orangepi          #修改orangepi密码
    New password:
    Retype new password:
    passwd: password updated successfully
    [root@opi-kvm:~#]
    [root@opi-kvm:~#]
    [root@opi-kvm:~#]kvmd-htpasswd set admin   #修改web密码
    Password:
    Repeat:
    [root@opi-kvm:~#]
    ```

    如果需要其他用户访问 Web UI，请使用以下命令:

    ```console
    [root@opi-kvm:~#] kvmd-htpasswd set <user> # 设置新用户或更改现有用户密码
    Password:
    Repeat:
    [root@opi-kvm:~#]
    [root@opi-kvm:~#] kvmd-htpasswd del <user> # 删除用户
    ```
    **或者，您也可以启用[2FA双因素身份验证](auth.md#2fa-two-factor-authentication)以提高安全性。**

    *在 PiKVM 首次启动时不需要更改 [VNCAuth密钥](https://github.com/pikvm/pikvm/blob/master/docs/vnc.md) 和 [IPMI密码](https://github.com/pikvm/pikvm/blob/master/docs/ipmi.md)，因为这些服务在默认情况下是禁用的。*
