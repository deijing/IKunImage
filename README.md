<div align="center">

# IKunImage

**Claude Code Skill for AI Image Generation**

通过 [ikun API](https://api.ikuncode.cc) 调用 Gemini 3 Pro Image Preview 模型
在对话中一句话生成高质量图片

[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python&logoColor=white)](https://python.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-blueviolet?logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZmlsbD0id2hpdGUiIGQ9Ik0xMiAyQzYuNDggMiAyIDYuNDggMiAxMnM0LjQ4IDEwIDEwIDEwIDEwLTQuNDggMTAtMTBTMTcuNTIgMiAxMiAyeiIvPjwvc3ZnPg==)](https://docs.anthropic.com/en/docs/claude-code)

</div>

---

## Features

| 能力 | 说明 |
|:---:|------|
| **文生图** | 用自然语言描述场景，AI 生成对应图片 |
| **图生图** | 上传本地图片 + 编辑描述，AI 修改生成新图片 |
| **10 种宽高比** | `1:1` `16:9` `9:16` `4:3` `3:4` `3:2` `2:3` `21:9` `5:4` `4:5` |
| **3 档分辨率** | 1K 快速预览 / **2K 推荐** / 4K 超高清 |
| **文字渲染** | 支持在图片中渲染文字 —— 招牌、海报、标语 |
| **并发批量** | 多张图片并发生成，大幅缩短总耗时 |
| **一键配置** | `--setup` 交互式引导，API Key 安全存储在本地 |

## Quick Start

### 1. 安装

```bash
git clone https://github.com/deijing/IKunImage.git
cd IKunImage

# 复制 skill 到 Claude Code skills 目录
mkdir -p ~/.claude/skills
cp -r skills/ikunimage ~/.claude/skills/

# 安装依赖
pip install httpx
```

### 2. 配置 API Key

```bash
python ~/.claude/skills/ikunimage/scripts/generate_ikun.py --setup
```

按提示输入你的 API Key（从 [api.ikuncode.cc](https://api.ikuncode.cc) 获取），配置会保存到 `~/.ikunimage/config.json`。

> 也支持环境变量 `IKUN_API_KEY` 或命令行 `--api-key` 参数。

### 3. 使用

在 Claude Code 对话中输入：

```
/ikunimage
```

然后描述你想要的图片：

- *"画一张江南水乡的风景"*
- *"生成一张 4K 超宽屏的故宫雪景"*
- *"批量生成 5 张不同风格的古风人像"*
- *"编辑 photo.jpg，把背景改成竹林"*

---

## CLI 独立使用

也可以脱离 Claude Code，直接命令行调用。

**文生图**

```bash
python ~/.claude/skills/ikunimage/scripts/generate_ikun.py \
  -p "一位中国女性，身穿汉服，站在竹林中，晨雾缭绕" \
  -ar 3:4 -s 2K -o ./output.png
```

**图生图**

```bash
python ~/.claude/skills/ikunimage/scripts/generate_ikun_edit.py \
  -i ./photo.jpg \
  -p "将背景改为雪景，保持人物不变" \
  -ar 3:4 -o ./edited.png
```

**批量生成**

```bash
cat > tasks.json << 'EOF'
[
  {"prompt": "描述1", "aspect_ratio": "3:4", "size": "2K", "output": "./out1.png"},
  {"prompt": "描述2", "aspect_ratio": "16:9", "size": "1K", "output": "./out2.png"}
]
EOF

python ~/.claude/skills/ikunimage/scripts/generate_ikun.py --batch tasks.json --workers 2
```

---

## 参数速查

### generate_ikun.py（文生图）

| 参数 | 简写 | 说明 | 默认值 |
|------|:----:|------|:------:|
| `--setup` | | 交互式配置 API Key | — |
| `--api-key` | | 指定 API Key | 配置文件 |
| `--prompt` | `-p` | 图片描述 | **必填** |
| `--aspect-ratio` | `-ar` | 宽高比 | `1:1` |
| `--size` | `-s` | 分辨率 1K / 2K / 4K | `2K` |
| `--output` | `-o` | 输出路径 | `output.png` |
| `--batch` | `-b` | 批量任务 JSON | — |
| `--workers` | `-w` | 并发数 | 自动 |
| `--retry` | `-r` | 重试次数 0-10 | `3` |

### generate_ikun_edit.py（图生图）

| 参数 | 简写 | 说明 | 默认值 |
|------|:----:|------|:------:|
| `--setup` | | 交互式配置 API Key | — |
| `--api-key` | | 指定 API Key | 配置文件 |
| `--input` | `-i` | 输入图片路径 | **必填** |
| `--prompt` | `-p` | 编辑描述 | **必填** |
| `--aspect-ratio` | `-ar` | 输出宽高比 | `1:1` |
| `--output` | `-o` | 输出路径 | `output.png` |
| `--batch` | `-b` | 批量任务 JSON | — |
| `--workers` | `-w` | 并发数 | 自动 |
| `--retry` | `-r` | 重试次数 0-10 | `3` |

---

## 分辨率参考

<details>
<summary><b>点击展开完整分辨率表</b></summary>

### 1K（快速预览）

| 宽高比 | 分辨率 | 宽高比 | 分辨率 |
|:------:|:------:|:------:|:------:|
| 1:1 | 1024×1024 | 16:9 | 1376×768 |
| 9:16 | 768×1376 | 4:3 | 1200×896 |
| 3:4 | 896×1200 | 3:2 | 1232×816 |
| 2:3 | 816×1232 | 21:9 | 1584×672 |
| 5:4 | 1136×896 | 4:5 | 896×1136 |

### 2K（推荐）

| 宽高比 | 分辨率 | 宽高比 | 分辨率 |
|:------:|:------:|:------:|:------:|
| 1:1 | 2048×2048 | 16:9 | 2752×1536 |
| 9:16 | 1536×2752 | 4:3 | 2400×1792 |
| 3:4 | 1792×2400 | 3:2 | 2464×1632 |
| 2:3 | 1632×2464 | 21:9 | 3168×1344 |
| 5:4 | 2272×1792 | 4:5 | 1792×2272 |

### 4K（超高清）

| 宽高比 | 分辨率 | 宽高比 | 分辨率 |
|:------:|:------:|:------:|:------:|
| 1:1 | 4096×4096 | 16:9 | 5504×3072 |
| 9:16 | 3072×5504 | 4:3 | 4800×3584 |
| 3:4 | 3584×4800 | 3:2 | 4928×3264 |
| 2:3 | 3264×4928 | 21:9 | 6336×2688 |
| 5:4 | 4544×3584 | 4:5 | 3584×4544 |

</details>

---

## FAQ

<details>
<summary><b>提示 "未找到 API Key"</b></summary>

运行 `python ~/.claude/skills/ikunimage/scripts/generate_ikun.py --setup` 配置你的 key。
</details>

<details>
<summary><b>请求超时</b></summary>

4K 图片生成较慢，脚本已设置充足的超时时间（4K 为 1200s）。如仍然超时，可降低分辨率到 2K 或 1K。
</details>

<details>
<summary><b>收到 429 错误</b></summary>

触发了 API 频率限制。脚本会自动指数退避重试（默认 3 次）。如需更多重试：`--retry 5`。
</details>

<details>
<summary><b>图生图支持哪些格式？</b></summary>

JPG / JPEG / PNG / WebP / GIF，推荐图片大小 < 4MB。
</details>

---

## 项目结构

```
IKunImage/
├── README.md
└── skills/
    └── ikunimage/
        ├── SKILL.md                  # Claude Code Skill 定义
        ├── scripts/
        │   ├── generate_ikun.py      # 文生图
        │   └── generate_ikun_edit.py # 图生图
        └── references/
            └── api-reference.md      # API 参考
```

## License

[MIT](LICENSE)
