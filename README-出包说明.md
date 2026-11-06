# 冠华MES iOS 工程（出包说明）

原生壳工程（Capacitor），加载车间 MES（`http://10.10.10.10:8000/m`），功能对齐安卓版：扫码报工、聊天、私聊、报工记录、语音入口。
Windows 侧已生成并校验 22 项结构项，**未做编译验证**（本机无 macOS）。

出包有两条路，**没有 Mac 就走路线 B**。

---

# 路线 B：GitHub 云编译（不需要 Mac）

仓库里已经放好流水线：`.github/workflows/ios-build.yml`。它会在苹果的机器上自动编译出 `.ipa`，你在 Windows 下载后自签安装。

## B1. 传到 GitHub

1. 注册 / 登录 https://github.com
2. 右上角 `+` → `New repository` → 名字填 `mes-ios` → 选 `Private` → 不勾任何初始化文件 → `Create repository`
3. 进入仓库页 → `Add file` → `Upload files` → 把解压出来的**全部内容**拖进去（`ios` 文件夹、`capacitor.config.json`、`package.json`、`.github`、`.gitignore` 都要）→ 底部 `Commit changes`
   - 若 `.github` 文件夹没上传成功（网页偶尔忽略点开头的目录）：`Add file` → `Create new file`，文件名输入 `.github/workflows/ios-build.yml`（斜杠会自动建目录），把工程里这个文件的全文粘贴进去，提交。
4. 仓库 → `Actions` 标签 → 左侧列表点 `Build iOS IPA (unsigned)` → 右侧 `Run workflow` → `Run workflow`
5. 等 10～20 分钟（第一次要拉苹果依赖，慢一点）。变成绿勾后 → 点进这次运行 → 页面**底部** `Artifacts` → 下载 `MES-iOS-unsigned-ipa`（是个 zip）→ 解压得到 `MES-iOS-unsigned.ipa`

> 失败排查：点进失败的步骤看日志。最常见是 `Setup Xcode` 版本问题，把 `xcode-version: latest-stable` 改成具体版本（如 `16.1`）再跑；或 `npm install` 网络问题，重跑一次即可。

## B2. 装到自己手机（Windows 自签，免费）

1. Windows 装 **iTunes**（Microsoft Store 搜 `iTunes` 安装，不要装苹果官网旧版）
2. 下载 **Sideloadly**：https://sideloadly.io → Windows 版，装好
3. iPhone 数据线连电脑，手机上点「信任此电脑」
4. 打开 Sideloadly：`IPA 文件` 拖入 `MES-iOS-unsigned.ipa`；`Apple ID` 填你自己的 Apple ID 邮箱（**免费账号就行，不用买开发者账号**）；`iDevice` 选你的 iPhone → 点 `Start`
   - 中途会弹 Apple ID 密码，输入即可（Sideloadly 是本地签名，密码只走苹果服务器）
5. 装完手机上：设置 → 通用 → VPN与设备管理 → 开发者 App → 点你的 Apple ID → `信任`
6. 桌面出现「冠华MES」图标，点开即用

**免费账号的限制**：签名有效期 **7 天**，到期 App 打不开，重连电脑在 Sideloadly 里再点一次 `Start` 即可（10 秒）。一台设备 7 天内最多签 3 个 App。
**要给几十个工人装**：免费签名不现实（每人每 7 天重签），需要 Apple Developer Program（99 美元/年）+ 走 `Ad Hoc`（最多 100 台设备，收集 UDID）或 `Enterprise`（299 美元/年）。这一步只能在苹果侧办，办完把证书配进流水线就能出「直接能装」的包。

---

# 路线 A：有 Mac（最快，10 分钟）

1. App Store 装 Xcode（15 以上），首次打开同意协议
2. 双击 `ios/App/App.xcodeproj`
3. `Signing & Capabilities`：`Team` 选自己的 Apple ID（Add an Account 登录即可）；`Bundle Identifier` 改成 `com.guanhua.mes` 或加后缀避免冲突；勾 `Automatically manage signing`
4. iPhone 连 Mac → 顶部选设备 → `⌘R`
5. 首次信任：设置 → 通用 → VPN与设备管理 → 信任
6. 正式包（多人分发）：需开发者账号 → `Product` → `Archive` → `Distribute App`

---

# 已经配好的（不用再动）

| 项 | 位置 | 值 |
|---|---|---|
| 服务器地址 | `capacitor.config.json` → `server.url` | `http://10.10.10.10:8000/m` |
| 允许 http 明文 | `ios/App/App/Info.plist` → `NSAppTransportSecurity` | 已开 |
| 相机权限 | `NSCameraUsageDescription` | 扫码报工需要使用相机拍摄工单二维码 |
| 麦克风权限 | `NSMicrophoneUsageDescription` | 与同事进行语音通话需要使用麦克风 |
| 相册权限 | `NSPhotoLibraryUsageDescription` | 发送聊天图片、选择工单照片需要访问相册 |
| 应用名 | `CFBundleDisplayName` | 冠华MES |
| 版本 | `MARKETING_VERSION` / `CURRENT_PROJECT_VERSION` | 4.2 / 15（与安卓同号） |
| 图标 | `Assets.xcassets/AppIcon.appiconset/icon-1024.png` | 1024 不透明蓝底 |
| 编译 scheme | `ios/App/App.xcodeproj/xcshareddata/xcschemes/App.xcscheme` | 已生成（云端构建必需） |
| 依赖 | `CapApp-SPM/Package.swift` | SPM，不需要 CocoaPods |

# 改服务器地址

改 `capacitor.config.json` 的 `server.url`（根目录和 `ios/App/App/` 两份保持一致），推回 GitHub 会自动重跑流水线；本机也可以 `npm install && npx cap sync ios`。

# 这一版做不到的

| 功能 | 原因 | 需要补的条件 |
|---|---|---|
| 语音电话 | WKWebView 的 `getUserMedia` 要求 https | 服务器上 https（cpolar 域名或自建证书） |
| 锁屏消息通知 | 未接 APNs | 开发者账号 + APNs 证书 + 服务端推送代码 |
| 全屏来电界面 | 未接 PushKit + CallKit | 同上 + VoIP 证书 |

要补这三样，代码继续写；证书和上架只能在苹果侧完成。
