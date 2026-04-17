# Personal Website

This repository contains a single-page personal website built on Jekyll. The website features three main sections:

1. **Research** - Showcase your research projects and publications
2. **Teaching** - Display your teaching experience and courses
3. **Work Experience** - Highlight your professional experience

## Customization

To customize this website for your own use:

1. **Update _config.yml**: 
   - Change `name` to your name
   - Update `avatar` to point to your profile photo
   - Update `email`, `github`, `linkedin` and other social links
   - Update the `url` to your GitHub Pages URL

2. **Update Bio in _layouts/default.html**:
   - Edit the bio section (around lines 33-47) with your own information
   - Update the research interests description (around line 60)

3. **Add Your Content**:
   - Create posts in the `_posts/` directory
   - Use category `research` for research projects
   - Use category `teaching` for teaching experience
   - Use category `work_experience` for work experience
   - Follow the format in the sample posts

4. **Add Your Images**:
   - Place images in the `images/` directory
   - Update your profile photo and reference it in `_config.yml`
   - Images referenced in posts will automatically use thumbnails from `tn/images/`

## Blogs Section

The site now includes a dedicated blogs experience at `/blogs/`, powered by the `_blogs/` collection.

### Create a New Blog Post

1. Add a new Markdown file in `_blogs/`, for example:
   - `_blogs/my-topic-part-1.md`
2. Use front matter like this:

```yaml
---
title: "My Blog Title"
date: 2026-04-17 12:00:00 +00:00
author: "Your Name"
summary: "One-line summary shown on /blogs/"
subtitle: "Optional subtitle shown on the post page"
reading_time: "8 min read"
tags: [robotics, sim2real]
series: "My Series Name"   # optional, for multi-page/chapter blogs
part: 1                    # optional, works with series for previous/next links
toc:
  - id: section-id
    title: Section title
---
```

### Multi-Page Blogs

- Use the same `series` value across chapters.
- Increment `part` (`1`, `2`, `3`, ...).
- The blog layout automatically renders previous/next chapter links.

### Supported Rich Content

- **Images + captions**:
  - Use `<figure class="blog-figure"> ... </figure>`
- **Videos**:
  - YouTube/Vimeo iframe: wrap in `<div class="blog-video-embed">...</div>`
  - Native video: `<video controls><source src="/videos/your-file.mp4" type="video/mp4"></video>`
- **Code snippets**:
  - Use fenced code blocks (triple backticks), syntax highlighting is preserved
- **Math (LaTeX)**:
  - Inline math: `$E = mc^2$`
  - Display math: `$$ ... $$`
  - KaTeX is enabled automatically for blog pages
- **Callouts**:
  - `<div class="blog-callout"> ... </div>`
  - Variants: `blog-callout success`, `blog-callout note`

### Starter Templates

See these starter examples:

- `_blogs/sim2real-practitioners-guide-part-1.md`
- `_blogs/sim2real-practitioners-guide-part-2.md`

## Original Template

This repo is built on a fork of **Jekyll Now** from [this repository](https://github.com/barryclark/jekyll-now). **Jekyll** is a static site generator that's perfect for GitHub hosted blogs ([Jekyll Repository](https://github.com/jekyll/jekyll))

The website design is a modification of [Jon Barron's website](https://jonbarron.info/). 
