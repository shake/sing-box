# sing-box macos 客户端 （配置1）

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

# sing-box macos 客户端 （配置2）

服务器端：sing-box 运行 hysteria2

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



