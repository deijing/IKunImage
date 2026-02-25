---
name: ikunimage
description: |
  ikunimage - AI 图片生成器（ikun 专用渠道）。通过 ikun API（api.ikuncode.cc）调用 NanoBananaPro（gemini-3-pro-image-preview）模型，支持文生图和图生图编辑。
  单渠道设计，用户只需配置一个 API Key 即可使用。支持 10 种宽高比、3 种分辨率（1K/2K/4K）、文字渲染、图生图编辑。
  触发条件：
  (1) 用户说 /ikunimage 或要求使用 ikun 渠道生图
  (2) 用户说"ikun 生图"、"用 ikun 画"、"ikun 生成图片"
  (3) 用户说"生成图片"且明确指定使用 ikun
  (4) 用户说"高清生图"、"4K生图"且指定 ikun
  (5) 用户说"图片编辑"、"修改图片"、"图生图"且指定 ikun
  (6) 用户提供图片路径并要求使用 ikun 渠道进行修改
  (7) 用户说"ikun 批量生图"
  (8) 用户说"配置 ikun"、"ikun setup"
---

# ikunimage - AI 图片生成器（ikun 渠道）

通过 ikun API 调用 NanoBananaPro（gemini-3-pro-image-preview）模型，支持文生图和图生图编辑。

## 全局风格约束（强制）

**所有生成的图片必须严格遵循中国风格**：

| 维度 | 强制要求 |
|------|---------|
| **人物** | 中国面孔、东方五官、黑色或深棕色头发（除非用户明确要求其他发色） |
| **服饰** | 中式服装为首选：汉服、旗袍、唐装、中山装、新中式、国潮。现代装也必须符合中国审美 |
| **场景** | 中国场景：江南水乡、古镇、故宫、竹林、茶室、胡同、现代中国都市、中国校园等 |
| **元素** | 中国文化元素：灯笼、折扇、油纸伞、毛笔、茶具、瓷器、梅兰竹菊、祥云、中国结等 |
| **建筑** | 中式建筑：飞檐翘角、白墙黛瓦、园林亭台、现代中国风建筑 |
| **色调** | 偏好中国传统色：朱红、靛蓝、月白、鹅黄、黛绿、藕粉、琥珀、墨色 |
| **文字** | 如需渲染文字，必须使用中文 |
| **氛围** | 融入东方美学意境：留白、含蓄、诗意、雅致 |

**提示词语言**：统一使用中文撰写提示词。

## 输出目录与命名规范

| 项目 | 值 |
|------|------|
| **输出目录** | `./outimage/ikunimage/` |
| **扩展名** | `.png` |

**文件命名规则**：`{YYYYMMDD}_{HHMM}_{主题简称}.png`

| 项目 | 规则 |
|------|------|
| **日期格式** | 当日日期，`YYYYMMDD`，如 `20260225` |
| **时间格式** | 当前时间的时分，24小时制，如 `1430` |
| **主题简称** | 从用户描述中提炼 2-6 个汉字，如 `古风仙侠`、`江南水乡` |
| **批量递增** | 同一批次多张同主题时追加序号：`..._主题_01.png`、`..._主题_02.png` |

**示例**：
```
./outimage/ikunimage/20260225_1430_古风仙侠.png
./outimage/ikunimage/20260225_1430_江南水乡_01.png
./outimage/ikunimage/20260225_1430_江南水乡_02.png
```

生成前必须先 `mkdir -p` 确保目录存在，然后将 `--output` 参数指向按规范命名的文件。

---

## 首次使用配置

ikunimage 需要用户提供 ikun 渠道的 API Key。

### 方式 1：交互式配置（推荐）

```bash
python ~/.claude/skills/ikunimage/scripts/generate_ikun.py --setup
```

### 方式 2：手动创建配置文件

```bash
mkdir -p ~/.ikunimage
echo '{"api_key": "sk-你的key"}' > ~/.ikunimage/config.json
```

### 方式 3：环境变量

```bash
export IKUN_API_KEY="sk-你的key"
```

### API Key 加载优先级

1. `--api-key` CLI 参数（最高）
2. `IKUN_API_KEY` 环境变量
3. `~/.ikunimage/config.json` 中的 `api_key`
4. 均无 → 报错退出，提示运行 `--setup`

---

## 文生图工作流

### Step 1: 解析用户需求

从用户描述中提取：
- **prompt**：图片描述（如果太短则润色补充）
- **宽高比**：默认 `1:1`。映射用户意图："竖版" → `9:16`，"横版" → `16:9`，"超宽" → `21:9`
- **分辨率**：默认 `2K`。映射："快速预览" → `1K`，"超高清/4K" → `4K`
- **输出路径**：`./outimage/ikunimage/{YYYYMMDD}_{HHMM}_{主题简称}.png`

### Step 2: 构建提示词

**全部使用中文撰写**，结构：中国面孔主体 + 中式服饰 + 中国场景 + 东方美学风格 + 光影 + 构图 + 约束

提示词模板：
```
[主体]：一位中国年轻女性，东方精致五官，黑色/深棕色头发，[发型]
[服饰]：[中式服装：汉服/旗袍/新中式/国潮等]
[场景]：[中国场景：江南水乡/古镇/故宫/竹林/中国都市等]
[元素]：[中国文化元素：灯笼/折扇/油纸伞/茶具等]
[光影]：[具体光影描述]
[色调]：[中国传统色：朱红/靛蓝/月白/黛绿等]
[氛围]：[东方美学意境：诗意/雅致/含蓄/留白等]
[约束]：无水印，无多余文字，画面干净，构图完整
```

分辨率详情和提示词技巧参见 [references/api-reference.md](references/api-reference.md)。

### Step 3: 运行脚本

```bash
python ~/.claude/skills/ikunimage/scripts/generate_ikun.py \
  --prompt "描述内容" \
  --aspect-ratio 16:9 \
  --size 2K \
  --output ./outimage/ikunimage/YYYYMMDD_HHMM_主题简称.png \
  --retry 3
```

### Step 4: 展示结果

**生成成功输出格式**:

```
━━━ 图片 #N ━━━
引擎：ikunimage (NanoBananaPro)
比例：[aspect_ratio] → [实际分辨率]
分辨率等级：[1K/2K/4K]

提示词：
[完整提示词]

文件：[保存路径]

调整建议：
- [可调方向]
━━━━━━━━━━━━
```

- 失败：显示错误信息，建议检查 API Key 或调整提示词

---

## 图生图 / 编辑工作流

上传本地图片 + 文字编辑描述，AI 理解原图内容后生成修改后的新图片。

### 支持的图片格式

JPG / JPEG / PNG / WebP / GIF，推荐图片大小 < 4MB。

### 编辑类型示例

| 编辑类型 | 提示词示例 |
|---------|-----------|
| **添加元素** | "在人物旁边添加一只白色的猫" |
| **修改背景** | "将背景改为日落时分的海滩，保持人物不变" |
| **风格转换** | "将这张照片转换为水彩画风格，保持原有构图" |
| **服饰更换** | "将人物的服装改为红色汉服" |
| **季节变换** | "将场景改为冬天下雪的景象" |
| **文字添加** | "在图片顶部添加中文标题「春日物语」" |

### Step 1: 确认输入

从用户输入中提取：
- **输入图片路径**：用户提供的本地图片路径
- **编辑描述**：用户想要的修改内容
- **宽高比**：默认 `1:1`，或根据原图推断

### Step 2: 构建编辑提示词

**提示词结构**：`生成图片：[具体编辑描述]，[保留约束]`

示例：
```
生成图片：将背景改为古镇青石板路，远处有白墙黛瓦的建筑和红灯笼，保持人物姿态和服装不变，光线改为午后暖阳
```

### Step 3: 运行脚本

```bash
python ~/.claude/skills/ikunimage/scripts/generate_ikun_edit.py \
  --input /path/to/original.jpg \
  --prompt "编辑描述" \
  --aspect-ratio 3:4 \
  --output ./outimage/ikunimage/YYYYMMDD_HHMM_编辑主题.png \
  --retry 3
```

### Step 4: 展示结果

```
━━━ 编辑 #N ━━━
引擎：ikunimage (NanoBananaPro 图生图)
输入：[输入图片路径]
比例：[aspect_ratio]

编辑描述：
[完整编辑提示词]

文件：[保存路径]

调整建议：
- [可调方向]
━━━━━━━━━━━━
```

---

## 批量生成

当用户需要一次生成多张图片时，使用并发批量模式。

### 文生图批量

**Step 1: 准备批量任务 JSON 文件**

```json
[
  {
    "prompt": "提示词内容1",
    "aspect_ratio": "3:4",
    "size": "2K",
    "output": "./outimage/ikunimage/20260225_1430_主题_01.png"
  },
  {
    "prompt": "提示词内容2",
    "aspect_ratio": "3:4",
    "size": "2K",
    "output": "./outimage/ikunimage/20260225_1430_主题_02.png"
  }
]
```

**Step 2: 执行**

```bash
python ~/.claude/skills/ikunimage/scripts/generate_ikun.py \
  --batch /tmp/ikun_batch.json \
  --workers 2 \
  --retry 3
```

### 图生图批量

```json
[
  {
    "input": "/path/to/photo1.jpg",
    "prompt": "将背景改为雪景",
    "aspect_ratio": "3:4",
    "output": "./outimage/ikunimage/20260225_1430_雪景编辑_01.png"
  }
]
```

```bash
python ~/.claude/skills/ikunimage/scripts/generate_ikun_edit.py \
  --batch /tmp/ikun_edit_batch.json \
  --workers 2 \
  --retry 3
```

---

## 参数速查表

### 文生图 (generate_ikun.py)

| 参数 | 可选值 | 默认值 | 模式 |
|------|--------|--------|------|
| `--setup` | 无 | - | 配置 |
| `--api-key` | API Key 字符串 | 从配置加载 | 通用 |
| `--prompt` / `-p` | 图片描述文本 | 必填（单图） | 单图 |
| `--aspect-ratio` / `-ar` | 1:1, 16:9, 9:16, 4:3, 3:4, 3:2, 2:3, 21:9, 5:4, 4:5 | 1:1 | 单图 |
| `--size` / `-s` | 1K, 2K, 4K | 2K | 单图 |
| `--output` / `-o` | 文件路径 | output.png | 单图 |
| `--batch` / `-b` | JSON 文件路径 | 无 | 批量 |
| `--workers` / `-w` | 正整数 | 自动（默认 2） | 批量 |
| `--retry` / `-r` | 0-10 | 3 | 通用 |

> `--prompt` 和 `--batch` 互斥，必须二选一。

### 图生图 (generate_ikun_edit.py)

| 参数 | 可选值 | 默认值 | 模式 |
|------|--------|--------|------|
| `--setup` | 无 | - | 配置 |
| `--api-key` | API Key 字符串 | 从配置加载 | 通用 |
| `--input` / `-i` | 输入图片路径 | 必填（单图） | 单图 |
| `--prompt` / `-p` | 编辑描述文本 | 必填（单图） | 单图 |
| `--aspect-ratio` / `-ar` | 1:1, 16:9, 9:16, 4:3, 3:4, 3:2, 2:3, 21:9, 5:4, 4:5 | 1:1 | 单图 |
| `--output` / `-o` | 输出文件路径 | output.png | 单图 |
| `--batch` / `-b` | JSON 文件路径 | 无 | 批量 |
| `--workers` / `-w` | 正整数 | 自动（默认 2） | 批量 |
| `--retry` / `-r` | 0-10 | 3 | 通用 |

> `--input`/`--prompt` 和 `--batch` 互斥。

---

## 用户追加修改

用户可在生成后要求调整：
- "光线换成逆光" → 修改光影段重新生成
- "背景改成户外" → 修改背景 + 环境光重新生成
- "换成竖版" → 仅更换 aspect_ratio 重新生成
- "换 4K" → 使用 4K 分辨率重新生成

### 图生图追加修改

- "再加一只猫" → 修改编辑描述重新生成
- "背景再暗一些" → 调整编辑描述中的光影/色调部分
- "换成横版" → 仅更换 aspect_ratio 重新生成
- "用另一张图" → 替换 --input 参数

## 注意事项

- 单渠道（ikun），无多渠道切换，重试在同渠道内进行（指数退避）
- 图片过大（> 4MB）会导致上传变慢或超时，建议压缩后再上传
- 编辑提示词中明确说"保持XX不变"可以提高保留原图元素的准确率
- 依赖：`pip install httpx`
