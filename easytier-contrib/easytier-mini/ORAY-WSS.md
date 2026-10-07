# Oray X1 Mini + WS/WSS

本构建面向 OrayBox X1（MT7628AN、OpenWrt `mipsel_24kc`）。
静态链接、软浮点 MIPSEL，使用 Mini 的体积优化策略。

## 下载与构建

GitHub Actions 的 **Oray X1 Mini WSS** 工作流会在 `oray-mini-wss` 或
`main` 的相关文件推送时运行，也可手动运行。下载成功运行页面底部的 artifact：

- `easytier-mini-oray-wss`：未压缩的精简程序。
- `easytier-mini-oray-wss-upx`：相同程序的 UPX 压缩版。
- `SHA256SUMS`、`SIZES.txt`、`SOURCE_COMMIT.txt`、`FEATURES.txt`：校验、体积和构建来源。
- `oray.example.toml`：无真实凭据的配置样例。

本地 Linux 交叉编译需要先准备仓库现有的 musl-cross 工具链和 Rust 1.95
`rust-src`，然后执行：

```sh
EASYTIER_MINI_FEATURES=websocket ./easytier-contrib/easytier-mini/build-mips.sh mipsel
```

不设置 `EASYTIER_MINI_FEATURES` 时仍生成原有 TCP/UDP Mini。
WS/WSS 增加 TLS 和 WebSocket 代码，最终体积以工作流的 `SIZES.txt` 为准，
不能沿用原版 Mini 的 5.5 MB 大小上限。

## 部署与配置

在本机解压 artifact，先执行 `shasum -a 256 -c SHA256SUMS`，
选择其中一个二进制上传到路由器 `/root/`。不要覆盖正在运行的程序；
先停止旧实例，再替换 `/root/easytier-mini` 并执行 `chmod 700`。
`/root` 属于持久存储，程序没有放在内存 `/tmp` 中。

保留真实 `/root/easytier.conf` 的网络名称、密钥、虚拟 IP 和 WSS 地址。
若同时运行 HEV，应在 TOML 的 `[flags]` 中设置 `dev_name = "et0"`，
避免与 HEV 的 `tun0` 重名。样例中的网段、地址和服务器均需按实际组网修改。

```sh
cd /root
./easytier-mini --config /root/easytier.conf
```

这是前台命令，Ctrl+C 会停止。需要脱离 SSH 后台运行可使用已有的 `nohup`：

```sh
nohup /root/easytier-mini --config /root/easytier.conf </dev/null >/tmp/easytier-mini.log 2>&1 &
```

未默认配置开机启动。安装后检查 Flash 余量，并实际验证对端连接和虚拟 IP 互通。
与 HEV 同时运行还需确保组网流量的路由和防火墙配置符合实际用途。

## 支持范围与验证

启用原生 WS/WSS 协议适配器，并让 compact runtime 保留 WS/WSS 节点和监听设置；
仍保留 TCP/UDP、TUN、AES-GCM、UDP 打洞和原 Mini 的 Web 管理功能。
支持 `[[proxy_network]]` 子网代理，包括路由通告与 TCP/UDP/ICMP 转发；
原 `8bdf682b` 构建会在 compact runtime 中忽略这项配置，需更新二进制才能生效。
未启用 QUIC、WireGuard、KCP、TCP 打洞或完整管理功能。
TXT DNS 查询的裁剪行为保持原样。

WSS 使用上游 EasyTier 的 TLS 实现和证书验证策略，本修改不另外改变该策略。
工作流验证默认 Mini 和 WebSocket Mini、节点配置过滤、WS/WSS 原生握手，
再用 QEMU 检查普通版及 UPX 版可执行。QEMU 检查不能替代真机的公网 WSS 组网测试。
