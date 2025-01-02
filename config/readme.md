目前伪装最好的两种协议，一个代表tcp协议（reality）的目前巅峰，一个新型UDP协议（hysteria2）。sing-box 支持两种协议，但是并不是所有功能都支持，hysteria2端口跳跃功能是不支持的。如果你希望使用这个功能，那么就只能独立部署hysteria2。

# 服务器端安装sing-box

采用官方的脚本 https://sing-box.sagernet.org/zh/installation/package-manager/

```
bash <(curl -fsSL https://sing-box.app/deb-install.sh)

```
启动sing-box服务


- 启动

```bash
systemctl start sing-box
```

- 状态

```bash
systemctl status sing-box
```

- 实时日志

```bash
journalctl -u sing-box -o cat -f
```

# 准备证书

- 自签证书申请，这里申请的是bing.com，申请了10年
  
```
mkdir -p /etc/hysteria && openssl ecparam -genkey -name prime256v1 -out /etc/hysteria/private.key && openssl req -new -x509 -days 3650 -key /etc/hysteria/private.key -out /etc/hysteria/cert.pem -subj "/CN=bing.com"
```

uud

```
sing-box generate uuid
```
Reality 使用私钥

```
sing-box generate reality-keypair
```

# /etc/sing-box/config.json

参考上面的服务器端端配置文件就可以。


# 客户端

客户端，mac下，使用sing-box。上面两个client的配置文件，分别使用reality 和hysteria2 。








