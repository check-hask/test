[README.md](https://github.com/user-attachments/files/34068806/README.md)
# 行间 · 笔记 — 安装包

移动端笔记应用。可安装到手机主屏幕，离线运行，无外部依赖。

---

## 一、这个包是什么

一个 **PWA（渐进式 Web 应用）安装包**。装到主屏幕后：

- 全屏运行，没有浏览器地址栏，和原生 App 的观感一致
- 断网可用（应用外壳已缓存到本地）
- 有独立图标、启动画面、主题色
- 支持长按图标弹出的快捷方式：新建笔记 / 搜索

> 为什么不是 .apk / .ipa：npx 打包原生壳需要 Android SDK 或 Xcode 环境，且会引入数百 MB 依赖。当前这版零依赖、双端通用（iOS / Android / 桌面都能装）。若确实需要 .apk，见文末「升级为原生包」。

---

## 二、三种安装方式

### 方式 1｜本地预览（最快，1 分钟）

需要已安装 Python 3。

```bash
cd 行间-移动端App-安装包
python serve.py
```

浏览器打开 `http://localhost:8080`。

> ⚠️ 必须通过 http/https 打开，直接双击 `index.html` 用 `file://` 协议时 Service Worker 不生效（离线能力失效，其余功能正常）。

### 方式 2｜部署到 HTTPS 静态托管（给真机装）

把本目录整个上传到任意静态托管：Vercel、Netlify、Cloudflare Pages、GitHub Pages、腾讯云 COS、对象存储均可。**不需要后端，不需要构建，直接传文件夹。**

拿到形如 `https://your-domain.com/` 的地址后：

- **Android / Chrome**：打开页面 → 点右上角菜单「安装应用」→ 桌面出现图标
- **iOS / Safari**：打开页面 → 底部「分享」→「添加到主屏幕」
- **桌面 Chrome / Edge**：地址栏右侧出现安装图标，点一下即可

> 也可在 App 内「我的」页点击「安装到主屏幕」触发。

### 方式 3｜局域网内测（同一 WiFi 下的真机）

```bash
python serve.py 8080
```

查本机内网 IP（Windows 用 `ipconfig`），手机浏览器访问 `http://192.168.x.x:8080`。

> 注意：非 localhost 的 http 地址，Chrome 会判定为不安全上下文，Service Worker 与安装入口会被禁用。功能可正常体验，但装不了。要装请用方式 2 的 HTTPS。

---

## 三、目录结构

```
行间-移动端App-安装包/
├── index.html               # 应用本体（单文件，含全部样式与逻辑）
├── manifest.webmanifest     # 应用清单：名称、图标、主题色、快捷方式
├── sw.js                    # Service Worker：外壳离线缓存
├── serve.py                 # 本地预览服务器
├── README.md
└── icons/
    ├── icon-192.png         # 主屏图标
    ├── icon-512.png         # 启动画面 / 应用商店
    ├── icon-maskable-512.png# Android 自适应图标（安全区已留足）
    └── apple-touch-icon.png # iOS 主屏图标
```

---

## 四、包含哪些界面

| 界面 | 说明 |
|---|---|
| 登录 | 手机号 + 6 位验证码，60 秒倒计时，第三方登录占位 |
| 首页 | 置顶区、筛选标签、最近编辑列表、新建入口 |
| 笔记详情 | 可编辑正文，实时字数统计，待办项可勾选 |
| 新建笔记 | 独立撰写流，保存后回到列表 |
| 笔记本 | 概览数据、精选封面、封面网格 |
| 笔记本详情 | 该本下的笔记列表 |
| 搜索 | 输入即筛选，AI 摘要卡 + 引用溯源 + 相关度 |
| 我的 | 账户、统计、同步/隐私锁/导出、外观切换 |

**交互**：页面推入/返回转场、Tab 切换、深色模式、Toast 反馈、路由返回栈（Esc 可返回）。
**深色模式**：设置 → 外观 → 浅色 / 深色 / 跟随系统，会同步更新浏览器主题色。

---

## 五、已验证项

- [x] 所有资源 200，MIME 类型正确（`application/manifest+json`、`text/javascript`）
- [x] 清单可解析，三档图标齐全（含 maskable）
- [x] Service Worker 注册与离线外壳缓存
- [x] 独立运行模式（standalone）下安全区适配、无横向溢出
- [x] 深色 / 浅色双主题
- [x] 桌面与移动视口下布局一致

---

## 六、升级为原生包（.apk / .ipa）

需要原生壳时，用 Capacitor 包一层，无需改代码：

```bash
npm i -g @capacitor/cli
cd 行间-移动端App-安装包
# 先把本目录内容放进一个 web 目录
mkdir -p www && cp -r index.html manifest.webmanifest sw.js icons www/
npx cap init "行间" app.xingjian.note --web-dir=www
npx cap add android
npx cap sync
cd android && ./gradlew assembleDebug
```

产物在 `android/app/build/outputs/apk/debug/app-debug.apk`。
需要本机装好 JDK 17 + Android SDK。iOS 同理需 macOS + Xcode。

---

版本 1.0.0 · 2026-10
