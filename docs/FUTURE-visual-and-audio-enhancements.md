# Enhanced Mahjong Curriculum - Visual Tiles & Audio Pronunciations

## Objective

Add visual tile images and pronunciation audio links to the existing mahjong curriculum to improve learning effectiveness for visual/auditory learners.

## User Requirements

- **Tile Display**: Use actual PNG/SVG image files (not Unicode symbols)
- **Pronunciation Audio**: Link to external services (Forvo.com primary)
- **Coverage**: Focus on glossary terms (~80-100 pronunciation links)

## Current State

- 22 markdown curriculum files with ASCII art tile representations
- Chinese terms include pinyin but no audio
- No image assets in repository
- Inconsistent formatting across files

## Implementation Plan

### 1. Directory Structure

Create new directory structure:
```
/
├── assets/
│   ├── tiles/
│   │   ├── bamboo/        # 1-bam.png through 9-bam.png (generated)
│   │   ├── characters/    # 1-wan.png through 9-wan.png (generated)
│   │   ├── dots/          # 1-dot.png through 9-dot.png (generated)
│   │   ├── winds/         # east.png, south.png, west.png, north.png (generated)
│   │   ├── dragons/       # red.png, green.png, white.png (generated)
│   │   └── bonus/         # flower-1.png through season-4.png (generated)
│   └── README.md          # Generation documentation
├── scripts/
│   ├── generate_tiles.py  # Main generation script
│   ├── tile_prompts.py    # Prompt definitions for DALL-E
│   ├── config.example.json # Template config (committed)
│   ├── config.json         # Actual config with API key (gitignored)
│   └── requirements.txt    # Python dependencies
└── .gitignore             # Updated to ignore config.json
```

**Total Required**: 42 unique tile images (27 suited + 7 honor + 8 bonus)

### 2. Image Generation Strategy

**Primary Method**: OpenAI DALL-E API Script

Create `scripts/generate_tiles.py` to generate all 42 tile images using OpenAI's image generation API.

**Script Components**:
1. `scripts/generate_tiles.py` - Main generation script
2. `scripts/config.example.json` - Template config file
3. `scripts/config.json` - User's actual config (gitignored)
4. `scripts/tile_prompts.py` - Prompt definitions for each tile

**Config Format** (`config.json`):
```json
{
  "openai_api_key": "sk-...",
  "model": "gpt-image-1.5",
  "size": "1024x1536",
  "quality": "high",
  "format": "png",
  "output_dir": "../assets/tiles",
  "base_style": "minimalist traditional mahjong tile, clean white background, professional quality"
}
```

**Model Information**:
- **Model**: `gpt-image-1.5` (OpenAI's latest, replaces deprecated DALL-E 3)
- **Why gpt-image-1.5**: Better instruction following, 20% cheaper than gpt-image-1, best quality
- **Alternative**: `gpt-image-1-mini` for cost savings (lower quality but faster)
- **Note**: May require [API Organization Verification](https://platform.openai.com/docs/guides/image-generation)

**Tile Specs**:
- Generated size: 1024x1536 (portrait orientation for tiles)
- Format: PNG (supports transparency)
- Quality: High
- Resized to: 80x110px for web use
- Style: Consistent traditional mahjong aesthetic

### 3. Pronunciation Link Format

**Service**: Forvo.com (primary)
- Pattern: `https://forvo.com/word/[chinese-term]/#zh`
- Markdown: `**Term (Chinese)** [🔊](forvo-url)`

**Fallbacks**:
- Wiktionary audio links
- Google Translate with pre-filled term
- Pinyin romanization (already present)

### 4. File Update Priority

#### Priority 1: Core Files (Days 1-6)

**A. `/curriculum/01-basics/tiles-and-suits.md`**
- Lines 25-99: Replace ASCII art with image galleries
- Add pronunciation links for suit names (条 tiáo, 万 wàn, 筒 tǒng)
- Keep ASCII in collapsible `<details>` sections
- Add visual tile grid for each suit

**Example transformation**:
```markdown
### 1. Bamboo (Sticks, Bams)
**Chinese:** 条 (tiáo) [🔊](https://forvo.com/word/条/#zh)

<p align="center">
  <img src="../../assets/tiles/bamboo/1-bam.png" width="60" alt="1-Bamboo"/>
  <img src="../../assets/tiles/bamboo/2-bam.png" width="60" alt="2-Bamboo"/>
  <!-- ... all 9 tiles ... -->
</p>

<details>
<summary>ASCII representation (legacy)</summary>
[existing ASCII art]
</details>
```

**B. `/resources/glossary.md`**
- Audit all ~80-100 Chinese terms
- Add [🔊] pronunciation links after each term
- Add "How to Use Pronunciation Links" section
- Standardize entry format

**Template**:
```markdown
**Pong (碰)** [🔊](https://forvo.com/word/碰/#zh) - Triplet
- Mandarin: Peng (pèng)
- Cantonese: Pong
- Three identical tiles
```

**C. `/assets/README.md`** (NEW FILE)
- Document all image sources and URLs
- Include license information (CC-BY-SA, CC0, etc.)
- List attribution requirements
- Note date accessed and contributors
- Provide alternative source documentation

#### Priority 2: Reference Files (Days 7-10)

**D. `/curriculum/01-basics/basic-terminology.md`**
- Lines 13-19: Add audio links to terminology table
- Add pronunciation links for: Pong 碰, Kong 杠, Chow 吃, Hu 和
- Add wind direction pronunciations: 东南西北

**E. `/resources/quick-reference.md`**
- Add small inline tile images in reference tables
- Maintain text for printability
- Add pronunciation quick reference section

#### Priority 3: Extended Files (Days 11-15)

**F. `/curriculum/02-fundamentals/making-sets.md`**
- Add visual examples for Pong/Kong/Chow formations
- Replace text tile descriptions with images in examples

**G. Remaining 16+ curriculum files**
- Update tile references with inline images where helpful
- Add pronunciation links for newly introduced terms
- Maintain consistency with established formats

### 5. Markdown Syntax Standards

**Single Tile**:
```markdown
![1-Bamboo](../../assets/tiles/bamboo/1-bam.png "1-Bamboo (一条)")
```

**Tile Row**:
```markdown
<p align="center">
  <img src="path/1-bam.png" width="60" alt="1-Bam"/>
  <img src="path/2-bam.png" width="60" alt="2-Bam"/>
</p>
```

**Pronunciation Link**:
```markdown
**Mahjong (麻将)** [🔊](https://forvo.com/word/麻将/#zh)
```

**With Fallback**:
```markdown
**Mahjong (麻将)** [🔊 Listen](https://forvo.com/word/麻将/#zh) | [Translate](https://translate.google.com/?sl=zh-CN&text=麻将)
```

### 6. Image Generation Script Details

**Script Structure** (`scripts/generate_tiles.py`):

```python
#!/usr/bin/env python3
"""Generate mahjong tile images using OpenAI GPT-Image-1.5 API"""

import json
import os
from openai import OpenAI
from PIL import Image
import requests
from io import BytesIO

# Tile definitions with prompts
TILES = {
    'bamboo': {
        '1': "Traditional Chinese mahjong tile featuring a sparrow bird, vertical rectangular format, clean white background",
        '2': "Traditional Chinese mahjong tile showing 2 green bamboo sticks arranged vertically, clean white background",
        # ... 3-9
    },
    'characters': {
        '1': "Traditional Chinese mahjong tile displaying the red character 一萬 (one wan), vertical format, clean white background",
        # ... 2-9
    },
    'winds': {
        'east': "Traditional Chinese mahjong tile with the character 东 (East) in black, vertical format, clean white background",
        # ... south, west, north
    },
    'dragons': {
        'red': "Traditional Chinese mahjong tile with the red character 中 (Red Dragon), vertical format, clean white background",
        # ... green, white
    },
    # ... bonus tiles
}

def load_config():
    """Load configuration from config.json"""
    with open('config.json') as f:
        return json.load(f)

def generate_tile(client, prompt, tile_name, config):
    """Generate a single tile image using GPT-Image-1.5"""
    response = client.images.generate(
        model=config['model'],  # gpt-image-1.5
        prompt=f"{config['base_style']}, {prompt}",
        size=config['size'],  # 1024x1536
        quality=config['quality'],  # high
        response_format=config['format']  # png
    )
    return response.data[0].url

def resize_and_optimize(image_path, target_size=(80, 110)):
    """Resize and optimize generated image for web use"""
    with Image.open(image_path) as img:
        img.thumbnail(target_size, Image.Resampling.LANCZOS)
        img.save(image_path, optimize=True, quality=95)

def main():
    """Generate all 42 mahjong tiles"""
    config = load_config()
    client = OpenAI(api_key=config['openai_api_key'])

    for category, tiles in TILES.items():
        for tile_id, prompt in tiles.items():
            tile_name = f"{category}/{tile_id}"
            url = generate_tile(client, prompt, tile_name, config)
            # Download and save tile...
```

**Usage**:
```bash
# 1. Setup
cd scripts
cp config.example.json config.json
# Edit config.json with your OpenAI API key

# 2. Install dependencies
pip install openai pillow requests

# 3. Generate all tiles
python generate_tiles.py

# 4. Generate specific tiles only
python generate_tiles.py --tiles bamboo winds
```

**Cost Estimation** (GPT-Image-1.5, 2026 pricing):
- GPT-Image-1.5 high quality: Approximately $0.032 per image (20% cheaper than GPT-Image-1)
- 42 tiles × $0.032 = ~$1.34 total
- One-time generation cost
- **Alternative**: Use `gpt-image-1-mini` for lower cost (~$0.015/image = $0.63 total)

**Note**: Verify current pricing at [OpenAI Pricing](https://openai.com/pricing)

**Dependencies** (`scripts/requirements.txt`):
```
openai>=1.0.0
Pillow>=10.0.0
requests>=2.31.0
```

### 7. Implementation Sequence

**Days 1-2: Script Development**
1. Create `/scripts/` directory
2. Write `generate_tiles.py` with tile prompts
3. Create `config.example.json` template
4. Write `tile_prompts.py` with all 42 tile definitions
5. Create `.gitignore` entry for `config.json`
6. Add `requirements.txt`

**Days 3-4: Image Generation**
1. User adds OpenAI API key to `config.json`
2. Run generation script for all 42 tiles
3. Review generated images for quality/consistency
4. Regenerate any tiles that need refinement
5. Optimize images (resize, compress)
6. Create `/assets/README.md` documenting generation

**Days 5-6: Testing & Validation**
1. Test one sample markdown file with generated images
2. Verify all images render correctly on GitHub
3. Check file sizes (<50KB after optimization)
4. Ensure consistent styling across all tiles

**Days 7-9: Glossary & Core Files**
1. Update `glossary.md` with pronunciation links (~80-100 terms)
2. Test Forvo availability for top 20 common terms
3. Update `tiles-and-suits.md` with generated tile images
4. Add pronunciation guide section to glossary

**Days 10-12: Reference Files**
1. Update `basic-terminology.md` with audio links
2. Update `quick-reference.md` with tile images
3. Review and test all image paths and pronunciation links
4. Update main `README.md` with enhancement notes

**Days 13-16: Extended Coverage**
1. Update `making-sets.md` and other Phase 2 files
2. Update Phase 3-4 files where relevant
3. Final quality assurance pass (check all 42 images, 80+ audio links)
4. Accessibility review (alt text, screen reader testing)

### 7. Quality Assurance Checklist

**Pre-Deployment**:
- [ ] All 42 tile images present and correctly named
- [ ] All image file sizes <50KB
- [ ] All relative paths correct across file depths
- [ ] 80%+ pronunciation links functional
- [ ] All images have descriptive alt text
- [ ] License documentation complete
- [ ] Test in GitHub markdown renderer
- [ ] Test in VS Code preview
- [ ] ASCII fallbacks in collapsible sections

**Metrics**:
- Image coverage: 42/42 unique tiles (100%)
- Pronunciation coverage: 80-100 glossary terms (80%+)
- Link validity: >95% functional
- Total asset size: <3MB
- Accessibility: All images have alt text

### 8. Accessibility & Fallbacks

**Image Unavailable Fallbacks**:
1. Unicode mahjong characters (🀀 🀁 🀂 etc.)
2. ASCII art in `<details>` sections
3. External link to Wikimedia image

**Pronunciation Link Failures**:
1. Pinyin romanization (already present)
2. Google Translate link with pre-filled term
3. YouTube search for pronunciation videos

**Accessibility Requirements**:
- All images have descriptive alt text
- Pronunciation links have text labels (not just 🔊)
- Text descriptions maintained alongside images
- Screen reader testing

### 9. Maintenance Plan

**Quarterly Tasks**:
- Validate pronunciation links (check for broken links)
- Monitor repository size (<3MB for assets)
- Review license compliance

**As Needed**:
- Update if Forvo or image sources change
- Add new pronunciations based on user requests
- Optimize images if size becomes issue

### 10. Critical Files to Create/Modify

**New Files to Create**:
1. **`scripts/generate_tiles.py`** - Main image generation script
2. **`scripts/tile_prompts.py`** - Prompt definitions for all 42 tiles
3. **`scripts/config.example.json`** - Template config file
4. **`scripts/requirements.txt`** - Python dependencies
5. **`assets/README.md`** - Generation documentation
6. **`.gitignore` updates** - Add `scripts/config.json`

**Existing Files to Modify**:
7. **`curriculum/01-basics/tiles-and-suits.md`** - Replace ASCII with images
8. **`resources/glossary.md`** - Add ~80-100 pronunciation links
9. **`curriculum/01-basics/basic-terminology.md`** - Add audio links
10. **`resources/quick-reference.md`** - Add tile images to references

### 11. Success Criteria

**Must Have**:
- ✓ All 42 unique tiles visualized with images
- ✓ 80+ Chinese terms have pronunciation links
- ✓ Proper licensing documentation
- ✓ All existing ASCII preserved as fallback

**Nice to Have**:
- Interactive tile galleries
- Multiple accent options (Mandarin/Cantonese)
- Printable PDF versions
- Tile flashcard generator

## Estimated Effort

- **Core functionality** (glossary + tiles): 6-8 days
- **Full implementation** (all 22 files): 15-18 days
- **Ongoing maintenance**: 1-2 hours quarterly

## Notes

- Keep existing ASCII art in collapsible sections for accessibility
- Prioritize glossary.md as central pronunciation reference
- Images should enhance, not replace, text descriptions
- Test on mobile devices (GitHub renders differently)
- May need to complete [API Organization Verification](https://platform.openai.com/docs/guides/image-generation) before using GPT-Image models

## Sources

Information about OpenAI's latest image generation models:
- [Introducing 4o Image Generation | OpenAI](https://openai.com/index/introducing-4o-image-generation/)
- [GPT Image 1.5 Model | OpenAI API](https://platform.openai.com/docs/models/gpt-image-1.5)
- [Image generation | OpenAI API](https://platform.openai.com/docs/guides/image-generation)
- [Images | OpenAI API Reference](https://platform.openai.com/docs/api-reference/images)
- [GPT-Image-1.5 rolling out in the API and ChatGPT - OpenAI Developer Community](https://community.openai.com/t/gpt-image-1-5-rolling-out-in-the-api-and-chatgpt/1369443)
- [Gpt-image-1.5 Prompting Guide | OpenAI Cookbook](https://cookbook.openai.com/examples/multimodal/image-gen-1.5-prompting_guide)
