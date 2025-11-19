# search-bot 使用说明（Docker 部署）

  镜像：liqman/search-bot:latest
  功能：通过企业微信自建应用交互，搜索网盘/磁力资源并回传结果。支持多源指令（夸克、阿里、百度、天翼、UC、移动云盘、
  115、PikPak、迅雷、123、磁力、eD2k、Pianer），分页（n/p/c），源切换，以及 getBT/get115 优先并合并 Pansou 结果。支持
  HTTP(S)/SOCKS 代理与功能开关（开/关 getBT、get115、Pianer）。

  ———

  ## 1. 前置条件

  - 已创建企业微信自建应用，具备“发送消息”接口权限，并能配置消息回调 URL。
  - 宿主机可访问：
      - https://qyapi.weixin.qq.com（企业微信 API）
      - Pansou 基础地址，依赖项目见https://github.com/fish2018/pansou
      - （可选）getBT/get115 所需外部站点nullbr.eu.org、www.1lou.pro、www.1lou.pro
  - 准备 config.json，容器运行时通过卷挂载到 /app/config.json。

  ———

  ## 2. 配置文件（config.json）

  将以下内容保存为 config.json（请按需填写/修改）：

  {
    "wecom": {
      "corp_id": "wwxxxxxxxxxxxxxxxx",    # 企业微信的企业ID
      "agent_id": 1000002,    # 企业微信的应用AgentID
      "agent_secret": "YOUR_APP_SECRET",    # 企业微信的应用密钥
      "token": "WEComCallbackToken",    # 企业微信的应用回调token
      "encoding_aes_key": "Your43CharEncodingAESKey",    # 企业微信的应用回调aes key
      "api_proxy_url": "",    # 企业微信的自建回调代理
      "use_env_proxy": true
    },
    "pansou": {
      "base_url": "http://192.168.1.1:8888",    # 部署的pansou，具体部署见https://github.com/fish2018/pansou，建议部署时启用认证，AUTH_TOKEN_EXPIRY参数设置为24000
      "token": "YOUR_PANSOU_TOKEN",    # pansou 的token
      "default_limit": 8,
      "request_timeout": 15
    },
    "features": {
      "enable_getbt": true,    # 是否开启磁力之家的磁力爬取，资源内容详见https://www.1lou.pro/
      "enable_get115": true,    # 是否开启nullbr 115网盘资源搜索，开启该选项要填写app_id和api_key，资源内容详见https://nullbr.eu.org/
      "enable_pianer": false    # 是否开启磁力家的磁力爬取，该资源网站包含9KG，请酌情开启，资源内容详见https://cilijia.net/
    },
	"oneonefive": {
      "app_id": "YOUR_115_APP_ID",    # nullbr的APP_ID，申请见https://nullbr.eu.org/
      "api_key": "YOUR_115_API_KEY"    # nullbr的API_KEY，申请见https://nullbr.eu.org/
    },
    "magnet": {
      "bt_workers": 4    # 根据机器性能变更，默认不动即可
    },
    "proxies": {
      "http": "",    # 代理设置，值形如 http://127.0.0.1:7890
      "https": "",
      "socks": ""
    },
    "app": {
      "host": "0.0.0.0",    # 默认不用变更
      "port": 18001,    # 项目端口，也是容器内部端口
      "log_level": "INFO",
      "default_types": "quark"    # 默认搜索的类型，可填写baidu aliyun quark tianyi uc mobile 115 pikpak xunlei 123 magnet ed2k
    }
  }

  说明：

  - 企业微信回调需使用 wecom.token 与 wecom.encoding_aes_key。
  - features 用于开/关数据源：
      - enable_getbt：/magnet 时启用 getBT（并优先展示）
      - enable_get115：/115 时启用 get115（并优先展示）
      - enable_pianer：启用 Pianer 源
  - proxies：统一代理配置（requests 使用）：
      - 可填 http/https 代理，或 socks（如 socks5h://host:port），未填则不使用。
  - oneonefive：get115-raw API 的 APP 信息（也可通过环境变量 W115_APP_ID/W115_API_KEY 提供）。
  - magnet.bt_workers：getBT 并发抓取线程数。
  - app.port：服务监听端口（与端口映射一致，并用于企业微信回调）。

  ———

  ## 3. Docker Compose 部署

  创建 docker-compose.yml：

  version: "3.8"
  services:
    search-bot:
      image: liqman/search-bot:latest
      container_name: search-bot
      restart: unless-stopped
      volumes:
        - ./config.json:/app/config.json:ro
      ports:
        - "18001:18001"
      environment:
        # 可选：用环境变量覆盖（当 config.json 未设置时）
        # WECOM_CORP_ID: "wwxxxxxxxxxxxxxxxx"
        # WECOM_AGENT_ID: "1000002"
        # WECOM_AGENT_SECRET: "YOUR_APP_SECRET"
        # WECOM_TOKEN: "WEComCallbackToken"
        # WECOM_ENCODING_AES_KEY: "Your43CharEncodingAESKey"
        # WECOM_API_PROXY_URL: ""
        # WECOM_USE_ENV_PROXY: "false"
        # W115_APP_ID: "YOUR_115_APP_ID"
        # W115_API_KEY: "YOUR_115_API_KEY"
        # BT_CONCURRENCY: "6"
        # BT_SEARCH_NUM: "100"

  启动与查看日志：

  - 启动：docker compose up -d
  - 查看日志：docker compose logs -f

  ———

  ## 4. 直接 Docker 命令

  # Linux/Mac
  docker run -d --name search-bot \
    -p 18001:18001 \
    -v "$(pwd)/config.json:/app/config.json:ro" \
    liqman/search-bot:latest

  # Windows PowerShell
  docker run -d --name search-bot `
    -p 18001:18001 `
    -v "D:\path\to\config.json:/app/config.json:ro" `
    liqman/search-bot:latest

  ———

  ## 5. 企业微信回调配置

  - 回调 URL：http://你的主机IP或域名:18001/wecom/callback
  - Token：使用 config.json 的 wecom.token
  - EncodingAESKey：使用 config.json 的 wecom.encoding_aes_key
  - 模式：建议“兼容模式”或“安全模式”（本项目支持解密）。纯明文也可。
  - 确保自建应用“可见范围”包含目标用户，并且具备“发送消息”接口权限。

  ———

  ## 6. 健康检查

  - 服务健康：
      - GET http://主机:18001/health → ok
  - 企业微信 token：
      - GET http://主机:18001/health/wecom → ok token=xxxx...（若失败，检查凭据或网络）
  - 主动消息测试：
      - GET http://主机:18001/health/send?user=你的UserID&text=ping
      - 你的企业微信应收到“ping”，并在日志中看到发送结果。

  ———

  ## 7. 使用方法（企业微信）

  - 帮助：/start
      - 返回使用说明与支持的指令列表
  - 搜索命令（指令 + 关键词）：
      - /baidu 关键词 /aliyun 关键词 /quark 关键词 /tianyi 关键词 /uc 关键词
      - /mobile 关键词 /115 关键词 /pikpak 关键词 /xunlei 关键词 /123 关键词
      - /magnet 关键词 /ed2k 关键词 /pianer 关键词
  - 直接发关键词（无指令）
      - 默认搜索 quark
  - 翻页与控制：
      - n → 下一页
      - p → 上一页
      - c → 取消会话
  - 切换源：
      - 直接发送类型名（如 aliyun）切换源，并复用当前关键词
  - 展示格式（每条两行，首行编号用 emoji，第二行对齐；有密码则第三行）：
      - 1️⃣ 📄 标题
      -    🔗 URL
      -    🔑 密码（如有）
      - 页脚：
          - 📑 第X/Y页 · 共N条
          - 💡 操作：n 下一页 · p 上一页 · c 取消 | 切换源：/baidu /aliyun /quark /tianyi /uc /mobile /115 /pikpak /
            xunlei /123 /magnet /ed2k /pianer
  - 磁力与 115 合并逻辑：
      - /magnet：优先展示 getBT 结果，再拼接 Pansou 的磁力链接；全量严格去重（规范化标题+URL）
      - /115：优先展示 get115 结果，再拼接 Pansou 的 115 链接（含密码）

  ———

  ## 8. 功能开关与代理

  - 功能开关（config.json.features）：
      - enable_getbt：控制 getBT 是否参与 /magnet
      - enable_get115：控制 get115 是否参与 /115
      - enable_pianer：控制 /pianer 是否可用
  - 全局代理（config.json.proxies）：
      - http / https / socks（如 socks5h://127.0.0.1:7891）
      - 自动应用到企业微信 API、Pansou、get115、getBT
  - 企业微信 API 独立代理（wecom.api_proxy_url）
      - 若只想给企业微信 API 配置单独代理，可设置该字段

  ———

  ## 9. 环境变量（可选覆盖）

  当不方便挂载 config.json，或想临时覆盖时可使用：

  - 企业微信：
      - WECOM_CORP_ID、WECOM_AGENT_ID、WECOM_AGENT_SECRET
      - WECOM_TOKEN、WECOM_ENCODING_AES_KEY
      - WECOM_API_PROXY_URL、WECOM_USE_ENV_PROXY（1/true/yes）
  - 115：
      - W115_APP_ID、W115_API_KEY
  - getBT 调优：
      - BT_CONCURRENCY、BT_SEARCH_NUM

  ———

  ## 10. 常见问题

  - 日志提示“检测到加密消息，但解密配置不完整或 wechatpy 未安装”：
      - 确认容器内存在 /app/config.json 且填了 wecom.token/encoding_aes_key/corp_id
      - 或设置上述环境变量
      - 确认镜像内已包含 wechatpy 与 pycryptodome（已内置）
  - 收到“未识别的指令”：
      - 可能误写指令，如 /ali 应使用 /aliyun；系统会提供修正建议
  - 无法发送消息：
      - 检查应用“可见范围”和接口权限
      - 检查容器是否能访问 qyapi.weixin.qq.com
  - Pansou 401 或超时：
      - 检查 pansou.token、pansou.base_url 与 request_timeout

  ———

  ## 11. 更新/重启

  - 更新镜像：docker pull liqman/search-bot:latest && docker compose up -d
  - 重启服务：docker compose restart