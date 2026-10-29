# PS5LM — Chinese UI

使用 DeepSeek V4.1 Flash 一句话汉化：把界面文案换成中文、给字体烘一套 GB2312 汉字，然后重新编译。

[PS5LM](https://github.com/cobanov/PS5LM) 让零售 PS5 直接跑 llama.cpp，把这台游戏机变成局域网里的模型服务器。
本仓库是它的**中文界面版**：电视上的 home / models / api / console / settings 五个页面全部中文，
聊天面板里模型吐出来的中文也不会再变成 `????`。

下载即用，不需要编译、不需要 WSL、不需要认识那堆交叉编译依赖。

> 上游项目：[cobanov/PS5LM](https://github.com/cobanov/PS5LM) · 许可：GPL-3.0-or-later
> 本仓库的改动同样以 GPL-3.0-or-later 发布。

---

## 一、装到 PS5 上（下载即用）

前提：PS5 已经越狱，`/data` 可写，`ps5-payload-dev` 那套 payload 能跑。这些是上游的事，[它的 README](https://github.com/cobanov/PS5LM) 写得很清楚。

1. 到本仓库的 **Releases** 下最新那个 `ps5lm-app-v*-zh*.zip`
2. 解压，把整个 `PPSA99575` 文件夹传到 PS5 的 `/data/homebrew/` 下（FTP 走 2121 端口）
3. **重启 PS5**（ShadowMountPlus 只在开机那一刻扫 `/data/homebrew`）
4. 主屏点 `WebKit Autoloader`，等它跑完
5. 再点 `PS5LM` 图标

进不去的话看日志：`/data/PS5LM/app.log`，里面应该有 `ps5lm-app: dashboard ready`。

模型自己去 Hugging Face 下（`.gguf`），放进 `/data/PS5LM/models/`。本仓库不提供模型。

## 二、和原版的差别

| 项 | 原版 | 本仓库 |
|---|---|---|
| 界面语言 | 英文硬编码 | 中文 |
| dashboard 字体 | Manrope，只含拉丁字符 | Noto Sans SC，含 **GB2312 全集 6763 汉字** |
| 字体体积 | 约 1.8 MB | 约 16.7 MB（含汉字的代价） |
| 聊天面板 | 模型输出中文显示为 `????` | 正常显示 |
| 中文长文本换行 | 按空格断词，整段被截断 | 按 UTF-8 字符断行 |

改动都在这几个文件里：

```
ps5/app/dashboard.cpp      界面文案（词条表见 scripts/translations-zh.csv）
ps5/app/main.cpp           电视通知文案 + 字体读取上限
ps5/app/stats.cpp          Console 页标签
ps5/app/market.cpp         下载页提示
ps5/app/model_plan.cpp     模型适配说明
ps5/app/activity.cpp       聊天头信息（"N 个问题" / "N 张图"）
ps5/app/suggested.hpp      推荐模型的一句话介绍
ps5/app/assets/fonts/      烘焙好的中文字体
scripts/bake-fonts-cn.sh   字体烘焙脚本
scripts/gen-charset.py     字符集生成器
scripts/apply-translations.py  按词条表批量替换源码
scripts/translations-zh.csv    词条表（文件 / 英文 / 中文）
scripts/build-app.sh       CMake 4.x 的 RPATH 兼容
```

## 三、自己编译

想改译文、或者要跟自己的上游同步，就自己编。**必须在 x86-64 Linux 上**（Arch / Fedora 都行，Ubuntu 26.04 实测也过），Windows 只能当宿主。

装依赖见上游的 [docs/BUILDING.md](docs/BUILDING.md)，另外补这些：

```bash
# Ubuntu 26.04 / Debian
apt-get install -y llvm-21-dev libllvmspirvlib-21-dev libclang-21-dev libclc-22-dev python3-fonttools
```

> 注意：`vulkan-headers`、`shaderc` 这两个包名在新版 Debian/Ubuntu 里不存在，真名是
> `libvulkan-dev`、`libshaderc-dev`。一包失败整批 apt 都会废掉。

然后：

```bash
# 上游的流程，一步不少
PS5_MESA_REVISION=$(git -C .deps/src/PS5_Mesa rev-parse HEAD) \
  bash .deps/src/PS5_Vulkan/tools/build-radv.sh release
scripts/build-app.sh        # 产出 build/app-ps5/title/PPSA99575

# 打包
scripts/release-app.sh
```

**就这样，直接能出中文版** —— 中文字体已经烘好提交在 `ps5/app/assets/fonts/` 里（16.7 MB），
`build-app.sh` 只是把它拷进 title 目录，不重新烘。

只有下面两种情况才需要动字体：

```bash
# ① 改了译文，新加的字不在现有字符集里
# ② 想换字号/换字体
mkdir -p .deps/fonts
curl -L -o .deps/fonts/NotoSansSC-VF.ttf \
  'https://raw.githubusercontent.com/google/fonts/main/ofl/notosanssc/NotoSansSC%5Bwght%5D.ttf'
scripts/bake-fonts-cn.sh     # 会先扫源码生成字符集，再烘三个字重
```

`bake-fonts-cn.sh` 用的是 `scripts/font-baker/bake_font.cpp`（本仓库自带的那份，
支持 `@字符集文件` 参数），不依赖 submodule 里原版烘焙器。

字体必须是 **TrueType 轮廓**（文件头 `00 01 00 00`）。OTF 是 CFF 轮廓（`OTTO`），
`stb_truetype` 根本不认，烘出来是空的 —— 这个坑踩过。

## 四、没汉化的地方

老实列出来，省得你找：

- **接收到的英文原文**：别的设备发来的英文提问、模型的英文回答，原样显示（那是内容不是界面）
- **生僻字**：字体装的是 GB2312 全集，覆盖日常用字 99.9%。超出范围的冷僻字、异体字
  （比如 `𠮷`）仍然缺字形。要全覆盖得换 GBK 或做分块加载，工作量另算
- **日志和代码块**：Console 页的日志是 llama.cpp 原样输出，保持英文（那是给排查用的）
- **模型名和量化名**：`Qwen3.8 27B`、`UD-Q2_K_XL` 这些不翻，翻了反而认不出

## 五、跟上游同步

```bash
git remote add upstream https://github.com/cobanov/PS5LM.git
git fetch upstream
git rebase upstream/main zh-cn
```

冲突只在译文那几行。上游要是改了界面布局（加了新的英文串），跟着在 `dashboard.cpp` 里补一条就行。

## 六、反馈

界面漏翻、翻译别扭、某个字还是问号 —— 开 issue，附上：哪个页面、原句、截图（有最好）。
日志在 `/data/PS5LM/app.log`，`grep font` 能看出字体有没有加载成功。

---

<sub>上游 PS5LM 由 cobanov 与各位贡献者维护。PS5 是 Sony 的商标，本仓库与 Sony 无关。</sub>
