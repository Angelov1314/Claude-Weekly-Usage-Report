---
name: project-viz-report
description: Generate the full weekly Claude Code project report with 5 sections — Ghibli art panels, project overview, tech stack + API links, deployment links, and token/model usage — sent as ONE email to herojerryyang@gmail.com. Trigger when user asks for weekly project report, project visualization, or weekly summary.
---

# Project Visualization Report Skill

Generates Jerry's weekly project report in **one email** with 5 sections. Images are uploaded to GitHub so they embed correctly in Gmail.

---

## Overview of 5 Sections

| # | Section | Content |
|---|---------|---------|
| 1 | 🎨 项目可视化 | Ghibli-style art panels (3 groups of 5) from Nano Banana API, hosted on GitHub |
| 2 | 📋 项目概览 | Project name, date, core features table |
| 3 | 🔧 技术栈 & API 链接 | Tech stack, API services used, GitHub repos |
| 4 | 🔗 部署链接 | Platform badges + live/repo URLs |
| 5 | ⚡ Token & 模型使用情况 | From token-usage skill: totals, per-model, daily bar chart |

---

## Step 1: Collect Project Data

For each project this week, gather:
- **name**, **date range**, **core features** (3 bullets)
- **tech_stack**: read from `package.json`, `requirements.txt`, project files
- **api_links**: services actually used (Anthropic, Firebase, Railway, etc.)
- **deploy_url** or **github_url**: confirmed from git remote or config files
- **description**: one sentence for the Ghibli image prompt

Scan these directories:
- `/Users/jerry/Claude/*/`
- `/Users/jerry/Desktop/projects/*/`
- `/Users/jerry/godot-farm`
- `/Users/jerry/Documents/Unreal Projects/Aurora`
- `/Users/jerry/Downloads/` (Unity projects)

Check git activity: `git -C <path> log --since="7 days ago" --format="%ad | %s" --date=short`
Check Claude sessions: `ls -lt ~/.claude/sessions/*.json` → extract `cwd` field

---

## Step 2: Get Token Usage

Run the token-usage skill script:
```bash
python3 ~/.claude/skills/token-usage/scripts/analyze_tokens.py
```

Extract:
- Total tokens, estimated cost, cache savings, cache hit rate
- Per-model breakdown (Opus/Sonnet/Haiku): responses, tokens, cost, avg per response
- Daily breakdown table

---

## Step 3: Generate Ghibli Art Panels

Group projects into batches of 5. For each group, call the Nano Banana API:

**API:**
- URL: `https://api.vectorengine.ai/v1/chat/completions`
- Model: `gemini-3.1-flash-image-preview`
- Auth: Bearer key from memory (Nano Banana API Key)

**Prompt template:**
```
A wide illustrated storybook panel divided into [N] cozy vignettes,
Studio Ghibli inspired, Hayao Miyazaki art style, watercolor gouache painting,
hand-painted soft brushstrokes, warm amber golden sunlight, muted naturalistic palette,
nostalgic atmosphere, soft watercolor texture, visible painterly brushwork.
Each vignette: 1. [Project] — [scene]. 2. ...
Title banner: "Week of Projects — [Theme]"
```

Save to: `~/Desktop/weekly-project-panels/<group_key>.png`

---

## Step 4: Add Text Overlay Panel

Use Pillow with **STHeiti** font (supports Chinese):

```python
from PIL import Image, ImageDraw, ImageFont

FONT_BOLD = "/System/Library/Fonts/STHeiti Medium.ttc"
FONT_REG  = "/System/Library/Fonts/STHeiti Light.ttc"

def add_info_panel(img_path, group_data, out_path):
    img = Image.open(img_path).convert("RGB")
    w, h = img.size
    panel_w = int(w * 0.42)
    panel = Image.new("RGB", (panel_w, h), (18, 18, 30))
    draw = ImageDraw.Draw(panel)

    fh = ImageFont.truetype(FONT_BOLD, 26)  # group header
    ft = ImageFont.truetype(FONT_BOLD, 17)  # project name
    fs = ImageFont.truetype(FONT_REG,  13)  # features / deploy

    y = 22
    draw.text((20, y), group_data["title"], fill=(255, 200, 60), font=fh); y += 34
    draw.text((20, y), group_data["date"],  fill=(140, 140, 165), font=fs); y += 24
    draw.line([(20, y), (panel_w-20, y)], fill=(55, 55, 80), width=1); y += 14

    for proj in group_data["projects"]:
        if y > h - 70: break
        draw.text((20, y), f"▸ {proj['name']}", fill=(110, 190, 255), font=ft); y += 24
        for feat in proj["features"]:
            draw.text((28, y), f"• {feat}", fill=(200, 200, 215), font=fs); y += 17
        draw.text((28, y), f"⬡ {proj['deploy']}", fill=(80, 210, 120), font=fs); y += 17
        y += 8
        draw.line([(28, y), (panel_w-28, y)], fill=(38, 38, 58), width=1); y += 12

    img_resized = img.resize((w - panel_w, h), Image.LANCZOS)
    combined = Image.new("RGB", (w, h), (18, 18, 30))
    combined.paste(img_resized, (0, 0))
    combined.paste(panel, (w - panel_w, 0))
    combined.save(out_path, "PNG")
```

Save to: `~/Desktop/weekly-project-panels/<group_key>_annotated.png`

---

## Step 5: Upload Images to GitHub

Upload to `Angelov1314/weekly-reports` repo under `YYYY-MM-DD/` folder.

```bash
gh repo view Angelov1314/weekly-reports || gh repo create Angelov1314/weekly-reports --public
# Use GitHub API to upload files (avoids credential issues):
python3 << 'EOF'
import base64, requests, os
from datetime import date

TOKEN = "<gh auth token from `gh auth token`>"
REPO = "Angelov1314/weekly-reports"
FOLDER = date.today().isoformat()
PANEL_DIR = os.path.expanduser("~/Desktop/weekly-project-panels")

for key in ["group1_dev_tools", "group2_visual_web", "group3_games"]:
    fname = f"{key}_annotated.png"
    with open(os.path.join(PANEL_DIR, fname), "rb") as f:
        content = base64.b64encode(f.read()).decode()
    requests.put(
        f"https://api.github.com/repos/{REPO}/contents/{FOLDER}/{fname}",
        headers={"Authorization": f"token {TOKEN}", "Content-Type": "application/json"},
        json={"message": f"Add {fname}", "content": content}
    )
EOF
```

Raw image URLs:
```
https://raw.githubusercontent.com/Angelov1314/weekly-reports/main/YYYY-MM-DD/group1_dev_tools_annotated.png
https://raw.githubusercontent.com/Angelov1314/weekly-reports/main/YYYY-MM-DD/group2_visual_web_annotated.png
https://raw.githubusercontent.com/Angelov1314/weekly-reports/main/YYYY-MM-DD/group3_games_annotated.png
```

---

## Step 6: Build & Send Single HTML Email

Send ONE email via `gmail_create_draft` to herojerryyang@gmail.com.

**Subject:** `📊 Claude Code 周报 — YYYY.MM.DD–MM.DD | {N}项目 · ${cost} · {tokens} tokens`

**5-section HTML structure:**
1. **Header** — date range, total projects, key stats (cost, cache savings, hit rate)
2. **Section 1: 项目可视化** — 3 `<img>` tags using raw GitHub URLs
3. **Section 2: 项目概览** — table: name | date | core features
4. **Section 3: 技术栈 & API** — table: name | tech stack | API links | GitHub
5. **Section 4: 部署链接** — table: name | platform badge | URL
6. **Section 5: Token 使用** — stat cards (total, cost, savings, hit rate) + model table + daily bar chart

**Key constraints:**
- Keep body under 200KB (no inline base64 images — use GitHub URLs instead)
- Write all text in Chinese (Simplified)
- Do NOT include Co-Authored-By in any git commits

---

## Output

- Annotated panels: `~/Desktop/weekly-project-panels/<group>_annotated.png`
- Images live at: `https://github.com/Angelov1314/weekly-reports`
- ONE email draft created in Gmail → send manually or auto-send if tool available
