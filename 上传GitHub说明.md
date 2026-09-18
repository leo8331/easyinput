# 上传到 GitHub 说明（V0.1）

> **核心结论**：**259 MB 的 EXE 和 427 MB 的模型 zip 都不进 Git 仓库** —— 走 **Releases 附件**。
> 仓库里只放源码 + 小文件（README、固件 bin、.gitignore）。

## 一、为什么不直接 commit 大文件

| 文件 | 大小 | GitHub 限制 | 结论 |
|---|---|---|---|
| `_internal/`（259 MB / 1251 文件） | 259 MB | 单文件 ≤100 MB，仓库建议 ≤1 GB | ❌ 不进仓库 |
| `EasyInput.exe` | 13 MB | 可以，但没意义（缺 `_internal` 跑不了） | ❌ 不进仓库 |
| `models-faster-whisper-small.zip` | 427 MB | **超 100 MB 单文件硬限** | ❌ 进 **Release 附件** |
| `firmware/easyinput_fw.bin` | 1.2 MB | ✅ 可以 | ✅ 进仓库 |
| `README.md` | 小 | ✅ | ✅ 进仓库 |

> 想强行把大文件进仓库得用 **Git LFS**（免费额度 1 GB 存储/月）——不推荐，Release 附件更省事。

## 二、推荐流程（10 分钟）

### 1. 建仓库 + 放小文件

```bash
# 在发布目录里初始化（只提交小文件）
cd "d:\Easyinput project\软件平台\发布\esayinputV0.1"
git init
git add README.md 使用说明.md 上传GitHub说明.md firmware/ .gitignore
git commit -m "EasyInput V0.1: README + firmware + release notes"
git branch -M main
git remote add origin https://github.com/<你的用户名>/<仓库名>.git
git push -u origin main
```

### 2. 发 Release（把大文件作为附件）

```bash
# 用 gh CLI（最简单）
gh release create v0.1 \
  --title "EasyInput V0.1" \
  --notes "首个公开打包版：转录 / 划词 AI 编辑 / 语音提问（含联网）/ 宏 / 灯效" \
  models-faster-whisper-small.zip
```

没有 `gh` 的话，网页操作：
**仓库 → Releases → Draft a new release → Tag `v0.1` → 拖入 zip 到附件区 → Publish**

> ⚠️ Release 单附件上限 **2 GB**，427 MB 没问题。

### 3. 如果想把整个 EXE 目录也给用户

两个选择：

| 方案 | 做法 | 用户拿到 |
|---|---|---|
| **A. EXE 也放 Release**（推荐） | 把整个 `esayinputV0.1` 压成 zip（约 120 MB，DLL 压缩率高）再上传 | 解压即用 |
| **B. 仓库里放打包脚本** | 上传 `打包EXE.bat` + `app/` 源码，用户自己打 | 需要 Python 环境 |

**A 的压缩命令**（PowerShell，在 `发布\` 目录）：
```powershell
Compress-Archive -Path "esayinputV0.1\*" -DestinationPath "EasyInput-V0.1-win64.zip" -CompressionLevel Optimal
```

## 三、`.gitignore`（已随本目录提供）

关键几条：
```
_internal/                 # 259MB 程序本体 → 走 Release
*.zip                      # 模型/打包 zip → 走 Release
config.json                # ⚠️ 含 API Key，绝不能提交
macros.json
app_log.txt
last_recording.wav
easyinput_exe_error.log
models/                    # 模型目录（用户自己解压）
```

## 四、⚠️ 上传前检查（重要）

```powershell
cd "d:\Easyinput project\软件平台\发布\esayinputV0.1"
# 1) 确认没有配置文件（含密钥）
dir config.json, macros.json, app_log.txt, last_recording.wav
# 2) 确认 git 不会追踪大文件
git status --short
git ls-files | Measure-Object    # 文件数应该很小（十几条）
```

**必查项**：

- [ ] `config.json` **不在**待提交列表（里面有你的 API Key）
- [ ] `_internal/` 被 .gitignore 排除（`git status` 看不到它）
- [ ] 没有 `.wav` / `.log` 文件
- [ ] README 里的下载链接与实际 Release 附件名一致

## 五、用户下载后的路径

Release 页面会给用户两个文件：

1. `EasyInput-V0.1-win64.zip`（程序，解压即用）
2. `models-faster-whisper-small.zip`（可选，离线识别模型）

README 里已经写清"解压到同级目录"，用户照做即可。
