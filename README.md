# search-bot 使用说明（Docker 部署与使用）

  镜像：liqman/search-bot:latest

  通过企业微信（WeCom）自建应用进行命令式搜索，整合网盘与磁力源结果并返回至聊天窗口。支持多源指令、分页（n/p/c）、源切
  换、结果合并与去重、HTTP(S)/SOCKS 代理及数据源开关。

  ———

  ## 功能概览

  - 企业微信交互：支持“/指令 关键词”，纯关键词默认搜索 quark。
  - 数据源与指令：/baidu、/aliyun、/quark、/tianyi、/uc、/mobile、/115、/pikpak、/xunlei、/123、/magnet、/ed2k、/
    pianer。
  - 分页控制：n（下一页）、p（上一页）、c（取消）。
  - 结果合并：
      - /magnet：优先展示 getBT 结果，再拼接 Pansou 的磁力链接；统一去重后展示。
      - /115：优先展示 get115 结果，再拼接 Pansou 的 115 链接；统一去重后展示。
  - 输出格式：每条两行（名称与链接分行，首字对齐），如有密码第三行展示。
  - 去重：严格“标题（归一化）+ URL”维度去重（全角半角统一、去标点、压缩空白、小写）。
  - 代理支持：全局 http/https/socks 代理，以及 WeCom API 独立代理。
  - 功能开关：可分别开/关 getBT、get115、pianer 源。

  ———

  ## 前置条件

  - 已创建企业微信“自建应用”，具备“发送消息”接口权限，且可配置消息回调 URL。
  - 宿主环境可访问：
      - https://qyapi.weixin.qq.com（企业微信 API）
      - Pansou 基础地址
      - （可选）getBT/get115 所需外部站点  nullbr.eu.org、cilijia.net、www.1lou.pro
  - 准备 config.json，容器运行时挂载到 /app/config.json。

  - pansou docker命令：
      docker run -d --name pansou -p 18880:80 -e AUTH_ENABLED=true -e AUTH_USERS=<管理员用户名>:<管理员密码> -e AUTH_TOKEN_EXPIRY=24000 -e ENABLED_PLUGINS=labi,zhizhen,shandian,duoduo,muou,wanou,hunhepan,jikepan,panwiki,pansearch,panta,qupansou,hdr4k,pan666,susu,thepiratebay,xuexizhinan,panyq,ouge,huban,cyg,erxiao,miaoso,fox4k,pianku,clmao,wuji,cldi,xiaozhang,libvio,leijing,xb6v,xys,ddys,hdmoli,yuhuage,u3c3,javdb,clxiong,jutoushe,sdso,xiaoji,xdyh,haisou,bixin,djgou,nyaa,xinjuc,aikanzy,qupanshe,xdpan,discourse,yunsou,ahhhhfs,nsgame,quark4k,quarksoo,sousou,ash,feikuai -e CHANNELS=tgsearchers4,Aliyun_4K_Movies,bdbdndn11,yunpanx,bsbdbfjfjff,yp123pan,sbsbsnsqq,yunpanxunlei,tianyifc,BaiduCloudDisk,txtyzy,peccxinpd,gotopan,PanjClub,kkxlzy,baicaoZY,MCPH01,MCPH02,MCPH03,bdwpzhpd,ysxb48,jdjdn1111,yggpan,MCPH086,zaihuayun,Q66Share,ucwpzy,shareAliyun,alyp_1,dianyingshare,Quark_Movies,XiangxiuNBB,ydypzyfx,ucquark,xx123pan,yingshifenxiang123,zyfb123,tyypzhpd,tianyirigeng,cloudtianyi,hdhhd21,Lsp115,oneonefivewpfx,qixingzhenren,taoxgzy,Channel_Shares_115,tyysypzypd,vip115hot,wp123zy,yunpan139,yunpan189,yunpanuc,yydf_hzl,leoziyuan,Q_dongman,yoyokuakeduanju,TG654TG,WFYSFX02,QukanMovie,yeqingjie_GJG666,movielover8888_film3,Baidu_netdisk,D_wusun,FLMdongtianfudi,KaiPanshare,QQZYDAPP,rjyxfx,PikPak_Share_Channel,btzhi,newproductsourcing,cctv1211,duan_ju,QuarkFree,yunpanNB,kkdj001,xxzlzn,pxyunpanxunlei,jxwpzy,kuakedongman,liangxingzhinan,xiangnikanj,solidsexydoll,guoman4K,zdqxm,kduanju,cilidianying,CBduanju,SharePanFilms,dzsgx,BooksRealm,Oscar_4Kmovies,douerpan,baidu_yppan,Q_jilupian,Netdisk_Movies,yunpanquark,ammmziyuan,ciliziyuanku,cili8888,jzmm_123pan,Q_dianying,domgmingapk,dianying4k,q_dianshiju,tgbokee,ucshare,godupan,gokuapan ghcr.io/fish2018/pansou-web

  ———

  ## 配置文件（config.json）

  详见 config.json 并按需填写

  说明要点:

  - 企业微信回调需使用 wecom.token 与 wecom.encoding_aes_key，并与后台配置一致。
  - features 控制是否启用相应源：
      - enable_getbt：/magnet 是否启用 getBT（优先展示）。
      - enable_get115：/115 是否启用 get115（优先展示）。
      - enable_pianer：是否启用 Pianer 源。
  - proxy_url：全局代理配置，若填写 socks 且未设置 http/https，会回退使用 socks。
  - oneonefive：get115-raw API 的 APP 信息（也可通过环境变量 W115_APP_ID/W115_API_KEY 提供）。
  - magnet.bt_workers：getBT 并发抓取线程数。
  - app.port：服务监听端口，需与端口映射一致，并用于企业微信回调。

  ———

  ## Docker Compose 部署

  详见 docker-compose.yml

  启动与日志：

  - 启动：docker compose up -d
  - 查看日志：docker compose logs -f

  ———

  ## Docker 直接运行
  docker run -d --name search-bot \  
      -p 18001:18001 \  
      -v "$(pwd)/config.json:/app/config.json:ro" \  
    liqman/search-bot:latest

  ———

  ## 企业微信回调配置

  - 回调 URL：http://你的主机或域名:18001/wecom/callback
  - Token：使用 config.json 的 wecom.token
  - EncodingAESKey：使用 config.json 的 wecom.encoding_aes_key
  - 模式：推荐“兼容模式”或“安全模式”（本项目支持解密），纯明文也可。
  - 应用“可见范围”需包含目标用户，且应用需具备“发送消息”接口权限。

  ———

  ## 使用说明（企业微信）

  - 帮助：/start（返回用法与指令说明）
  - 搜索命令（指令 + 关键词）：
      - /baidu 关键词 /aliyun 关键词 /quark 关键词 /tianyi 关键词 /uc 关键词
      - /mobile 关键词 /115 关键词 /pikpak 关键词 /xunlei 关键词 /123 关键词
      - /magnet 关键词 /ed2k 关键词 /pianer 关键词
  - 直接发关键词（无指令）：默认搜索 quark。
  - 分页与控制：
      - n → 下一页
      - p → 上一页
      - c → 取消会话
  - 切换源：直接发送类型名（如 aliyun）切换源，并复用当前关键词。
  - 展示格式（每条两行，首字对齐，有密码则第三行）：
      - 1️⃣ 📄 标题
      -    🔗 URL
      -    🔑 密码（如有）
  - 页脚：
      - 📑 第X/Y页 · 共N条
      - 💡 操作：n 下一页 · p 上一页 · c 取消 | 切换源：/baidu /aliyun /quark /tianyi /uc /mobile /115 /pikpak /
        xunlei /123 /magnet /ed2k /pianer
  - 合并逻辑：
      - /magnet：先 getBT，再 Pansou 磁力，严格去重后展示。
      - /115：先 get115，再 Pansou 115，严格去重后展示。

  ———

  ## 功能开关与代理

  - 功能开关（config.json.features）：
      - enable_getbt：控制 getBT 是否参与 /magnet
      - enable_get115：控制 get115 是否参与 /115
      - enable_pianer：控制 /pianer 是否可用
  - 全局代理（config.json.proxies）：
      - http / https / socks（如 socks5h://127.0.0.1:7891）
      - 自动应用于 WeCom API、Pansou、get115、getBT 等请求
  - WeCom API 独立代理（wecom.api_proxy_url）：
      - 若仅希望企业微信 API 使用独立代理，可设置该字段（优先级高于全局代理）

  ———

  ## 更新与重启

  - 拉取最新镜像并重启（Compose）：
      - docker compose pull && docker compose up -d
  - 重启服务：
      - docker compose restart

  ———

  ## 免责声明

  本项目仅用于学习与研究用途，请勿将其用于任何违反法律法规或服务条款的用途。使用者需自行承担相应责任。
