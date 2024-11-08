---
title: "Clash配置示例"
date: 2021-04-13T18:38:14+08:00
draft: false
tags: ["网络工具"]
categories: ["折腾日记"]
typora-root-url: ./
typora-copy-images-to: ./
---

## Clash配置示例

### Config.yaml

#### 1. PC And Android

```yaml
# 混合代理端口
mixed-port: 1080

# 允许局域网的连接
allow-lan: true

# 绑定局域网ip
bind-address: "*"

# Clash 的 RESTful API
external-controller: "127.0.0.1:9090"

# 您可以将静态网页资源（如 clash-dashboard）放置在一个目录中，clash 将会服务于 `${API}/ui`
# external-ui: folder

# RESTful API 的口令
secret: ""

# 规则模式：Rule（规则） / Global（全局代理）/ Direct（全局直连）
mode: Rule

# 设置日志输出级别 (默认级别：silent，即不输出任何内容，以避免因日志内容过大而导致程序内存溢出）。
# 5 个级别：silent / info / warning / error / debug。级别越高日志输出量越大，越倾向于调试，若需要请自行开启。
log-level: info

ipv6: false

# 控制是否让Clash 去匹配进程,always开启，强制匹配所有进程，strict 默认，由 Clash 判断是否开启，off 不匹配进程，推荐在路由器上使用此模式
find-process-mode: strict

# 开启统一延迟时，会进行两次延迟测试，以消除连接握手等带来的不同类型节点的延迟差异
unified-delay: false

# 全局客户端指纹，优先低于 proxy 内的 client-fingerprint，目前支持开启 TLS 传输的 TCP/grpc/WS/HTTP , 支持协议有 VLESS,Vmess 和 trojan.
# 可选：chrome, firefox, safari, iOS, android, edge, 360, qq, random, 若选择 random, 则按 Cloudflare Radar 数据按概率生成一个现代浏览器指纹。
global-client-fingerprint: chrome

# TCP 并发
tcp-concurrent: true

# 全局UA
global-ua: clash.meta

profile:
    store-selected: true
    # 储存 API 对策略组的选择，以供下次启动时使用
    store-fake-ip: false
    # 储存 fakeip 映射表，域名再次发生连接时，使用原有映射地址

# host:
#     gist.github.com: 192.30.255.112
#     gist.githubusercontent.com: 151.101.56.133

sniffer:
    enable: true
    force-dns-mapping: true
    parse-pure-ip: true
    override-destination: true
    sniff:
        HTTP:
            ports: [80, 8080-8880]
            override-destination: true
        TLS:
            ports: [443, 8443]
        QUIC:
            ports: [443, 8443]
    force-domain:
        - "+.v2ex.com"
        - "+.google.com"
    skip-domain:
        - "Mijia Cloud"
        - "dlg.io.mi.com"
        - "+.apple.com"

dns:
    enable: true
    listen: 0.0.0.0:53
    ipv6: false
    prefer-h3: false
    respect-rules: false
    use-hosts: true # lookup hosts and return IP record
    use-system-hosts: true
    # fake-ip/redir-host/normal
    enhanced-mode: fake-ip
    # fake ip 池
    fake-ip-range: 198.18.0.1/16
    # fake ip 白名单
    fake-ip-filter:
        - "*.lan"
        - "localhost"
    # 用于解析nameserver中的域名
    default-nameserver:
        - 114.114.114.114
        - 119.29.29.29
    nameserver:
        - 223.5.5.5
        - 119.29.29.29
    # 与 nameserver 内的服务器列表同时发起请求，当规则符合 GEOIP 在 CN 以外时，fallback 列表内的域名服务器生效。
    fallback:
        - https://doh.pub/dns-query
        - https://dns.alidns.com/resolve
        - tls://dot.pub:853
    proxy-server-nameserver:
        - https://doh.pub/dns-query
        - https://dns.alidns.com/resolve
        - tls://dot.pub:853
    # nameserver-policy:
    #     "www.baidu.com": "114.114.114.114"
    fallback-filter:
        geoip: true # 默认
        geoip-code: CN
        ipcidr: # 在这个网段内的 IP 地址会被考虑为被污染的 IP
            - 240.0.0.0/4
        domain:
            - "+.google.com"
            - "+.facebook.com"
            - "+.youtube.com"

tun:
    enable: true
    stack: mixed # gvisor 或者 system 或者mixed
    dns-hijack:
        - any:53
    auto-route: true
    auto-redirect: true
    auto-detect-interface: true
    strict-route: true

proxy-providers-common: &proxy-providers-common
    type: http
    interval: 43200
    # filter: "(?i)港|hk|hongkong|hong kong" # 筛选满足关键词或正则表达式的节点
    # exclude-filter: "xxx" # 排除满足关键词或正则表达式的节点
    # exclude-type: "ss|http" # 不支持正则表达式，通过 | 分割，根据节点类型排除
    header:
        User-Agent:
            - "Clash/v1.18.0"
            - "mihomo/1.18.3"
        # Authorization:
        #     - 'token 1231231'
    health-check:
        enable: true
        url: https://www.gstatic.com/generate_204
        interval: 300
        timeout: 5000
        lazy: true
    override:
        skip-cert-verify: false
        # udp: true
        # down: "50 Mbps"
        # up: "10 Mbps"
        # dialer-proxy: proxy
        # interface-name: tailscale0
        # routing-mark: 233
        # ip-version: ipv4-prefer
        # additional-prefix: "provider1 prefix |"
        # additional-suffix: "| provider1 suffix"

# github代理：https://gh.chjina.com/ https://ghproxy.com/ https://gh.not.icu/
proxy-providers:
    ProxyHome:
        <<: *proxy-providers-common
        path: ./Proxies/ProxyHome.yaml
        url: url
    ProxyNetease:
        <<: *proxy-providers-common
        path: ./Proxies/ProxyNetease.yaml
        url: url
    ProxyFree:
        <<: *proxy-providers-common
        path: ./Proxies/ProxyFree.yaml
        url: url
        interval: 3600

proxy-groups-common: &proxy-groups-common
    url: "https://www.gstatic.com/generate_204"
    interval: 300 # 健康检查间隔，如不为 0则启用定时测试，单位为秒
    lazy: true
    timeout: 5000
    max-failed-times: 5
    # disable-udp: true
    # interface-name: en0
    # routing-mark: 11451
    # include-all: false
    # include-all-proxies: false
    # include-all-providers: false
    # filter: "(?i)港|hk|hongkong|hong kong"
    # exclude-filter: "美|日"
    # exclude-type: "Shadowsocks|Http"
    # expected-status: 204
    # hidden: true
    # icon: xxx

# 代理组策略
proxy-groups:
    - name: "🐟默认"
      <<: *proxy-groups-common
      type: select
      proxies:
          - "🍪国内服务"
          - "🚀国际服务"

    - name: "🚀国际服务"
      <<: *proxy-groups-common
      type: select
      proxies:
          - "🍀IKUUU-延迟最低"
          - "🍀IKUUU-手动选择"
          - DIRECT
          - 🏠回家

    - name: "🍪国内服务"
      <<: *proxy-groups-common
      type: select
      proxies:
          - DIRECT
          - 🏠回家
          - "🍀IKUUU-延迟最低"
          - "🍀IKUUU-手动选择"

    - name: "🍀IKUUU-手动选择"
      <<: *proxy-groups-common
      type: select
      use:
          - ProxyFree

    - name: "🍀IKUUU-延迟最低"
      <<: *proxy-groups-common
      type: url-test
      tolerance: 100 # 节点切换容差，单位 ms
      lazy: true
      use:
          - ProxyFree

    - name: "💤测速"
      <<: *proxy-groups-common
      type: select
      proxies:
          - "🚀国际服务"
          - "🍪国内服务"
          - "DIRECT"

    - name: "🎵网易云"
      <<: *proxy-groups-common
      type: select
      use:
          - ProxyNetease
      proxies:
          - "DIRECT"

    - name: "🏠回家"
      <<: *proxy-groups-common
      type: select
      use:
          - ProxyHome

rule-providers:
    GlobalMedia:
        type: http
        behavior: classical
        path: ./Rules/GlobalMedia.yaml
        url: https://gh.chjina.com/https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/Providers/ProxyMedia.yaml
        interval: 86400
    ChinaMedia:
        type: http
        behavior: classical
        path: ./Rules/ChinaMedia.yaml
        url: https://gh.chjina.com/https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/Providers/ChinaMedia.yaml
        interval: 86400
    Global:
        type: http
        behavior: classical
        path: ./Rules/Global.yaml
        url: https://gh.chjina.com/https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/Providers/ProxyGFWlist.yaml
        interval: 86400
    China:
        type: http
        behavior: classical
        path: ./Rules/China.yaml
        url: https://gh.chjina.com/https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/Providers/ChinaDomain.yaml
        interval: 86400
    ChinaIp:
        type: http
        behavior: ipcidr
        path: ./Rules/ChinaIp.yaml
        url: https://gh.chjina.com/https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/Providers/ChinaIp.yaml
        interval: 86400
    Speedtest:
        type: http
        behavior: classical
        path: ./Rules/Speedtest.yaml
        url: url
        interval: 86400
    CustomGlobal:
        type: http
        behavior: classical
        path: ./Rules/CustomGlobal.yaml
        url: url
        interval: 86400
    CustomChina:
        type: http
        behavior: classical
        path: ./Rules/CustomChina.yaml
        url: url
        interval: 86400
    LocalAreaNetwork:
        type: http
        behavior: classical
        path: ./Rules/LocalAreaNetwork.yaml
        url: https://gh.chjina.com/https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/Providers/LocalAreaNetwork.yaml
        interval: 86400
    NeteaseAndroid:
        type: http
        behavior: classical
        path: ./Rules/NeteaseAndroid.yaml
        url: https://gh.chjina.com/https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Clash/NetEaseMusic/NetEaseMusic_No_Resolve.yaml
        interval: 86400
    CustomNeteaseAndroid:
        type: http
        behavior: classical
        path: ./Rules/CustomNeteaseAndroid.yaml
        url: url
        interval: 86400

# 规则
rules:
    # 网易云音乐APP
    - PROCESS-NAME,cloudmusic.exe,🎵网易云
    - PROCESS-NAME,com.netease.cloudmusic,🎵网易云
    - RULE-SET,NeteaseAndroid,🎵网易云
    - RULE-SET,CustomNeteaseAndroid,🎵网易云
    # 代理服务器
    - PROCESS-NAME,Clash.exe,DIRECT
    - PROCESS-NAME,xray.exe,DIRECT
    - PROCESS-NAME,wv2ray.exe,DIRECT
    - PROCESS-NAME,v2ray.exe,DIRECT

    - RULE-SET,CustomChina,🍪国内服务
    - RULE-SET,CustomGlobal,🚀国际服务
    - RULE-SET,Speedtest,💤测速
    - RULE-SET,ChinaMedia,🍪国内服务
    - RULE-SET,China,🍪国内服务
    - RULE-SET,GlobalMedia,🚀国际服务
    - RULE-SET,Global,🚀国际服务

    ############## IP规则 ##################

    # 家庭局域网
    - IP-CIDR,192.168.2.0/24,🏠回家
    # Local Area Network
    - RULE-SET,LocalAreaNetwork,DIRECT
    # 使用来自 ipipdotnet 的 ChinaIp 以解决数据不准确的问题，使用 ChinaIp.yaml 时可禁用下列直至（包括）「GEOIP,CN」规则
    - RULE-SET,ChinaIp,🍪国内服务
    # GeoIP China
    # - GEOIP,CN,DIRECT
    - MATCH,🐟默认
```

### Rules

#### 1. RuleCustomGlobal.yaml

```yaml
payload:
    - DOMAIN-SUFFIX,yugogo.xyz
    - DOMAIN-SUFFIX,celsoazevedo.com
    - DOMAIN-SUFFIX,typora.io
    - DOMAIN-SUFFIX,chocolatey.org
    - DOMAIN-SUFFIX,yandex.com
    - DOMAIN-SUFFIX,github.io
    - DOMAIN-SUFFIX,githubapp.com
    - DOMAIN-SUFFIX,vmware.com
    - DOMAIN-SUFFIX,v2fly.org
    - DOMAIN-SUFFIX,telegram.me
    - DOMAIN-SUFFIX,t.me
    - DOMAIN-SUFFIX,githubassets.com
    - DOMAIN-SUFFIX,v2ex.com
    - DOMAIN-SUFFIX,githubusercontent.com
    - DOMAIN-SUFFIX,apkpure.com
    - DOMAIN-SUFFIX,doubiyunbackup.com
    - DOMAIN-SUFFIX,repo.maven.apache.org
    - DOMAIN-SUFFIX,google.com
    - DOMAIN-SUFFIX,bitbucket.org
    - DOMAIN-SUFFIX,github.com
    - DOMAIN-SUFFIX,github-production-release-asset-2e65be.s3.amazonaws.com
    - DOMAIN-SUFFIX,homebrew.bintray.com
    - DOMAIN-SUFFIX,jetbrains.com
    - DOMAIN-SUFFIX,download.docker.com
    - DOMAIN-SUFFIX,s3.amazonaws.com
    - DOMAIN-SUFFIX,atlassian.com
    - DOMAIN-SUFFIX,downloads.vivaldi.com
    - DOMAIN-SUFFIX,panic.com
    - DOMAIN-SUFFIX,fatcatsoftware.com
    - DOMAIN-SUFFIX,freedownloadmanager.org
    - DOMAIN-SUFFIX,vo.msecnd.net
    - DOMAIN-SUFFIX,zeroturnaround.com
    - DOMAIN-SUFFIX,graphviz.org
    - DOMAIN-SUFFIX,dl.iina.io
    - DOMAIN-SUFFIX,softs.fun
    - DOMAIN-SUFFIX,wireshark.org
    - DOMAIN-SUFFIX,atom.io
    - DOMAIN-SUFFIX,maven.apache.org
    - DOMAIN-SUFFIX,233v2.com
    - DOMAIN-SUFFIX,sentry.io
    - DOMAIN-SUFFIX,nvidia.cn
    - DOMAIN-SUFFIX,inter.com
    - DOMAIN-SUFFIX,stackoverflow.com
    - DOMAIN-SUFFIX,freenom.com
    - DOMAIN-SUFFIX,thepiratebay.org
    - DOMAIN-SUFFIX,killcovid2021.com
```

#### 2. RuleLocalAreaIP.yaml

```yaml
payload:
    # 本地/局域网地址
    # 参考：https://en.wikipedia.org/wiki/Reserved_IP_addresses

    # ACL4SSR标志 如没有，代表不是用ACL4SSR规则
    - DOMAIN-SUFFIX,acl4.ssr

    # 本地/局域网地址
    - DOMAIN-SUFFIX,ip6-localhost
    - DOMAIN-SUFFIX,ip6-loopback
    - DOMAIN-SUFFIX,lan
    - DOMAIN-SUFFIX,local
    - DOMAIN-SUFFIX,localhost
    - IP-CIDR,0.0.0.0/8,no-resolve
    - IP-CIDR,10.0.0.0/8,no-resolve
    - IP-CIDR,100.64.0.0/10,no-resolve
    - IP-CIDR,127.0.0.0/8,no-resolve
    - IP-CIDR,172.16.0.0/12,no-resolve
    - IP-CIDR,192.168.0.0/16,no-resolve
    - IP-CIDR,198.18.0.0/16,no-resolve
    - IP-CIDR,224.0.0.0/4,no-resolve
    - IP-CIDR6,::1/128,no-resolve
    - IP-CIDR6,fc00::/7,no-resolve
    - IP-CIDR6,fe80::/10,no-resolve
    - IP-CIDR6,fd00::/8,no-resolve

    # Router managed 路由器管理域名
    - DOMAIN,instant.arubanetworks.com
    - DOMAIN,setmeup.arubanetworks.com
    - DOMAIN,router.asus.com
    - DOMAIN,www.asusrouter.com
    - DOMAIN-SUFFIX,hiwifi.com
    - DOMAIN-SUFFIX,leike.cc
    - DOMAIN-SUFFIX,miwifi.com
    - DOMAIN-SUFFIX,my.router
    - DOMAIN-SUFFIX,p.to
    - DOMAIN-SUFFIX,peiluyou.com
    - DOMAIN-SUFFIX,phicomm.me
    - DOMAIN-SUFFIX,router.ctc
    - DOMAIN-SUFFIX,routerlogin.com
    - DOMAIN-SUFFIX,tendawifi.com
    - DOMAIN-SUFFIX,zte.home
    - DOMAIN-SUFFIX,tplogin.cn
    - DOMAIN-SUFFIX,wifi.cmcc
```

#### 3. RuleNeteaseAndroid.yaml

```yaml
payload:
    # Netease Music
    - DOMAIN,apm3.music.163.com
    - DOMAIN,apm.music.163.com
    - DOMAIN,interface3.music.163.com
    - DOMAIN,interface3.music.163.com.163jiasu.com
    - DOMAIN,interface.music.163.com
    - DOMAIN,music.163.com
    - IP-CIDR,39.105.63.80/32,no-resolve
    - IP-CIDR,39.105.175.128/32,no-resolve
    - IP-CIDR,45.254.48.1/32,no-resolve
    - IP-CIDR,45.254.48.2/32,no-resolve
    - IP-CIDR,47.100.127.239/32,no-resolve
    - IP-CIDR,59.111.19.33/32,no-resolve
    - IP-CIDR,59.111.160.195/32,no-resolve
    - IP-CIDR,59.111.160.197/32,no-resolve
    - IP-CIDR,59.111.181.35/32,no-resolve
    - IP-CIDR,59.111.181.38/32,no-resolve
    - IP-CIDR,59.111.181.60/32,no-resolve
    - IP-CIDR,101.71.154.241/32,no-resolve
    - IP-CIDR,101.71.154.243/32,no-resolve
    - IP-CIDR,103.126.92.132/32,no-resolve
    - IP-CIDR,103.126.92.133/32,no-resolve
    - IP-CIDR,112.13.119.17/32,no-resolve
    - IP-CIDR,112.13.119.18/32,no-resolve
    - IP-CIDR,112.13.122.1/32,no-resolve
    - IP-CIDR,112.13.122.4/32,no-resolve
    - IP-CIDR,115.236.118.33/32,no-resolve
    - IP-CIDR,115.236.118.34/32,no-resolve
    - IP-CIDR,115.236.121.1/32,no-resolve
    - IP-CIDR,115.236.121.4/32,no-resolve
    - IP-CIDR,118.24.63.156/32,no-resolve
    - IP-CIDR,120.226.43.98/32,no-resolve
    - IP-CIDR,120.226.43.99/32,no-resolve
    - IP-CIDR,125.44.162.194/32,no-resolve
    - IP-CIDR,125.44.162.222/32,no-resolve
    - IP-CIDR,182.92.170.253/32,no-resolve
    - IP-CIDR,183.134.54.31/32,no-resolve
    - IP-CIDR,183.134.54.33/32,no-resolve
    - IP-CIDR,193.112.159.225/32,no-resolve
    - IP-CIDR,223.252.199.66/32,no-resolve
    - IP-CIDR,223.252.199.67/32,no-resolve
    # 自定义
    - DOMAIN,ac.dun.163yun.com
    - DOMAIN,api.iplay.163.com
    - DOMAIN,api2.music.163.com
    - DOMAIN,apm3.music.163.com
    - DOMAIN,c.dun.163yun.com
    - DOMAIN,c.m.163.com
    - DOMAIN,clientlog3.music.163.com
    - DOMAIN,comment.api.163.com
    - DOMAIN,conf-darwin.xycdn.com
    - DOMAIN,d1.music.126.net
    - DOMAIN,d2.music.126.net
    - DOMAIN,d3.music.126.net
    - DOMAIN,d4.music.126.net
    - DOMAIN,d5.music.126.net
    - DOMAIN,d6.music.126.net
    - DOMAIN,da.dun.163.com
    - DOMAIN,da.qiyukf.com
    - DOMAIN,dy.163.com
    - DOMAIN,gslb.live.126.net
    - DOMAIN,iadmat.nosdn.127.net
    - DOMAIN,iadmusicmat.music.126.net
    - DOMAIN,iadmusicmatvideo.music.126.net
    - DOMAIN,interface.music.163.com
    - DOMAIN,interface3.music.163.com
    - DOMAIN,interface3.music.163.com.163jiasu.com
    - DOMAIN,ipservice.ws.126.net
    - DOMAIN,lbs.netease.im
    - DOMAIN,login.sina.com.cn
    - DOMAIN,m7.music.126.net
    - DOMAIN,m701.music.126.net
    - DOMAIN,m702.music.126.net
    - DOMAIN,m705.music.126.net
    - DOMAIN,m8.music.126.net
    - DOMAIN,m801.music.126.net
    - DOMAIN,m802.music.126.net
    - DOMAIN,m805.music.126.net
    - DOMAIN,mam.netease.com
    - DOMAIN,mam6.netease.com
    - DOMAIN,nstool.netease.com
    - DOMAIN,p0.ssl.img.360kuai.com
    - DOMAIN,p1.music.126.net
    - DOMAIN,p101.music.126.net
    - DOMAIN,p2.music.126.net
    - DOMAIN,p3.music.126.net
    - DOMAIN,p4.music.126.net
    - DOMAIN,p5.music.126.net
    - DOMAIN,p6.music.126.net
    - DOMAIN,s1.music.126.net
    - DOMAIN,s6.music.126.net
    - DOMAIN,sdk1xyajs.data.p2cdn.com
    - DOMAIN,st.music.163.com
    - DOMAIN,sta-op.douyucdn.cn
    - DOMAIN,vodkgeyttp8.vod.126.net
    - DOMAIN,vodkgeyttp9.vod.126.net
    - DOMAIN,www.m701.music.126.net
    - IP-CIDR,39.105.63.80/32,no-resolve
    - IP-CIDR,39.105.175.128/32,no-resolve
    - IP-CIDR,45.254.48.2/32,no-resolve
    - IP-CIDR,47.100.127.239/32,no-resolve
    - IP-CIDR,54.223.118.211/32,no-resolve
    - IP-CIDR,59.111.19.33/32,no-resolve
    - IP-CIDR,59.111.36.167/32,no-resolve
    - IP-CIDR,59.111.160.195/32,no-resolve
    - IP-CIDR,59.111.160.197/32,no-resolve
    - IP-CIDR,59.111.179.213/32,no-resolve
    - IP-CIDR,59.111.179.214/32,no-resolve
    - IP-CIDR,59.111.210.180/32,no-resolve
    - IP-CIDR,59.111.239.61/32,no-resolve
    - IP-CIDR,59.111.239.62/32,no-resolve
    - IP-CIDR,60.169.6.91/32,no-resolve
    - IP-CIDR,60.169.6.100/32,no-resolve
    - IP-CIDR,61.184.215.69/32,no-resolve
    - IP-CIDR,61.184.215.74/32,no-resolve
    - IP-CIDR,101.71.154.243/32,no-resolve
    - IP-CIDR,106.14.12.130/32,no-resolve
    - IP-CIDR,115.236.118.34/32,no-resolve
    - IP-CIDR,115.236.121.4/32,no-resolve
    - IP-CIDR,115.236.121.51/32,no-resolve
    - IP-CIDR,115.236.121.195/32,no-resolve
    - IP-CIDR,115.238.119.68/32,no-resolve
    - IP-CIDR,118.24.63.156/32,no-resolve
    - IP-CIDR,122.225.83.109/32,no-resolve
    - IP-CIDR,122.225.83.110/32,no-resolve
    - IP-CIDR,182.92.170.253/32,no-resolve
    - IP-CIDR,183.134.34.249/32,no-resolve
    - IP-CIDR,183.136.182.19/32,no-resolve
    - IP-CIDR,193.112.159.225/32,no-resolve
    - IP-CIDR,199.190.44.36/32,no-resolve
    - IP-CIDR,223.242.32.32/32,no-resolve
```

