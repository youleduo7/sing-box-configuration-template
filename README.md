# sing-box Configuration Template

> 一份面向个人使用和学习的 sing-box 配置模板与部署记录，涵盖服务端双协议部署，以及 Windows、Android 和通用客户端配置。

本项目以 **sing-box 1.14.3** 的服务端部署方案为基础，记录从 VPS 安装、证书签发、协议配置到客户端分流的实践过程。配置文件仅为模板，**需要根据自己的服务器信息修改后才能使用**。

## ✨ 项目特色

- **双协议支持**：Hysteria2（UDP 443）与 VLESS + REALITY（TCP 443）。
- **跨平台模板**：包含 Windows、Android 和通用 sing-box 客户端配置。
- **灵活的出站选择**：通过 `MODE` 切换代理或直连，通过 `PROXY` 选择 Hysteria2 或 REALITY。
- **DNS 与路由分流**：结合国内外 DNS、GeoSite / GeoIP 规则集、广告拦截及指定服务分流。
- **服务端维护记录**：包括自签 CA、服务器证书自动续期、nftables 防火墙及可选网络优化。

## 📁 仓库结构

```text
sing-box-configuration-template/
├── README.md
├── 服务端配置/
│   └── 服务端配置.md
└── 客户端配置/
    ├── sing-box模板.json
    ├── sing-box-windows模板.json
    └── sing-box-android安卓模板.json
```

| 文件 | 内容 |
| --- | --- |
| [服务端配置.md](服务端配置/服务端配置.md) | Debian 服务端部署、双协议配置、证书续期与防火墙设置 |
| [sing-box模板.json](客户端配置/sing-box模板.json) | 通用客户端配置模板 |
| [sing-box-windows模板.json](客户端配置/sing-box-windows模板.json) | Windows 客户端模板，包含 `Discord.exe` 进程分流规则 |
| [sing-box-android安卓模板.json](客户端配置/sing-box-android安卓模板.json) | Android 客户端配置模板 |

## 🚀 使用方法

### 1. 准备服务端

准备一台拥有公网 IPv4 地址的 VPS，阅读 [服务端配置指南](服务端配置/服务端配置.md)，按文档完成安装和配置。

服务端文档使用 **Debian 13** 和 **sing-box 1.14.3** 作为示例环境；其他系统或 sing-box 版本的命令、参数可能有所不同。

服务端同时使用以下入站端口：

| 协议 | 端口 | 传输 |
| --- | --- | --- |
| Hysteria2 | `443` | UDP |
| VLESS + REALITY | `443` | TCP |

另外，请按实际 SSH 端口设置防火墙，并确认 VPS 服务商的安全组已放行所需端口。

### 2. 获取客户端连接信息

根据服务端部署文档生成并妥善保存连接参数，包括：

- 服务器公网 IP
- Hysteria2 密码及客户端信任的 CA 证书
- VLESS UUID、REALITY 公钥、Short ID 与 Server Name

这些信息应当来自**你自己的服务端**。不要将真实密码、UUID、私钥或完整的客户端连接信息提交到公开仓库。

### 3. 修改客户端模板

从 [客户端配置目录](客户端配置) 选择相应模板，至少完成以下修改：

1. 将 `自己的服务器ip` 替换为服务器的实际公网 IP。
2. 将各处 `服务端的配置` 替换为实际密码、UUID、Flow 等对应参数。
3. 将 Hysteria2 的 `tls.certificate` 替换为自己的 CA 公共证书内容；也可参照服务端文档使用本地证书路径。
4. 核对 REALITY 的 `server_name`、`public_key` 和 `short_id`，确保与服务端一致。
5. 根据需要调整 DNS、分流规则、TUN 参数和默认出站。

> [!IMPORTANT]
> 当前仓库中的三个 JSON 模板在 REALITY 的 `server_name` 附近保留了以 `####使用时删掉注释` 开头的说明文字，它**不是合法的 JSON 注释**。使用前请移除这段说明，保留正确的 JSON 逗号与字段结构，并完成所有占位参数的替换。模板不应直接原样导入。

### 4. 检查并运行

启动前先使用对应版本的 sing-box 检查配置语法及字段兼容性。服务端文档中的检查命令为：

```bash
sing-box check -D /var/lib/sing-box -C /etc/sing-box
```

客户端应根据自己的系统与应用选择相应的检查、导入或启动方式。启用 TUN 模式时，可能需要管理员权限或系统 VPN 授权。

## 🧩 分流设计

客户端模板主要包含以下逻辑：

| 类型 | 行为 |
| --- | --- |
| `MODE` | 在 `PROXY` 和 `DIRECT` 之间切换 |
| `PROXY` | 在 `Hysteria2` 和 `REALITY` 之间选择 |
| 国内域名及 GeoIP / GeoSite 规则 | 优先走 `DIRECT_CN` 与国内 DNS |
| Discord 相关域名 | 指定走代理；Windows 模板还包含进程规则 |
| 部分广告规则 | 使用 `reject` 拦截 |
| 其他流量 | 根据规则和最终出站设置处理 |

模板中的远程规则集需要能够正常下载和更新，实际分流行为也可能受规则命中顺序、DNS 解析和客户端平台差异影响。

## 🔐 安全提醒

- **不要公开**服务器私钥、CA 私钥、节点密码及包含真实认证信息的配置文件。
- Hysteria2 使用自签 CA 时，客户端必须正确信任对应证书；不要为了省事直接关闭证书校验。
- 修改 nftables 防火墙前，请先确认 SSH 放行规则及回滚措施，以免失去服务器连接。
- 服务端证书有有效期，需关注自动续期任务是否正常运行。
- 本仓库是个人实践记录，并非 sing-box 官方项目，也不是一键安装程序。

## 📚 参考资料

- [sing-box 官方文档](https://sing-box.sagernet.org/)
- [sing-box 官方 GitHub 仓库](https://github.com/SagerNet/sing-box)

## 💬 关于项目

这是我完成的第一个小项目。起初只是想搭建一套适合自己使用的 sing-box 配置，后来把部署步骤、踩过的坑和客户端模板整理了下来，希望也能给有类似需求的人提供一点参考。

项目仍会随着实际使用经验和 sing-box 版本变化继续完善。如果发现配置兼容性问题或文档错误，欢迎通过 Issue 提出建议。

---

**愿这份从零开始的实践记录，能帮助更多人少走一些弯路。**
