---
name: ppt-speech-fusion
slug: ppt-speech-fusion
displayName: PPT演讲稿
description: 会议录音+PPT一键生成演讲复原稿PDF。丢入录音和PPT文件，自动转写对话、提取幻灯片、逐页对齐生成上半页演讲叙述+下半页原版幻灯片的专业PDF，内置自检。适用于会议纪要、汇报整理、演讲复盘。支持 m4a/mp3/wav 音频 + pptx。
version: 1.0.3
author: ethan
trigger:
  - "把录音和PPT整理成演讲稿"
  - "生成PPT演讲复原稿"
  - "录音加PPT输出PDF"
  - "对话转写配PPT"
  - "帮我做会议演讲稿件"
  - "音频转写加幻灯片"
  - "整理汇报稿"
  - "PPT演讲稿"
  - "整理会议录音"
  - "录音转演讲稿"
  - "把PPT配对话生成讲稿"
  - "会议纪要配幻灯片"
  - "汇报录音整理成逐页稿件"
  - "对话转写+PPT输出PDF"
---

# PPT 演讲稿生成

将会议录音 + PPT 文件融合为演讲复原 PDF：每页上半部分为通顺流畅的演讲叙述，下半部分为对应的 PPT 幻灯片，并自动执行质量自检。

## 功能

1. 音频转写：对会议录音做完整中文转写，梳理对话脉络与核心要点
2. PPT 提取：提取 PPT 全部页面文字结构，导出每页幻灯片截图
3. 演讲复原：将对话内容按 PPT 页面顺序展开为通顺的演讲口吻，逐页对应
4. 自动自检：检查页数一致性、语句通顺度、主题匹配度，有问题自动修复

## 执行流程

用户提供录音文件和 PPT 文件，说出触发词即可执行。整个过程分三个阶段：

### 阶段一：音频转写与对话梳理
- 对录音做完整中文转写（推荐 Whisper，也可用平台自带语音识别）
- 按主题/阶段梳理对话脉络
- 提炼核心要点、数据、结论

### 阶段二：PPT 提取与融合生成 PDF
- 提取 PPT 全部页面文字与结构
- 将每页导出为完整幻灯片截图（非单元素图片；可用 LibreOffice、python-pptx + Pillow 等实现）
- 对话内容复原为演讲口吻叙述，按 PPT 页面顺序逐页展开
- 格式：叙述文字在上，对应 PPT 幻灯片截图在下
- 输出 PDF（可用 ReportLab、fpdf2、wkhtmltopdf 等生成）

### 阶段三：自检
- 核对 PDF 页数是否与 PPT 页数一致
- 抽查 3~5 页叙述文字是否通顺、有无语病或错别字
- 随机 3 页确认对话内容与 PPT 页面主题匹配
- 确认每页下方幻灯片截图清晰、无裁切
- 发现问题直接修复，重新输出 PDF

## 进度反馈

- 阶段一完成时：「音频转写完成，正在提取 PPT 内容...」
- 阶段二完成时：「PDF 已生成，正在执行自检...」
- 阶段三完成时：输出自检结果表格 + 产物文件路径

## 输出格式

- 文件：`{PPT主题}_演讲复原稿.pdf`
- 自检结果以 Markdown 表格呈现，逐项标注通过/修复情况

## 所需能力（平台自行映射）

本 skill 是纯指令型工作流，不含可执行脚本。各 AI 平台根据自身工具链映射以下能力：

| 能力 | 说明 | 常见实现方式 |
|------|------|-------------|
| 音频转写 | 中文语音转文字 | Whisper、平台内置语音识别、云 API |
| PPT 解析 | 读取 pptx 文字结构 | python-pptx、LibreOffice、平台内置 |
| PPT 截图 | 导出每页为图片 | LibreOffice --headless、python-pptx + Pillow、平台内置 |
| PDF 生成 | 图文排版输出 PDF | ReportLab、fpdf2、wkhtmltopdf、平台内置 |
| 文件读写 | 读录音/PPT，写 PDF | 各平台通用能力 |

## 适用场景

- 会议录音 + PPT → 会议演讲复原稿
- 产品汇报录音 + 产品 PPT → 汇报整理文档
- 培训录音 + 课件 → 培训讲稿
- 路演录音 + 路演 PPT → 演讲复盘文档

## 异常处理

- 若音频格式不支持：提示用户转换为 m4a/mp3/wav 后重试
- 若 PPT 格式不支持：提示用户确认文件为 .pptx 格式
- 若音频质量过差导致转写率低：告知用户并建议使用更清晰的录音
- 若 PPT 页数过多（超过 50 页）：提示用户可能耗时较长，确认是否继续

## WorkBuddy 执行映射（附录）

在 WorkBuddy 环境中，以下为确定性工具链映射，直接使用无需用户额外配置。

### 环境准备（首次执行时自动完成）

```bash
pip install openai-whisper python-pptx Pillow reportlab PyMuPDF 2>&1
```

### 阶段一：音频转写

```python
import whisper
model = whisper.load_model("medium")
result = model.transcribe("录音文件路径", language="zh")
with open("转写文本.txt", "w") as f:
    f.write(result["text"])
```

- 若录音超过 1 小时，先用 `ffmpeg -i 录音.m4a -ac 1 -ar 16000 录音_16k.wav` 预处理加速

### 阶段二：PPT 截图 + 生成 PDF

**A. PPT 截图（二选一，优先 LibreOffice）**

方案一（推荐）：
```bash
soffice --headless --convert-to pdf 文件.pptx
python3 -c "
import fitz
doc = fitz.open('文件.pdf')
for i, page in enumerate(doc):
    pix = page.get_pixmap(dpi=200)
    pix.save(f'slide_{i+1:02d}.png')
"
```

方案二（无 LibreOffice 时降级）：
```python
from pptx import Presentation
from PIL import Image, ImageDraw, ImageFont
prs = Presentation('文件.pptx')
for i, slide in enumerate(prs.slides):
    img = Image.new('RGB', (1280, 720), 'white')
    draw = ImageDraw.Draw(img)
    y = 20
    for shape in slide.shapes:
        if shape.has_text_frame:
            for para in shape.text_frame.paragraphs:
                draw.text((40, y), para.text, fill='black')
                y += 24
    img.save(f'slide_{i+1:02d}.png')
```

**B. 生成最终 PDF**

```python
from reportlab.lib.pagesizes import A4
from reportlab.platypus import SimpleDocTemplate, Paragraph, Image, Spacer
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.lib.units import mm

doc = SimpleDocTemplate("演讲复原稿.pdf", pagesize=A4)
styles = getSampleStyleSheet()
story = []

for i, text in enumerate(叙述文本数组):
    story.append(Paragraph(f"第{i+1}页", styles['Heading2']))
    story.append(Paragraph(text, styles['Normal']))
    story.append(Spacer(1, 10*mm))
    story.append(Image(f"slide_{i+1:02d}.png", width=180*mm, height=100*mm))
    story.append(Spacer(1, 5*mm))

doc.build(story)
```

### 阶段三：自检

```python
import os
slides = sorted([f for f in os.listdir('.') if f.startswith('slide_')])
print(f"PPT 页数: {len(slides)}")
print(f"叙述文本数: {len(叙述文本数组)}")
print(f"页数匹配: {len(slides) == len(叙述文本数组)}")
# 抽查叙述文本非空
for i, t in enumerate(叙述文本数组):
    if len(t) < 30:
        print(f"⚠ 第{i+1}页叙述过短")
```

### 产出物

- `{PPT主题}_演讲复原稿.pdf` — 最终交付文件
- `转写文本.txt` / `slide_*.png` — 中间产物（任务结束可清理）
