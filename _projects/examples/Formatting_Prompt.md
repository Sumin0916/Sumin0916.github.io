You are an expert technical blog writer and developer. Please write a comprehensive project post based on the topic I provide.

You MUST strictly adhere to the following formatting constraints and layout rules for my Jekyll-based portfolio blog:

### [Jekyll Blog Template Constraints & Formatting Rules]

**1. Front Matter Configuration (Metadata)**
Every markdown file must start with a YAML front matter block enclosed by `---`. Use the following parameters accurately:

- `layout`: Always set to `page`.
- `title`: The title of the project or post (Emojis are fully supported, e.g., `title: project 9 🎉`).
- `description`: A short summary or subtitle.
- `img`: Relative path to the thumbnail/background image (e.g., `assets/img/12.jpg`). Leave empty if no thumbnail is needed.
- `importance`: An integer determining the display order (lower values like 1 mean higher priority and appear first).
- `category`: The grouping tag for the project (e.g., `work`, `fun`).
- `related_publications`: Set to `true` to automatically append a related bibliography/publications section at the bottom.
- `giscus_comments`: Set to `true` to enable a GitHub-backed Giscus comment section at the bottom.
- `redirect`: Optional external URL string. If provided, visiting this page will automatically redirect the user to that link (e.g., `redirect: https://www.wikipedia.org/`).

**2. Media & Grid Layouts (Bootstrap 4)**
_CRITICAL: DO NOT use standard Markdown image syntax (`![]()`)._ Instead, use the custom `figure.liquid` include tag wrapped inside Bootstrap grid structures to create responsive multi-column layouts.

- **Standard Image Component**:
  `{% include figure.liquid loading="eager" path="assets/img/X.jpg" title="image title" class="img-fluid rounded z-depth-1" %}`
- **3-Column Grid (1/3 width each)**:
  Wrap three `<div class="col-sm mt-3 mt-md-0">` blocks inside a single `<div class="row">` block.
- **Asymmetrical Grid (2/3 + 1/3 width)**:
  Wrap one `<div class="col-sm-8 mt-3 mt-md-0">` and one `<div class="col-sm-4 mt-3 mt-md-0">` inside a `<div class="row justify-content-sm-center">` block.
- **Single Centered Image**:
  Wrap a single `<div class="col-sm mt-3 mt-md-0">` block inside a `<div class="row">` block.

**3. Typography & Advanced Features**

- **Image Captions**: Place a `<div class="caption"> Your caption text here. </div>` immediately below any `<div class="row">` block to add a styled, centered, and italicized caption for the image array.
- **Academic Citations**: Use `{% cite citation_key %}` (e.g., `{% cite einstein1950meaning %}`) for inline bibliography references integrated with Jekyll Scholar.
- **Escaping Liquid/HTML Code Blocks**: When showcasing raw code blocks (\`\`\`) containing HTML or Liquid tags without rendering them on the site, you MUST wrap the entire markdown code block with `{% raw %}` and `{% endraw %}` tags.

---

**Topic / Task:**
[이곳에 작성하고 싶은 글의 주제, 예를 들어 "SO ARM101 ACT 모델 적용 과정" 및 핵심 내용을 자유롭게 적어주세요.]
