# 星蝉im（学习通 IM 助手）

围绕环信（Easemob）IM WebSocket 通道搭建的**本地网页化工具**：登录学习通账号后自动接收 IM 消息并保存到本地，收到签到类消息时按配置自动处理签到，全程数据只保存在你自己的电脑上。

![Windows](https://img.shields.io/badge/平台-Windows-blue)
![FastAPI](https://img.shields.io/badge/后端-FastAPI-009688)
![Vue](https://img.shields.io/badge/前端-Vue3-4FC08D)
![Nuitka](https://img.shields.io/badge/打包-Nuitka%20onefile-2e7d32)
![语言](https://img.shields.io/badge/语言-Python%2FTypeScript-lightgrey)

---

## 功能特性

- **IM 消息实时监听**：连接学习通 IM 消息通道（protobuf WebSocket），支持文本、图片、语音、视频、文件、位置、命令、自定义、合并转发、扩展等消息类型。
- **消息本地持久化**：消息实时入库（本地 SQLite），网页不常开也不会丢消息，随时回看历史记录。
- **多账号隔离**：同一台设备可添加多个账号，每个账号拥有独立的 IM 连接与数据分区。
- **自动签到**：收到签到类消息后自动判断类型并处理，支持普通位置签到、拍照签到、人脸签到、二维码签到（含带位置二维码），并实时展示处理结果。
- **账号会话自动获取**：填写手机号+密码即可自动完成登录并获取后续签到所需的会话参数（uid / TOKEN / 环信凭证等）。
- **单文件 exe**：`打包.py` 一键产出单文件、无控制台、双击即用的 Windows 程序，启动后自动打开浏览器。
- **本地安全防护**：内部接口 HMAC 签名 + nonce 防重放、云端辅助接口 TLS 证书固定、日志全程脱敏，服务仅监听本机 `127.0.0.1`。

## 快速开始（普通用户）

### 1. 下载与运行

1. 前往 [GitHub Releases](https://github.com/kekeaiaixueer/Caoxing_xingchan_im/releases) 下载最新版 `star-chan-im-{日期}.exe`。
2. 双击运行：程序自动探测空闲端口（默认从 8000 起）并打开浏览器页面；控制窗口可随时「打开浏览器 / 退出程序」。
3. 首次运行会在 **exe 同目录** 生成 `data/` 文件夹，账号、消息、签到记录全部保存在这里（若目录不可写则回退到 `%LOCALAPPDATA%\星蝉im`）。

### 2. 添加账号

在「我的账号」页点击「**添加账号**」，两种方式任选其一：

| 方式 | 填写内容 | 说明 |
|------|----------|------|
| ① 手机号 + 密码（推荐） | 学习通手机号、密码、（可选）学校ID | 自动完成学习通登录并获取会话参数，创建后即可直接「启动」监听 |
| ② 仅 Cookie(不可用) | 学习通 Cookie（登录过学习通的浏览器中复制） | 后端自动解析 Cookie 获取会话参数 |

### 3. 启动监听

账号创建完成后，在账号行点击「**启动**」：程序会登录环信并开始监听消息，收到的消息实时入库并推送；需要停止时点击「停止」。

### 4. 签到配置（可选）

进入账号的「**配置**」页，可提前准备签到所需素材：

- **预设位置**：地址 / 纬度 / 经度。非指定位置签到、位置签到降级时会使用此处配置的位置。
- **拍照签到照片**：可上传多张，拍照签到时随机使用其中一张。
- **人脸照片**：仅保留一张，重新上传会覆盖旧图。

## 开发模式

### 后端

```bash
cd backend
python -m venv .venv
.venv\Scripts\activate          # Windows
pip install -r requirements.txt
copy .env.example .env          # 按需修改（管理端密码、环信集群、签到接口等）
.venv\Scripts\python.exe -m alembic upgrade head
python run.py                   # http://127.0.0.1:8000/docs
```

### 前端

```bash
cd frontend
npm install
npm run dev                     # http://127.0.0.1:5173
```

默认管理端账号：`admin / admin123`（请在 `.env` 中修改）。

## 目录结构

```
im_ws_html/
├── ws/                  # 环信 IM SDK（Python）
├── backend/             # FastAPI 后端（按业务域组织）
│   ├── src/             # 应用源码（auth/accounts/chaoxing/im/messages/signins/realtime/internal…）
│   ├── tests/           # pytest 异步测试
│   ├── migrations/      # Alembic 迁移
│   ├── run.py           # 开发启动入口
│   ├── pack_entry.py    # exe 冻结版入口（单实例/便携数据目录/自动开浏览器）
│   └── requirements.txt
├── frontend/            # Vue3 + Naive UI 前端
├── assets/icon.ico      # 打包 exe 图标
├── docs/
│   ├── OPENAPI.md       # Restful OpenAPI 接口文档
│   └── FEATURES.md      # 功能进度
├── 打包.py              # exe 打包脚本（Nuitka onefile）
├── version.txt          # 唯一版本源（发布 exe 与 GitHub tag 均以此为基准）
├── release/             # 打包产物（已 gitignore）
└── README.md
```

## 安全说明

- 本项目为**本地个人工具**：学习通密码、Cookie 与环信凭证明文存储在本地 SQLite，请勿部署到公网。
- 服务仅监听 `127.0.0.1`；打包版强制开启内部接口签名防护（bootstrap 握手 + HMAC-SHA256 + nonce 防重放），`/docs`、调试接口默认关闭。
- 对云端辅助接口做 TLS 证书固定（阻断 Fiddler/Charles 类中间人抓包）；日志全链路脱敏（token/cookie/password 等敏感键不落盘）。

## 免责声明

本项目仅供学习与技术交流使用。请遵守所在学校/机构的相关规定与学习通平台服务条款；因使用本工具产生的一切后果由使用者自行承担。

## 官网

更多信息请访问：<https://sisterxue.top/>
