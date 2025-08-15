# AI Assistant Instructions for Saggie Haim Blog

This repository contains a Hugo-based technical blog focused on PowerShell, Azure, DevOps, and automation topics. Here's what you need to know to assist effectively:

## Project Structure

- Built with Hugo static site generator using the Tailwind theme
- Content structure:
  - Blog posts in `Saggiehaim-blog/content/posts/` with each post in its own directory containing `index.md`
  - Static images in `Saggiehaim-blog/content/images/`
  - Page layouts in `Saggiehaim-blog/layouts/`
  - Configuration in `Saggiehaim-blog/hugo.toml` and `config/_default/`

## Content Conventions

1. Blog posts should follow this frontmatter structure:
```toml
+++
author = "Saggie Haim"
title = 'Post Title'
date = YYYY-MM-DD
image = "/images/cover-image.jpeg"
draft = false
description = "Concise description of the post"
toc = false
tags = ["Tag1", "Tag2"]
categories = ["Category1", "Category2"]
keywords = ["SEO Keyword1", "SEO Keyword2"]
+++
```

2. Images:
   - Store in `content/images/`
   - Use relative paths starting with `/images/`
   - Optimize for web before adding

## Development Workflow

1. Local development requires:
   - Node.js for Tailwind CSS
   - Hugo extended version
   
2. Development commands (from `Saggiehaim-blog/`):
```bash
npm install               # Install dependencies
npm run dev              # Start dev server with live reload
npm run dev-tailwind     # Watch and compile Tailwind CSS
```

## Common Patterns

- Use markdown for content with inline HTML only when necessary
- Posts typically include:
  - Clear prerequisites section
  - Code blocks with syntax highlighting (```PowerShell)
  - Screenshots or diagrams where helpful
  - References and links to official documentation

## Key Files

- `layouts/_default/baseof.html`: Main template structure
- `config/_default/hugo.toml`: Core site configuration
- `package.json`: Build scripts and dependencies
- `assets/css/main.css`: Custom Tailwind CSS

Remember to maintain the technical and educational focus of the blog while keeping content practical and actionable.
