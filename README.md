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

## Original Template

This repo is built on a fork of **Jekyll Now** from [this repository](https://github.com/barryclark/jekyll-now). **Jekyll** is a static site generator that's perfect for GitHub hosted blogs ([Jekyll Repository](https://github.com/jekyll/jekyll))

The website design is a modification of [Jon Barron's website](https://jonbarron.info/). 
