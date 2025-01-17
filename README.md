# 介绍

https://www.jamesflare.com/vless-tcp-reality-xtls-utls-xudp/

Sing-box和传统的软件client端，server端不太一样，一个安装包，设置不同的参数，就可以是客户端，服务器端。

* Sing-box，作为一个客户端，如果希望使用和支持hysteria2协议，并不要求服务器端必须是Sing-box，服务器端可以是独立安装的hysteria2，也可以是Sing-box上启用hysteria2协议。
* Sing-box 支持多种的协议，仅仅用hysteria2举例。reality协议，也是类似。

# 多协议

服务器端sing-box 运行hysteria2 和Reality 和客户端的配置文件。


## 服务器端

<details><summary>服务器端hysteria2和reality 配置</summary>

```
{
    "inbounds": [
        {
            "type": "hysteria2",
            "listen": "::",
            "listen_port": 8444,
            "users": [
                {
                    "password": "chenshake" 
                }
            ],
            "masquerade": "https://bing.com",
            "tls": {
                "enabled": true,
                "alpn": [
                    "h3"
                ],
                "certificate_path": "/etc/hysteria/cert.pem",
                "key_path": "/etc/hysteria/private.key"
            }
        },
        {
            "type": "vless",
            "listen": "::",
            "listen_port": 8445,
            "users": [
                {
                    "uuid": "52d8b1cc-e333-4b76-a7a4-9fd16e8b", // vps上执行 sing-box generate uuid
                    "flow": "xtls-rprx-vision"
                }
            ],
            "tls": {
                "enabled": true,
                "server_name": "www.tesla.com",
                "reality": {
                    "enabled": true,
                    "handshake": {
                        "server": "www.tesla.com",
                        "server_port": 443
                    },
                    "private_key": "0HcmI26kHsVTwUTmgRx06QgilKkV2vBkAmO02Gw", // vps上执行 sing-box generate reality-keypair
                    "short_id": [
                        "0123456789abcdef" // 0到f，长度为2的倍数，长度上限为16，默认这个也可以
                    ]
                }
            }
        }
    ],
    "outbounds": [
        {
            "type": "direct"
        }
    ]
}

```
</details> 






# 自己签发证书

证书存放位置：/etc/hysteria/。 hysteria2单独安装，还是通过sing-box server支持hysteria2协议，都是使用该证书。


```
openssl req -x509 -nodes -newkey ec:<(openssl ecparam -name prime256v1) -keyout /etc/hysteria/private.key -out /etc/hysteria/cert.crt -subj "/CN=bing.com" -days 36500 && sudo chown hysteria /etc/hysteria/private.key && sudo chown hysteria /etc/hysteria/cert.crt
```

# 服务器端和mac客户端安装和配置

<details><summary>服务器端hysteria2 配置</summary>

安装

```
bash <(curl -fsSL https://get.hy2.sh/)
```

修改 /etc/hysteria/config.yaml 配置文件

```
# 可以任意端口
listen: :8843

tls:
  cert: /etc/hysteria/cert.crt
  key: /etc/hysteria/private.key

quic:
  initStreamReceiveWindow: 8388608
  maxStreamReceiveWindow: 8388608
  initConnReceiveWindow: 20971520
  maxConnReceiveWindow: 20971520
  maxIdleTimeout: 30s
  maxIncomingStreams: 1024
  disablePathMTUDiscovery: false

bandwidth:
  up: 0 gbps
  down: 0 gbps

ignoreClientBandwidth: false
disableUDP: false
udpIdleTimeout: 60s

# 密码记得修改
auth:
  type: password
  password: chenshake

resolver:
  type: https
  https:
    addr: 1.1.1.1:443
    timeout: 10s
    sni: cloudflare-dns.com
    insecure: false

acl:
  inline:
    - reject(geoip:cn)

masquerade:
  type: proxy
  proxy:
    url: https://github.com/
    rewriteHost: true
  listenHTTP: :80
  listenHTTPS: :443
  forceHTTPS: true

```
</details> 


<details><summary>sing-box macos 客户端 hysteria 2配置</summary>


服务器端手工安装hysteria2

```
{
  "dns": {
    "servers": [
      {
        "tag": "cf",
        "address": "https://1.1.1.1/dns-query"
      },
      {
        "tag": "local",
        "address": "223.5.5.5",
        "detour": "direct"
      },
      {
        "tag": "block",
        "address": "rcode://success"
      }
    ],
    "rules": [
      {
        "geosite": "category-ads-all",
        "server": "block",
        "disable_cache": true
      },
      {
        "outbound": "any",
        "server": "local"
      },
      {
        "geosite": "cn",
        "server": "local"
      }
    ],
    "strategy": "ipv4_only"
  },
  "inbounds": [
    {
      "type": "tun",
      "inet4_address": "172.19.0.1/30",
      "auto_route": true,
      "strict_route": false,
      "sniff": true
    }
  ],
  "outbounds": [
    {
      "type": "hysteria2",
      "tag": "proxy",
      "server": "chenshake.com",             // VPS ip
      "server_port": 8843,         // 端口
      "up_mbps": 20,              //上传速率，实际填写，过大会导致流量浪费
      "down_mbps": 200,           //下载速率，实际填写，过大会导致流量浪费
      "password": "chenshake",   //hysteria2 服务密码，需要修改
      "tls": {
        "enabled": true,
        "server_name": "github.com",
        "insecure": true              
      }
    },
    {
      "type": "direct",
      "tag": "direct"
    },
    {
      "type": "block",
      "tag": "block"
    },
    {
      "type": "dns",
      "tag": "dns-out"
    }
  ],
  "route": {
    "rules": [
      {
        "protocol": "dns",
        "outbound": "dns-out"
      },
      {
        "geosite": "cn",
        "geoip": [
          "private",
          "cn"
        ],
        "outbound": "direct"
      },
      {
        "geosite": "category-ads-all",
        "outbound": "block"
      }
    ],
    "auto_detect_interface": true
  }
}


```
</details> 


<details><summary>Sing-box server hysteria2 配置</summary>

Sing-box server端安装

```
bash <(curl -fsSL https://sing-box.app/deb-install.sh)

```

```

{
  "log": {                                           #log部分可以不要，但作为服务器推荐还是保留日志
    "disabled": false,
    "level": "error",
    "output": "./log",
    "timestamp": true
  },
  "inbounds": [                                      #server的inbounds是为客户端提供服务的，接收客户端发送过来的数据
    {
      "type": "hysteria2",                           #协议类型为hysteria2
      "tag": "hy2-in",                               #这个入站的名字
      "listen": "::",                                #侦听地址
      "listen_port": 8888,                          #侦听端口
      "users": [                                     #用户密码
        {
          "password": "chenshake"
        }
      ],
      "tls": {                                       #tls配置
        "enabled": true,
        "alpn": [
          "h3"
        ],
        "certificate_path": "/etc/hysteria/cert.crt",      #证书
        "key_path": "/etc/hysteria/private.key"
      }
    }
  ],
  "outbounds": [
    {
      "type": "direct",
      "tag": "direct"
    },
    {
      "type": "block",
      "tag": "block"
    },
  ],
  "route": {
    "rules": [
      {
        "ip_is_private": true,                        #私网地址block
        "outbound": "block"
      },
      {
        "rule_set": [
          "geoip-cn",                                 #目的IP为CN
          "geosite-category-ads-all"                  #目的网站为广告网站
        ],
        "outbound": "block"                           #阻塞
      }
    ],
    "rule_set": [                                     #规则集
      {
        "tag": "geoip-cn",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/SagerNet/sing-geoip/rule-set/geoip-cn.srs",
        "download_detour": "direct"
      },
      {
        "tag": "geosite-category-ads-all",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/SagerNet/sing-geosite/rule-set/geosite-category-ads-all.srs",
        "download_detour": "direct"
      }
    ],
    "final": "direct"                                  #未匹配的规则默认以direct转发流量
  }
}



```
</details> 



<details><summary>Sing-box macos 客户端 hysteria2 配置</summary>



```
{
  "dns": {
    "servers": [
      {
        "tag": "google",
        "address": "tls://dns.google",
        "address_resolver": "alidns",
        "detour": "site"
      },
      {
        "tag": "alidns",
        "address": "223.5.5.5",
        "detour": "direct"
      },
      {
        "tag": "block",
        "address": "rcode://success"
      }
    ],
    "rules": [
      {
        "rule_set": "geosite-category-ads-all",
        "server": "block"
      },
      {
        "rule_set": [
          "geosite-gfw"
        ],
        "server": "google"
      }
    ],
    "final": "alidns",
    "disable_cache": true,
    "strategy": "prefer_ipv4"
  },
  "inbounds": [
    {
      "type": "tun",
      "address": [
        "172.20.0.1/30"
      ],
      "auto_route": true,
      "sniff": true
    }
  ],
  "outbounds": [
    {
      "type": "direct",
      "tag": "direct"
    },
    {                                                                 
      "type": "hysteria2",                                            
      "tag": "site",                                                  
      "server": "chenshake.com",                                       
      "server_port": 443,                                           
      "password": "chenshake",                                      
      "tls": {                                                        
        "enabled": true,
        "server_name": "github.com", 
        "insecure": true                                             
      }                                                               
    },
    {
      "type": "block",
      "tag": "block"
    },
    {
      "type": "dns",
      "tag": "dns-out"
    }
  ],
  "route": {
    "rules": [
      {
        "protocol": "dns",
        "outbound": "dns-out"
      }, 
      {
        "rule_set": [
          "geoip-cn"
        ],
        "outbound": "direct"
      },
      {
        "rule_set": [
          "geosite-gfw"
        ],
        "outbound": "site"
      },
      {
        "rule_set": "geosite-category-ads-all",
        "outbound": "block"
      }
    ],
    "rule_set": [
      {
        "tag": "geoip-cn",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/SagerNet/sing-geoip/rule-set/geoip-cn.srs",
        "download_detour": "site"
      },
      {                                                                                            
        "tag": "geosite-gfw",                                                          
        "type": "remote",                                                                          
        "format": "binary",                                                                                 
        "url": "https://github.com/MetaCubeX/meta-rules-dat/raw/refs/heads/sing/geo/geosite/gfw.srs",
        "download_detour": "site"                                                                         
      },
      {
        "tag": "geosite-category-ads-all",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/SagerNet/sing-geosite/rule-set/geosite-category-ads-all.srs",
        "download_detour": "site"
      }
    ],
    "final": "direct",
    "auto_detect_interface": true
  }
}


```

</details> 

