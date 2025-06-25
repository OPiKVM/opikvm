# 身份验证以及 2FA 验证

!!! warning "PiKVM 默认密码:"

    * **Linux 操作系统级别的管理员**（SSH、控制台等）:
        * 用户名 `root` 和 `orangepi`
        * 密码 `orangepi`

    * **KVM 用户**（Web 界面，[API](https://docs.pikvm.org/api/)，[VNC](https://docs.pikvm.org/vnc/)等）:
        * 用户名 `admin`
        * 密码 `admin`
        * 默认未启用2FA

    **它们是两个独立的账户，具有独立的密码。**

!!! danger "不要忘记在新设备上更改这两个密码"

    本页将介绍如何更改这些密码并启用双因素身份验证（2FA）。

    如果计划将 PiKVM 暴露到互联网或在不信任的网络中使用，强烈建议启用 2FA。

除了 KVM 用户和 Linux root 之外，还有一些其他的身份验证实体：

* **操作系统用户 `kvmd-webterm`**

    这是一个在 PiKVM 操作系统中具有非特权权限的特殊用户。
    它不能用于登录或通过 SSH 进行远程访问。密码访问和 `sudo` 也被禁用。
    它仅用于 Web 终端。出于安全原因，设置了这些限制。

* [**VNCAuth 密钥**](https://docs.pikvm.org/vnc/) - 默认禁用。

* [**IPMI 密码**](https://docs.pikvm.org/ipmi/) - 默认禁用。

-----

## Web 终端中的 Root 访问

如上所述，Web 终端以 `kvmd-webterm` 用户运行，并禁用了 `sudo` 和密码访问。
然而，大多数 PiKVM 管理命令需要 `root` 权限。
要在 Web 终端中获得它，输入 `su -`，然后输入 `root` 用户密码：

```console
[kvmd-webterm@opi-kvm:~$] su -
...
[root@opi-kvm:~#]
```

??? example "如何禁用 Web 终端"

    有时 PiKVM 设备的实际所有者和允许使用它的用户是不同的人。
    因此，你可能想禁用 Web UI 中的控制台访问。为此，请执行以下操作：

    ```console
    [kvmd-webterm@opi-kvm:~$] su -
    [root@opi-kvm:~#] systemctl disable --now kvmd-webterm
    [root@opi-kvm:~#]
    ```

    对于你自己对 PiKVM 操作系统的访问，仍然可以使用 SSH。

-----

## 更改 Linux 密码

```console
[root@opi-kvm:~#] passwd
New password:
Retype new password:
passwd: password updated successfully
[root@opi-kvm:~#]
```

-----

## 更改 KVM 密码

此密码用于 Web UI 登录、访问 [API](https://github.com/pikvm/pikvm/blob/master/docs/api.md)、[VNC](https://github.com/pikvm/pikvm/blob/master/docs/vnc.md)(如果启用)
以及其他与操作系统 shell 无关的功能。

默认情况下，配置了类似于 Apache Server 的身份验证方法：用户名和密码
以加密方式存储在 `/etc/kvmd/htpasswd` 文件中。要管理它们，可以使用 `kvmd-htpasswd` 工具。

```console
[root@opi-kvm:~#]kvmd-htpasswd set admin   #修改web密码
Password:
Repeat:
[root@opi-kvm:~#]
```

`admin` 是默认用户的名称。

??? example "如何添加 KVM 用户"
    你可以创建多个不同的用户，使用不同的密码访问 Web UI，但请记住，它们都具有相同的权限：

    ```console
    [root@opi-kvm:~#]kvmd-htpasswd set <user> # 添加新用户
    [root@opi-kvm:~#]kvmd-htpasswd list # 显示所有web ui账户
    [root@opi-kvm:~#]kvmd-htpasswd del <user> # 移除一个账户
    ```

    目前无法为不同的 KVM 用户创建任何ACL。

-----

## 2FA 双因素验证(Two-factor authentication) { #2fa-two-factor-authentication }

这是一种强力保护 PiKVM 的新方法，从 `KVM >= 3.196` 开始可用。
如果您需要将 PiKVM 暴露在互联网中，强烈建议启用它。

!!! warning
    使用 2FA 排除了使用 [IPMI](https://github.com/pikvm/pikvm/blob/master/docs/ipmi.md) 和 [VNC with vncauth](https://github.com/pikvm/pikvm/blob/master/docs/vnc.md) (默认情况下均禁用)的可能性。
    它还略微影响了 [API](https://github.com/pikvm/pikvm/blob/master/docs/api.md) 和带有用户/密码的常规 VNC 的使用，下文会介绍。

    请注意，2FA 不影响 Linux 操作系统对 `root` 用户的访问，因此请确保为 SSH 访问设置强密码(或设置[密钥访问](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server))。

??? example "在 PiKVM 上启用 2FA 验证"

    1. **确保 NTP 服务正在运行( `timedatectl` 命令)，否则您将无法访问**。时区无关紧要。

    2. 将 **Google Authenticator** 应用安装到您的移动设备
        ([iOS](https://apps.apple.com/us/app/google-authenticator/id388497605),
        [Android](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2))。 用来生成一次性访问代码。

    3. 在 PiKVM 上为一次性密码创建密钥，此命令会生成一个二维码：
       ```console
       [root@opi-kvm:~#]
       [root@opi-kvm:~#] kvmd-totp init
       [root@opi-kvm:~#]
       ```

    4. 运行 Google Authenticator 并扫描二维码。

    5. 现在在 PiKVM 登录页面上，您需要在`2FA验证`后输入6位验证码进行登录。

    *如果你无法使用 Google 服务，你同样可以使用*
    ***[Microsoft Authenticator](https://www.microsoft.com/zh-cn/security/mobile-authenticator-app)***
    *来实现 2FA 功能，步骤与上述一致*

所有 Web UI 用户都需要在登录时输入一次性密码。
换言之，**密钥对所有用户都是相同的**。

!!! note "注意"
    使用 2FA 进行 API 或 VNC 身份验证时，您需要将一次性密码附加到密码后面，不要留空格。
    例如：密码是 `foobar` ，一次性密码是 `123456`，那么你需要使用 `foobar123456` 作为密码。

要查看密钥的当前二维码，请使用命令`kvmd-totp show`。

要禁用2FA并删除密钥，请使用命令`kvmd-totp del`。

-----

## 会话过期(Session expiration)

从 KVMD 4.53 开始，在 PiKVM Web UI 登录页面，你可以选择身份验证会话的最大持续时间：
1 小时、12 小时或无限制（直到 PiKVM 重启或 `kvmd` 系统服务重启）。
所选的会话持续时间仅适用于此浏览器和此用户。
当时间到期时，身份验证 cookie 会被撤销。
它不会影响同一用户在其他浏览器中的会话。

请注意，如果你点击 **Logout** 按钮，它将注销该用户在所有浏览器中的所有会话。

!!! note "长时间连接"

    PiKVM 积极使用 WebSockets 和长期 HTTP 连接进行视频流传输。

    如果会话过期，将导致其授权 cookie 被撤销，
    且使用该授权 cookie 的新连接将无法建立。
    然而，长期连接不会被终止，直到用户关闭浏览器标签页。
    会话过期功能主要用于在用户关闭浏览器时“清理”，但没有点击注销按钮。

    未来我们计划添加立即终止过期连接的功能。

??? example "如何设置全局会话过期限制"

    你可以设置默认过期时间，以限制用户创建无限会话的能力。
    这将是一个不可见的限制，适用于 KVM 登录 Web UI（但**不适用于 VNC**，请注意 VNC 会话始终是无限的）。

    1. 编辑文件 `/etc/kvmd/override.yaml`：

        ```yaml
        kvmd:
            auth:
                expire: 21600  # 21600 秒即 6 小时
        ```

    2. 重启 `kvmd` 服务并确保该限制已应用：

        ```console
        [root@opi-kvm ~]# systemctl restart kvmd
        [root@opi-kvm ~]# journalctl -u kvmd -g 'Maximum user session'
        ... INFO --- Maximum user session time is limited: 6:00:00
        ```

-----

## 禁用身份验证

如果需要，你可以禁用 KVM 访问的身份验证（Web UI、VNC 等，除了 SSH）。

!!! warning

    不要在不信任的网络中这样做，因为你可能会给潜在的攻击者访问目标机器的权限。

    如果你真的需要此功能，请考虑禁用 Web 终端，以免开放对 PiKVM 控制台的 shell 访问。
    你仍然可以使用 SSH 访问控制台。

??? example "如何禁用身份验证"

    1. 编辑文件 `/etc/kvmd/override.yaml`：

        ```yaml
        kvmd:
            auth:
                enabled: false
        ```

    2. 重启 `kvmd`，可选地禁用 Web 终端并切换文件系统为只读模式：

        ```console
        [root@opi-kvm ~]# systemctl restart kvmd
        [root@opi-kvm ~]# systemctl disable --now kvmd-webterm  # 如果你有 SSH 访问权限，这一步是可选的
        ```
