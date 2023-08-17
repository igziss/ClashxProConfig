# Clashx Pro 自定义配置文件
### 本配置只针对clash的初学者，写的也有些拙劣，不喜勿喷，谢谢！
1. Clash Premium 自定义规则集(RULE-SET)，兼容 ClashX Pro、Clash for Windows 客户端。
2. Clash Premium <a href="https://install.appcenter.ms/users/clashx/apps/clashx-pro/distribution_groups/public">MacOS 下载链接</a> 和 <a href="https://github.com/Fndroid/clash_for_windows_pkg/releases">Windows 下载链接</a>
3. 用Clash工具的，大多数都是使用订阅链接的小伙伴。
4. 前段时间我在用订阅链接，想自定义一些满足自己使用的规则，每次更新订阅链接，自定义的规则都将会被覆盖(好像说了句废话)。
5. 于是各种查资料（感谢 <a href="https://www.jamesdailylife.com/rule-proxy-provider">https://www.jamesdailylife.com/rule-proxy-provider</a> 大佬文章提供的帮助）以及clash的<a href="https://dreamacro.github.io/clash/configuration/configuration-reference.html">文档</a>，找到了一个我个人认为比较好用的方法。
6. Proxy Provider 是 Clash 的一项功能，可以让用户从指定路径动态加载代理服务器列表。使用这个功能你可以将 Clash 订阅里面的代理服务器提取出来，放到你喜欢的配置文件里，也可以将多个 Clash 订阅里的代理服务器混合到一个配置文件里。
7. 大概讲一下原理：
   + proxy-providers：提供节点（根据订阅链接产生）
   + rule-providers：提供规则集，不同的在线规则集，以 url 链接形式提供，包含常见的域名和IP 地址列表，由项目大佬们实时维护的
   + proxy-groups：以“策略组”为单位，对 proxy-providers 提供的节点或者手动添加的节点进行分组管理 -- 控制分流访问
   + rules：引用规则集（对rule-providers设定走直连or代理）或手动自定义一些规则

### 废话不多说上配置文件代码

```yaml
# 基本信息  端口号等信息根据自己需求场景填写
mixed-port: 7890
allow-lan: true
bind-address: '*'
mode: rule
log-level: info
external-controller: '127.0.0.1:9090'


# 这里配置不同的订阅链接 需要添加新的 则在下一层(XFLTD同级)复制一份修改即可
# 本次演示用了两个订阅链接 即：
#    1. 下方的 XFLTD 和 XFLTDGPT使用《你的订阅链接1》
#    2. 下方的 RV 和 RVGPT使用《你的订阅链接2》
proxy-providers:
  XFLTD: 
    type: http    #类型 只有 file（拉到本地使用） 和 http 两种，大多数人使用的都是订阅链接 即http
    path: ./profiles/proxies/XFLTD.yaml  # 存放下方url拉到本地的存放目录（自己随便定义目录，有读写权限即可）
    url: "你的订阅链接1"
    interval: 3600  # 自动更新时间
    filter: '0'   # filter自定义模糊匹配<订阅中的 节点 名字--可以是中文英文随便符号>！！！匹配多个用 | 符号隔开
    health-check:
      enable: true
      url: http://www.gstatic.com/generate_204
      interval: 86400

# 从XFLTD分组出来给GPT用的，匹配内容 chatgpt 开放的国家
  XFLTDGPT: 
    type: http
    path: ./profiles/proxies/XFLTDGPT.yaml
    url: "你的订阅链接1"
    interval: 86400 
    filter: '日本|韩国|美国' 
    health-check:
      enable: true
      url: http://www.gstatic.com/generate_204
      interval: 300

  RV: 
    type: http
    path: ./profiles/proxies/RV.yaml
    url: "你的订阅链接2"
    interval: 86400 
    # filter: '日本|韩国|美国'
    health-check:
      enable: true
      url: http://www.gstatic.com/generate_204
      interval: 300

# 从RV分组出来给GPT用的，filter自定义模糊匹配节点名字(节点名称为中文就匹配中文，是英文就匹配英文)，匹配内容 chatgpt 开放的国家
  RVGPT:
    type: http
    path: ./profiles/proxies/RVGPT.yaml
    url: "你的订阅链接2"
    interval: 86400 
    filter: '2'
    health-check:
      enable: true
      url: http://www.gstatic.com/generate_204
      interval: 300

# 这里根据上方proxy-providers订阅链接提供 进行节点分组！
proxy-groups:  
  - 
    name: 'igziProxy'  # 组名
    # type 可写值：
    #      select（手动选择）
    #      url-test（速度最快，自动选择）
    #      fallback（可以尽量按照用户书写的服务器顺序，在确保服务器可用的情况下，由上至下自动选择服务器，不健康的代理会被跳过）
    #      load-balance（能充分利用多个代理的带宽，不健康的代理会被跳过）
    #      最常用的两个select 和 url-test
    type: select
    url: http://www.gstatic.com/generate_204
    interval: 86400
    proxies:    # 选择当前 proxy-groups 下的其他组
      - '🦄 igzi自动选择'   # 这个节点组使用了 type: url-test 自动选择最优节点
      - '🎃 rv自动选择'     # 这个节点组使用了 type: url-test 自动选择最优节点
    use:        # 使用的是上方 proxy-providers 提供的订阅链接中的所有节点
      - RV
      - XFLTD
  - 
    name: '🦄 igzi自动选择'
    type: url-test   # 这个节点组使用了 type: url-test 自动选择最优节点
    url: http://www.gstatic.com/generate_204
    interval: 86400
    use:
      - XFLTD   # 使用的是上方 proxy-providers 提供的《订阅链接1》中的所有节点
  - 
    name: '🎃 rv自动选择'
    type: url-test   # 这个节点组使用了 type: url-test 自动选择最优节点
    url: http://www.gstatic.com/generate_204
    interval: 86400
    use:
      - RV      # 使用的是上方 proxy-providers 提供的《订阅链接2》中的所有节点

```

> #### 接下来看一下阶段图，先看一下两个节点组'🦄 igzi自动选择' 和 '🎃 rv自动选择' 中的节点，之所以把 igziProxy 放在第一个，纯属个人习惯。

<img width="435" alt="image" src="https://github.com/igziss/ClashxProConfig/assets/29473502/73b81f3a-650f-4327-8b06-87c180280c2f">
<img width="500" alt="image" src="https://github.com/igziss/ClashxProConfig/assets/29473502/c2268c71-06d6-4232-8f6b-ebce5e3fc658">


> ### use: 加载 XFLTD =《你的订阅链接1》和 RV =《你的订阅链接2》的所有节点
> 
> ### 综上所述：主开关 igziProxy 
> > #### 1. 可选择《你的订阅链接1》中的最优路线='🦄 igzi自动选择' 
> > #### 2. 可选择《你的订阅链接2》中的最优路线='🎃 rv自动选择'
> > #### 3. 可选择 use: 《你的订阅链接1》和《你的订阅链接2》两个订阅中任意一个节点
> #### 如下图 igziProxy 代理主开关（放在第一个，方便使用/点击），里面控制了两个自定义节点组，两个节点组自动选择最优节点（使用了 type: url-test）和 use 全部节点（两个订阅的全部节点）
<img width="526" alt="image" src="https://github.com/igziss/ClashxProConfig/assets/29473502/9cfcb7c8-14f2-4583-a130-eeec42c735a7">

### 接下来配置继续

```yaml
  - 
    name: '🤖 chatGPT'
    type: select
    url: http://www.gstatic.com/generate_204
    interval: 86400
    use:  # 这里使用proxy-providers中的两个自定义（剔除chatgpt受限的访问国家）节点
      - RVGPT
      - XFLTDGPT
  - 
    name: '📲 电报消息'
    type: select
    url: http://www.gstatic.com/generate_204
    interval: 86400
    use:
      - RV
      - XFLTD
  -
    name: '🤺 微软服务'
    type: select
    url: http://www.gstatic.com/generate_204
    interval: 86400
    use:
      - RV
      - XFLTD
    proxies:
      - DIRECT
# 自定义黑/白名单。当前节点组选择 DIRECT = 黑名单，反之白名单。
# 如果你为了省流量就使用黑名单，不需要节省流量，防止出现请求超时访问不通等就用白名单。
  -
    name: '🐟 漏网之鱼'
    type: select
    proxies: 
      - DIRECT
      - 'igziProxy'
    use:
      - RV
      - XFLTD
```
> 以下使用的在线规则集，以 url 链接形式提供，包含常见的域名和IP 地址列表，由项目大佬们实时维护的！
> 
> 转自：<a href="https://github.com/Loyalsoldier/clash-rules">@Loyalsoldier/clash-rules</a>

```yaml
rule-providers:
# 广告域名列表 reject.txt
  reject:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/reject.txt"
    path: ./ruleset/reject.yaml
    interval: 86400
# Apple 在中国大陆可直连的域名列表 apple.txt：
  icloud:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/icloud.txt"
    path: ./ruleset/icloud.yaml
    interval: 86400
# iCloud 域名列表 icloud.txt：
  apple:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/apple.txt"
    path: ./ruleset/apple.yaml
    interval: 86400
# [慎用]Google 在中国大陆可直连的域名列表 google.txt
  google:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/google.txt"
    path: ./ruleset/google.yaml
    interval: 86400
# 代理域名列表 proxy.txt
  proxy:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/proxy.txt"
    path: ./ruleset/proxy.yaml
    interval: 86400
# 直连域名列表 direct.txt
  direct:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/direct.txt"
    path: ./ruleset/direct.yaml
    interval: 86400
# 私有网络专用域名列表 private.txt
  private:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/private.txt"
    path: ./ruleset/private.yaml
    interval: 86400
# GFWList 域名列表 gfw.txt
  gfw:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/gfw.txt"
    path: ./ruleset/gfw.yaml
    interval: 86400
# 非中国大陆使用的顶级域名列表 tld-not-cn.txt
  tld-not-cn:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/tld-not-cn.txt"
    path: ./ruleset/tld-not-cn.yaml
    interval: 86400
# Telegram 使用的 IP 地址列表 telegramcidr.txt
  telegramcidr:
    type: http
    behavior: ipcidr
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/telegramcidr.txt"
    path: ./ruleset/telegramcidr.yaml
    interval: 86400
# 中国大陆 IP 地址列表 cncidr.txt
  cncidr:
    type: http
    behavior: ipcidr
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/cncidr.txt"
    path: ./ruleset/cncidr.yaml
    interval: 86400
# 局域网 IP 及保留 IP 地址列表 lancidr.txt
  lancidr:
    type: http
    behavior: ipcidr
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/lancidr.txt"
    path: ./ruleset/lancidr.yaml
    interval: 86400
# 需要直连的常见软件列表 applications.txt
  applications:
    type: http
    behavior: classical
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/applications.txt"
    path: ./ruleset/applications.yaml
    interval: 86400
```
#### 以上是规则集，如果不想使用，只需自定义以下 rules 即可

```yaml
rules:
# START 这里是我自定义的规则
  - DOMAIN-SUFFIX,axure.cloud,igziProxy
# igziProxy 可以是 proxy-groups 节点组中的任意名字，写哪一个就意味着走哪个节点组
  - DOMAIN-SUFFIX,axshare.com,igziProxy
  - DOMAIN,translate.googleapis.com,igziProxy
# END 这里是我自定义的规则

# START 这里是我从别的地方获取的规则
  - DOMAIN-KEYWORD,openai,🤖 chatGPT
  - DOMAIN-SUFFIX,ai.com,🤖 chatGPT
  - DOMAIN-SUFFIX,openai.com,🤖 chatGPT
  - DOMAIN,chat.openai.com.cdn.cloudflare.net,🤖 chatGPT
  - DOMAIN,openaiapi-site.azureedge.net,🤖 chatGPT
  - DOMAIN,openaicom-api-bdcpf8c6d2e9atf6.z01.azurefd.net,🤖 chatGPT
  - DOMAIN,openaicomproductionae4b.blob.core.windows.net,🤖 chatGPT
  - DOMAIN,production-openaicom-storage.azureedge.net,🤖 chatGPT
  - DOMAIN,o33249.ingest.sentry.io,🤖 chatGPT
  - DOMAIN-KEYWORD,1drv,🤺 微软服务
  - DOMAIN-KEYWORD,microsoft,🤺 微软服务
  - DOMAIN-SUFFIX,aadrm.com,🤺 微软服务
  - DOMAIN-SUFFIX,acompli.com,🤺 微软服务
  - DOMAIN-SUFFIX,acompli.net,🤺 微软服务
  - DOMAIN-SUFFIX,aka.ms,🤺 微软服务
  - DOMAIN-SUFFIX,akadns.net,🤺 微软服务
  - DOMAIN-SUFFIX,aspnetcdn.com,🤺 微软服务
  - DOMAIN-SUFFIX,assets-yammer.com,🤺 微软服务
  - DOMAIN-SUFFIX,azure.com,🤺 微软服务
  - DOMAIN-SUFFIX,azure.net,🤺 微软服务
  - DOMAIN-SUFFIX,azureedge.net,🤺 微软服务
  - DOMAIN-SUFFIX,azurerms.com,🤺 微软服务
  - DOMAIN-SUFFIX,bing.com,🤺 微软服务
  - DOMAIN-SUFFIX,cloudapp.net,🤺 微软服务
  - DOMAIN-SUFFIX,cloudappsecurity.com,🤺 微软服务
  - DOMAIN-SUFFIX,edgesuite.net,🤺 微软服务
  - DOMAIN-SUFFIX,gfx.ms,🤺 微软服务
  - DOMAIN-SUFFIX,hotmail.com,🤺 微软服务
  - DOMAIN-SUFFIX,live.com,🤺 微软服务
  - DOMAIN-SUFFIX,live.net,🤺 微软服务
  - DOMAIN-SUFFIX,lync.com,🤺 微软服务
  - DOMAIN-SUFFIX,msappproxy.net,🤺 微软服务
  - DOMAIN-SUFFIX,msauth.net,🤺 微软服务
  - DOMAIN-SUFFIX,msauthimages.net,🤺 微软服务
  - DOMAIN-SUFFIX,msecnd.net,🤺 微软服务
  - DOMAIN-SUFFIX,msedge.net,🤺 微软服务
  - DOMAIN-SUFFIX,msft.net,🤺 微软服务
  - DOMAIN-SUFFIX,msftauth.net,🤺 微软服务
  - DOMAIN-SUFFIX,msftauthimages.net,🤺 微软服务
  - DOMAIN-SUFFIX,msftidentity.com,🤺 微软服务
  - DOMAIN-SUFFIX,msidentity.com,🤺 微软服务
  - DOMAIN-SUFFIX,msn.cn,🤺 微软服务
  - DOMAIN-SUFFIX,msn.com,🤺 微软服务
  - DOMAIN-SUFFIX,msocdn.com,🤺 微软服务
  - DOMAIN-SUFFIX,msocsp.com,🤺 微软服务
  - DOMAIN-SUFFIX,mstea.ms,🤺 微软服务
  - DOMAIN-SUFFIX,o365weve.com,🤺 微软服务
  - DOMAIN-SUFFIX,oaspapps.com,🤺 微软服务
  - DOMAIN-SUFFIX,office.com,🤺 微软服务
  - DOMAIN-SUFFIX,office.net,🤺 微软服务
  - DOMAIN-SUFFIX,office365.com,🤺 微软服务
  - DOMAIN-SUFFIX,officeppe.net,🤺 微软服务
  - DOMAIN-SUFFIX,omniroot.com,🤺 微软服务
  - DOMAIN-SUFFIX,onedrive.com,🤺 微软服务
  - DOMAIN-SUFFIX,onenote.com,🤺 微软服务
  - DOMAIN-SUFFIX,onenote.net,🤺 微软服务
  - DOMAIN-SUFFIX,onestore.ms,🤺 微软服务
  - DOMAIN-SUFFIX,outlook.com,🤺 微软服务
  - DOMAIN-SUFFIX,outlookmobile.com,🤺 微软服务
  - DOMAIN-SUFFIX,phonefactor.net,🤺 微软服务
  - DOMAIN-SUFFIX,public-trust.com,🤺 微软服务
  - DOMAIN-SUFFIX,sfbassets.com,🤺 微软服务
  - DOMAIN-SUFFIX,sfx.ms,🤺 微软服务
  - DOMAIN-SUFFIX,sharepoint.com,🤺 微软服务
  - DOMAIN-SUFFIX,sharepointonline.com,🤺 微软服务
  - DOMAIN-SUFFIX,skype.com,🤺 微软服务
  - DOMAIN-SUFFIX,skypeassets.com,🤺 微软服务
  - DOMAIN-SUFFIX,skypeforbusiness.com,🤺 微软服务
  - DOMAIN-SUFFIX,staffhub.ms,🤺 微软服务
  - DOMAIN-SUFFIX,svc.ms,🤺 微软服务
  - DOMAIN-SUFFIX,sway-cdn.com,🤺 微软服务
  - DOMAIN-SUFFIX,sway-extensions.com,🤺 微软服务
  - DOMAIN-SUFFIX,sway.com,🤺 微软服务
  - DOMAIN-SUFFIX,trafficmanager.net,🤺 微软服务
  - DOMAIN-SUFFIX,uservoice.com,🤺 微软服务
  - DOMAIN-SUFFIX,virtualearth.net,🤺 微软服务
  - DOMAIN-SUFFIX,visualstudio.com,🤺 微软服务
  - DOMAIN-SUFFIX,windows-ppe.net,🤺 微软服务
  - DOMAIN-SUFFIX,windows.com,🤺 微软服务
  - DOMAIN-SUFFIX,windows.net,🤺 微软服务
  - DOMAIN-SUFFIX,windowsazure.com,🤺 微软服务
  - DOMAIN-SUFFIX,windowsupdate.com,🤺 微软服务
  - DOMAIN-SUFFIX,wunderlist.com,🤺 微软服务
  - DOMAIN-SUFFIX,yammer.com,🤺 微软服务
  - DOMAIN-SUFFIX,yammerusercontent.com,🤺 微软服务
# END 这里是我从别的地方获取的规则

# START 这里是配置中 rule-providers 规则集
# RULE-SET：设定规则集，第二个参数（举例：telegramcidr）：rule-providers > telegramcidr，最后就是设定走代理/直连
# 控制访问权限
#   若下方RULE-SET未配置某一条规则（举例：telegramcidr） && 当前配置方式是白单  则telegramcidr自动走代理
#   若下方RULE-SET未配置某一条规则（举例：telegramcidr） && 当前是黑名单方式  则telegramcidr自动走直连 DIRECT

  - RULE-SET,direct,DIRECT
  - RULE-SET,proxy,igziProxy
  - RULE-SET,reject,REJECT  
  - RULE-SET,private,DIRECT
  - RULE-SET,apple,DIRECT  
  - RULE-SET,icloud,igziProxy
  - RULE-SET,gfw,igziProxy
  - RULE-SET,tld-not-cn,igziProxy
  - RULE-SET,telegramcidr,📲 电报消息
  - RULE-SET,lancidr,DIRECT
  - RULE-SET,cncidr,DIRECT
  - RULE-SET,applications,DIRECT
  - DOMAIN,clash.razord.top,DIRECT
  - DOMAIN,yacd.haishan.me,DIRECT
# END 这里是配置中 rule-providers 规则集

# MATCH 🐟 漏网之鱼 可自由切换，切换至DIRECT = 黑名单方式， 切换至其它节点(含自动选择等自定义节点) = 白名单方式
  - MATCH,🐟 漏网之鱼
```

> 1. 以上的code是一个完整的配置（为了方便演示中间有插图截断）。
> 2. 在配置文件目录 新建一个自定义名字的config.yaml，我的是 config-custom.yaml，编辑，一段段复制粘贴。
> 3. 以上挨个按照顺序复制一下（注意缩进，从左第一个空位开始粘贴），贴到自定义配置中，保存，使用即可。
> 4. 可根据自己的需求修改，绝大多数的注释都写好了。

<img width="337" alt="image" src="https://github.com/igziss/ClashxProConfig/assets/29473502/54391775-b62e-4a00-adfa-4bcb256de29d">
<img width="473" alt="image" src="https://github.com/igziss/ClashxProConfig/assets/29473502/f5a33591-ff65-49a6-a261-724fd8dc23eb">



### 最后推荐一个我自用的机场，<a href="https://xftld.org/index.php#/register?code=iSlnaMb0">XFLTD养鸡场</a> 根据自己需求购买，性价比比较高10-20元，速度基本满足日常需求！

