> 本文基于《打包提测-iOS》新增一些内容，兼容《打包提测-iOS》

## 一、Fastfile 文件新增打包发版的 lane

```ruby
#------ Fastfile ------#
# 不同项目需要修改【项目基础配置】的以下内容：
# 1、SCHEME_NAME：通常就是 Xxx.xcworkspace 和 Xxx.xcodeproj 的名字
# 2、APP_IDENTIFIER：App 的 Bundle ID
# 3、PGYER_API_KEY：登录蒲公英 > 账号 > API 信息 > API Key
# 4、WECOM_WEBHOOK：企业微信群机器人 Webhook 地址
# 5、TESTER_MOBILES：需要 @ 的人员手机号
# 6、APP_STORE_KEY_ID：使用有管理员权限的开发者账号登录 App Store Connect > 用户和访问
# > 集成 > App Store Connect API > 团队密钥 > 密钥 ID
# 7、APP_STORE_ISSUER_ID：... > 团队密钥 > Issuer ID
# 8、APP_STORE_KEY_PATH：... > 团队密钥 > 创建 API 密钥后会有下载按钮，下载下来是一个类
# 似 AuthKey_ABC1234567.p8 的文件，我们可以把它放进 fastlane 目录里

require "fileutils"

default_platform(:ios)

platform :ios do
  # ==============================
  # 1、项目基础配置
  # ==============================

  # 项目根目录
  PROJECT_ROOT = File.expand_path("..", __dir__)
  
  # SCHEME_NAME
  SCHEME_NAME = "NearStream"

  # 工作空间路径
  WORKSPACE_PATH = File.join(
    PROJECT_ROOT,
    "#{SCHEME_NAME}.xcworkspace"
  )
  
  # Xcode 工程名称
  XCODEPROJ_PATH = File.join(
    PROJECT_ROOT,
    "#{SCHEME_NAME}.xcodeproj"
  )

  # App 的 Bundle ID
  APP_IDENTIFIER = "com.auditoryworks.nearstreams"

  # 打包产物输出目录
  OUTPUT_DIR = File.join(
    PROJECT_ROOT,
    "build",
    "ios"
  )

  # 打包产物名称
  IPA_NAME = "#{SCHEME_NAME}.ipa"

  # 打包产物输出路径
  IPA_PATH = File.join(
    OUTPUT_DIR,
    IPA_NAME
  )

  # PGYER_API_KEY
  PGYER_API_KEY = "99d5f2bd5fca6f06478425d427613e74"

  # 企微群聊消息推送的 Webhook 地址
  WECOM_WEBHOOK = "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=01c7f32c-525e-4540-96e4-6c8dda8ac4bb"

  # 需要 @ 的人员手机号
  TESTER_MOBILES = [
    "手机号 1",
    "手机号 2"
  ]
  
  # 发版到 App Store 相关 
  APP_STORE_KEY_ID = "你的密钥 ID"
  APP_STORE_ISSUER_ID = "你的 Issuer ID"
  APP_STORE_KEY_PATH =
    File.join(
      PROJECT_ROOT,
      "fastlane",
      "AuthKey_xxxxx.p8"
    )
    
  # App Store Connect API 认证
  def app_store_api_key
    app_store_connect_api_key(
      key_id: APP_STORE_KEY_ID,
      issuer_id: APP_STORE_ISSUER_ID,
      key_filepath: APP_STORE_KEY_PATH
    )
  end
  
  # 读取 App Store Connect 当前启用的语言 
  def app_store_locales
    app = Spaceship::ConnectAPI::App.find(APP_IDENTIFIER)

    unless app
      UI.user_error!(
        "App Store Connect 找不到应用：#{APP_IDENTIFIER}"
      )
    end

    app_info = (
      app.fetch_edit_app_info ||
      app.fetch_live_app_info
    )

    unless app_info
      UI.user_error!(
        "无法读取 App Store Connect 的应用语言配置"
      )
    end

    locales = app_info
      .get_app_info_localizations
      .map(&:locale)
      .uniq

    if locales.empty?
      UI.user_error!(
        "App Store Connect 没有读取到任何语言"
      )
    end

    locales
  end
  
  # 生成 App Store Connect 版本元数据
  def prepare_app_store_metadata(options, locales)
    chinese_release_note = options[:chinese_release_note]
      .to_s
      .gsub("\\n", "\n")
      .strip

    english_release_note = options[:english_release_note]
      .to_s
      .gsub("\\n", "\n")
      .strip

    if chinese_release_note.empty?
      UI.user_error!(
        "缺少中文 Release Note，请传入 chinese_release_note"
      )
    end

    if english_release_note.empty?
      UI.user_error!(
        "缺少英文 Release Note，请传入 english_release_note"
      )
    end

    metadata_path = File.join(
      OUTPUT_DIR,
      "app_store_metadata"
    )

    FileUtils.rm_rf(metadata_path)

    locales.each do |locale|
      release_note = if locale == "zh-Hans"
        chinese_release_note
      else
        english_release_note
      end

      locale_path = File.join(
        metadata_path,
        locale
      )

      FileUtils.mkdir_p(locale_path)

      File.write(
        File.join(locale_path, "release_notes.txt"),
        release_note
      )
    end

    metadata_path
  end
   

  # ==============================
  # 2、打包 IPA
  # ==============================

  # 描述一下 fastlane 正在做什么
  desc "Build iOS IPA"
  # 将来执行 fastlane build 命令即可开始“打包 IPA”，build 可以自定义成其它字符串
  lane :build do

    # 终端里会打印这条信息
    UI.message("开始构建 iOS 安装包")

    build_app(
      workspace: WORKSPACE_PATH,
      scheme: SCHEME_NAME,
      output_directory: OUTPUT_DIR,
      output_name: IPA_NAME,

      # 打包模式：Release、Debug、Profile
      configuration: "Release",

      # 打什么包：ad-hoc、app-store
      export_method: "ad-hoc",

      # 每次打包之前 clean 一下以前的编译信息
      clean: true,

      # 是否包含bitcode
      include_bitcode: false,

      # 是否包含symbols
      include_symbols: true,

      # 这个设置是为了设置Xcode自动配置证书和配置文件，当然也可以手动配置
      export_xcargs: "-allowProvisioningUpdates"
    )

    UI.success("iOS 打包成功")

    UI.message("IPA 路径：#{IPA_PATH}")

  end
  
  # 描述一下 fastlane 正在做什么
  desc "Build App Store IPA"
  lane :build_store do
    UI.message("开始构建 App Store IPA")
    build_app(
      workspace: WORKSPACE_PATH,
      scheme: SCHEME_NAME,
      output_directory: OUTPUT_DIR,
      output_name: IPA_NAME,
      configuration: "Release",
      export_method: "app-store",
      clean: true,
      include_bitcode: false,
      include_symbols: true,
      export_xcargs: "-allowProvisioningUpdates"
    )
    UI.success("App Store IPA 打包成功")
  end


  # ==============================
  # 3、上传 IPA
  # ==============================

  # 描述一下 fastlane 正在做什么
  desc "Upload existing IPA to Pgyer"
  # 将来执行 fastlane upload 命令即可开始“上传 IPA”，upload 可以自定义成其它字符串
  # options 是一个 [String: Any] 类型的 HashMap，我们在终端执行 fastlane upload 可以传递参数进来
  # 例如：fastlane upload releaseNote:"1、修复已知Bug；\n2、提升部分性能。"
  lane :upload do |options|

    # 检查 IPA 是否存在
    unless File.exist?(IPA_PATH)
      UI.user_error!(
        "IPA 不存在：#{IPA_PATH}"
      )
    end

    # 终端里会打印这条信息
    UI.message("开始上传蒲公英")

    # 获取更新说明
    release_note = (
      options[:releaseNote] || "iOS 测试版本"
    ).gsub('\n', "  \n")

    result = pgyer(
      api_key: APP_STORE_KEY_ID,
      ipa: IPA_PATH,
      update_description: release_note
    )

    UI.success("蒲公英上传成功")

    # 检查上传结果
    unless result.is_a?(Hash)
      UI.user_error!("蒲公英未返回有效的上传结果")
    end

    # 将完整上传结果返回给调用方
    # beta 可以获取下载地址、版本号、Build 号
    result

  end
  
  # 描述一下 fastlane 正在做什么
  desc "Upload IPA to App Store Connect"
  lane :upload_store do |options|
    UI.message("开始上传 App Store Connect")

    metadata_path = nil

    begin
      api_key = app_store_api_key
      locales = app_store_locales
      metadata_path = prepare_app_store_metadata(
        options,
        locales
      )

      app_version = get_version_number(
        xcodeproj: XCODEPROJ_PATH
      )

      deliver(
        ipa: IPA_PATH,
        api_key: api_key,
        app_identifier: APP_IDENTIFIER,
        app_version: app_version,
        metadata_path: metadata_path,
        skip_screenshots: true,
        submit_for_review: false,
        automatic_release: false,
        force: true
      )

      UI.success(
        "App Store Connect 上传和版本信息同步成功"
      )
    ensure
      FileUtils.rm_rf(metadata_path) if metadata_path
    end
  end


  # ==============================
  # 4、企微群聊消息推送
  # ==============================

  # 描述一下 fastlane 正在做什么
  desc "Send release notification to WeCom"

  # 将来执行 fastlane notify 命令即可发起企微群聊消息推送
  # 例如：fastlane notify url:"https://www.pgyer.com/xxxx"
  #
  # 支持传入以下参数：
  # platform：发布平台（蒲公英 / App Store）
  # url：下载地址
  # qrcode_url：二维码地址
  # version：App 版本号
  # buildNumber：App Build 号
  # releaseNote：更新内容

  lane :notify do |options|

    # 引入 Ruby 标准库
    require "net/http"
    require "json"
    require "uri"

    # 获取发布平台
    publish_platform = options[:platform] || "蒲公英"

    # 获取二维码地址
    qrcode_url = options[:qrcodeUrl]
    UI.message("二维码地址：#{qrcode_url}")

    # 判断二维码地址是否存在
    qrcode_markdown = if qrcode_url && !qrcode_url.empty?
        "![蒲公英安装二维码](#{qrcode_url})"
    else
        ""
    end

    # 获取版本号
    version = options[:version] || "未知"

    # 获取更新说明
    release_note = (
      options[:releaseNote] || "iOS 测试版本"
    ).gsub('\n', "  \n")

    # 获取发布时间
    publish_time = Time.now.strftime(
      "%Y-%m-%d %H:%M:%S"
    )

    # ------------------------------
    # 构造企业微信 Markdown 消息
    # ------------------------------

    content = <<~MARKDOWN
      ## 🚀【iOS 端】提测

      > 应用名称：#{SCHEME_NAME}
      > 版本号：#{version}
      > 发布时间：#{publish_time}
      > 发布平台：#{publish_platform}

      #### 📝 更新内容

      #{release_note}

      #### 📲 扫码安装

      #{qrcode_markdown}
    MARKDOWN


    # ------------------------------
    # 发送 HTTP 请求
    # ------------------------------

    # 将 Webhook 字符串转为 URI 对象
    uri = URI.parse(WECOM_WEBHOOK)

    # 创建 POST 请求
    request = Net::HTTP::Post.new(uri)

    # 设置请求类型
    request["Content-Type"] = "application/json"

    # 构造请求 Body
    request.body = {
      msgtype: "markdown_v2",
      markdown_v2: {
        content: content
      }
    }.to_json

    # 输出日志
    UI.message("开始发送企业微信群通知")

    # 发送请求
    response = Net::HTTP.start(
      uri.hostname,
      uri.port,
      use_ssl: true,
      open_timeout: 10,
      read_timeout: 15
    ) do |http|

      http.request(request)

    end


    # ------------------------------
    # 检查发送结果
    # ------------------------------

    # 检查 HTTP 状态码
    unless response.is_a?(Net::HTTPSuccess)
      UI.user_error!(
        "企业微信通知失败：HTTP #{response.code}"
      )
    end

    # 解析企业微信返回的数据
    result = JSON.parse(response.body)

    # 判断企业微信是否接收成功
    if result["errcode"] == 0

      UI.success("企业微信群通知发送成功")

    else

      UI.user_error!(
        "企业微信群通知失败：#{result['errmsg']}"
      )

    end


    # ------------------------------
    # @ 指定人员
    # ------------------------------

    # 创建 POST 请求
    mention_request = Net::HTTP::Post.new(uri)

    # 设置请求类型
    mention_request["Content-Type"] = "application/json"

    # 构造请求 Body
    mention_request.body = {
      msgtype: "text",
      text: {
        content: "🚀 【iOS 端】已提测，请及时安装测试。",
        mentioned_mobile_list: TESTER_MOBILES
      }
    }.to_json

    # 输出日志
    UI.message("开始 @ 指定人员")

    # 发送请求
    mention_response = Net::HTTP.start(
      uri.hostname,
      uri.port,
      use_ssl: true,
      open_timeout: 10,
      read_timeout: 15
    ) do |http|

      http.request(mention_request)

    end


    # ------------------------------
    # 检查 @ 通知发送结果
    # ------------------------------

    # 检查 HTTP 状态码
    unless mention_response.is_a?(Net::HTTPSuccess)
      UI.user_error!(
        "企业微信 @ 通知失败：HTTP #{mention_response.code}"
      )
    end

    # 解析企业微信返回的数据
    mention_result = JSON.parse(mention_response.body)

    # 判断企业微信是否接收成功
    if mention_result["errcode"] == 0

      UI.success("企业微信 @ 通知发送成功")

    else

      UI.user_error!(
        "企业微信 @ 通知失败：#{mention_result['errmsg']}"
      )

    end

  end


  # ==============================
  # 5、一键打包 + 上传蒲公英 + 通知企微
  # ==============================

  # 描述一下 fastlane 正在做什么
  desc "Build and upload iOS to Pgyer"

  # 将来执行 fastlane beta 命令即可开始“一键打包并上传”，beta 可以自定义成其它字符串
  lane :beta do |options|

    # ------------------------------
    # 第一步：构建 IPA
    # ------------------------------
    build


    # ------------------------------
    # 第二步：上传蒲公英并获取发布信息
    # ------------------------------

    # 接收 upload 返回的完整上传结果
    result = upload(
      ipa: IPA_PATH,
      releaseNote: options[:releaseNote]
    )


    # ------------------------------
    # 第三步：发送企业微信通知
    # ------------------------------

    # 获取二维码图片地址
    qrcode_url = result["buildQRCodeURL"]

    # 获取 App 版本号
    version = result["buildVersion"] || "未知"

    notify(
      platform: "蒲公英",
      qrcodeUrl: qrcode_url,
      version: version,
      releaseNote: options[:releaseNote]
    )


    # ------------------------------
    # 第四步：完成
    # ------------------------------

    UI.success("iOS 自动打包发布完成")

  end
  
  # ==============================
  # 6、一键打包 + 上传 App Store Connect
  # ==============================
  desc "Build and upload App Store Connect"
  lane :release do |options|
    build_store

    upload_store(
      chinese_release_note: options[:chinese_release_note],
      english_release_note: options[:english_release_note]
    )

    UI.success(
      "App Store Connect 发布流程完成，未提交审核"
    )
  end

end
```

## 二、Codex 里发起一键打包 + 上传 App Store Connect

我们编写了一个 app-store-ios Skill，以便直接在 Codex 里通过显示调用或意图识别的方式触发这个 Skill 来发起一键打包 + 上传 App Store Connect，而不用每次都去终端里敲命令。`为了确保提审版本信息没有错误，我们没有设计自动提交审核，还是由人工发起提交审核。`

![](img/708925a3a52812b433586370a4dcf8be01e05397.png)

![](img/99357b2b5591c2f8c12ebbb09a6df801d2833509.png)
