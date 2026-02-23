
### **提交给 iStoreOS 社区的问题报告**

**标题：** iStoreOS 24.10.4 (版本号 2025120511) 系统网页加载异常：部分网络接口配置页打不开，提示 404 和 JS 错误

**正文：**

各位 iStoreOS 社区的开发者和用户，大家好！

我目前遇到一个比较奇怪的问题，在我的设备上，升级到 iStoreOS 24.10.4 (固件编译日期 2025120511) 后，有大约 7 个左右的网页无法正常加载，主要集中在 **“网络”->“接口”** 相关的配置页面。其他大部分系统页面包括应用商店、基本设置等均运行正常。我已尝试升级固件（从 24.10.X 升级到 24.10.4，并再次刷写了 24.10.4），并彻底清除了浏览器缓存，但问题依旧。

**设备信息：**
*   **iStoreOS 版本：** 24.10.4
*   **固件编译日期/版本号：** 2025120511 (这个日期看起来有些超前，不知是否正常)
*   **CPU 架构：** x86_64
*   **内核版本：** 6.6.119-1-484466e2719a743506c36b4bb2103582
*   **问题页面示例：** 预期是网络接口配置页面中，与 `bonding`、`WWAN` 等协议相关的部分。

**遇到的具体现象和错误信息：**

当我尝试访问这些打不开的页面时，浏览器（Chrome/Edge）的开发者工具控制台显示以下错误：

1.  **`NetworkError: HTTP error 404 while loading class file "/luci-static/resources/protocol/bonding.js?v=..."`**
    *   **现象：** 明确指示 `bonding.js` 等 `.js` 文件未找到，返回 HTTP 404 错误。在多次刷新或尝试不同页面后，还曾出现过 `wwan.js` 和 `directip.js` 等类似的 404 错误。
2.  **`TypeError: _(...).format is not a function`**
    *   **现象：** 紧随 404 错误之后，在 `29_ports.js` 文件中报告 `format` 函数未定义。这很可能是因为其依赖的 `protocol/*.js` 文件未能成功加载所导致。

**我已进行的诊断和排除：**

为了定位问题，我进行了以下 SSH 命令检查，并排除了浏览器缓存问题：

1.  **浏览器缓存已彻底清除并使用无痕模式测试。**
2.  **通过 SSH 检查文件系统：**
    *   我登录设备，进入 `/www/luci-static/resources/protocol/` 目录。
    *   执行 `ls` 命令后，**确实发现 `bonding.js`, `wwan.js`, `directip.js` 等在浏览器报错中出现的 `.js` 文件，它们在这个目录中是缺失的**。
    *   其他例如 `3g.js`, `dhcp.js`, `ppp.js` 等普通协议文件是存在的。

    ```bash
    root@iStoreOS:/www/luci-static/resources/protocol# ls
    3g.js            6to4.js          external.js      grev6tap.js      map.js           none.js          pptp.js          static.js
    464xlat.js       dhcp.js          gre.js           ipip.js          mbim.js          ppp.js           qmi.js           unet.js
    6in4.js          dhcpv6.js        gretap.js        ipip6.js         modemmanager.js  pppoa.js         relay.js         wireguard.js
    6rd.js           dslite.js        grev6.js         l2tp.js          ncm.js           pppoe.js         sstp.js
    ```

3.  **尝试通过 `opkg` 安装缺失的 LuCI 协议包：**
    *   我尝试更新 `opkg` 软件包列表并安装 `luci-proto-bonding` 和 `luci-proto-wwan`。
    *   **结果显示这些软件包是 `Unknown package`**，这意味着在当前 iStoreOS 的软件源中没有这些包。

    ```bash
    root@iStoreOS:/www/luci-static/resources/protocol# opkg update
    # ... (更新成功，无明显错误) ...
    root@iStoreOS:/www/luci-static/resources/protocol# opkg install luci-proto-bonding
    Unknown package 'luci-proto-bonding'.
    Collected errors:
     * opkg_install_cmd: Cannot install package luci-proto-bonding.
    root@iStoreOS:/www/luci-static/resources/protocol# opkg install luci-proto-wwan
    Unknown package 'luci-proto-wwan'.
    Collected errors:
     * opkg_install_cmd: Cannot install package luci-proto-wwan.
    ```

**问题总结与初步推测：**

*   **核心问题：** 这些浏览器错误并非浏览器导致，而是因为 iStoreOS 设备上的 `/www/luci-static/resources/protocol/` 目录下**确实缺失了 `bonding.js`、`wwan.js` 等核心 LuCI 网页资源文件**。
*   **深层原因：** 这些文件缺失的原因并非是安装错误，而是因为系统在安装这些文件所需的 `luci-proto-bonding`、`luci-proto-wwan` 等软件包时**提示“Unknown package”**，表明这些软件包在 iStoreOS 的软件源中**根本不可用或已被移除**。
*   **现有推测：**
    1.  当前 `iStoreOS 24.10.4 (2025120511)` 版本可能是一个**特殊定制版、开发版或被剪裁过的版本**，为了减小体积或聚焦核心功能，移除了对 `bonding`、`WWAN` 等 OpenWrt 标准 LuCI 协议的 Web 界面支持。或者 iStoreOS 有自己的方式实现这些功能，不再依赖标准的 `luci-proto-*` 包。
    2.  `2025120511` 这个未来日期版本的固件本身可能存在异常。

**寻求帮助：**

我想请问：

1.  当前 `iStoreOS 24.10.4 (2025120511)` 这个版本是否正常？这个日期版本号是否意味着这是一个开发版本？
2.  对于 `bonding`、`WWAN` 等网络协议的配置，iStoreOS 当前版本是否通过其他方式提供 Web 配置界面？是否有对应的 iStoreOS 插件可以在应用商店安装？
3.  如果我需要这些高级网络协议的功能，官方建议是升级到哪个稳定版本，或者是否存在其他解决方案？

感谢各位的阅读和帮助！

---
