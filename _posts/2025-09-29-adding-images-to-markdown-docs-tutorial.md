---
layout: post
title: "Adding Images to Markdown Documentation"
draft: false
date: 2025-09-29
description: A simple tutorial for adding and sizing images in markdown files using VS Code, with practical examples for Linux documentation websites.
---

# Adding Images to Markdown Documentation

## Folder Structure Setup

In VS Code, organize your project with a dedicated images folder:

```
your-documentation-site/
├── _posts/
│   ├── 2025-09-25-your-article.md
├── assets/
│   └── images/
│       ├── screenshots/
│       ├── diagrams/
│       └── logos/
└── _config.yml
```

**Best Practice**: Create a `assets/images/` folder at your site root. This keeps images organized and easily accessible.

## Image Sizing Guidelines

### Recommended Image Sizes

- **Screenshots**: 800-1200px wide (will scale down automatically)
- **Small icons/buttons**: 50-100px wide
- **Diagrams**: 600-1000px wide
- **Full-width images**: 1200px+ wide

### File Size Optimization

- Keep images under 500 KB when possible
- Use PNG for screenshots with text
- Use JPG for photos
- Consider WebP for better compression

## Basic Image Syntax

### Standard Markdown Image

```markdown
![Alt text](path/to/image.png)
```

### With Title (hover text)

```markdown
![Alt text](path/to/image.png "Image title")
```

## Sizing Images in Markdown

### Method 1: HTML Tags (Most Control)

```html
<img src="/assets/images/screenshot.png" alt="Terminal screenshot" width="600">
```

### Method 2: HTML with Percentage

```html
<img src="/assets/images/diagram.png" alt="Network diagram" style="width: 80%;">
```

### Method 3: CSS Classes (if your site supports them)

```markdown
![Proxmox dashboard](/assets/images/proxmox-dash.png){: .img-responsive .img-center}
```

## Three Image Examples

### Example 1: Small UI Element (150px)

<img src="/assets/images/terminal-icon.png" alt="Terminal icon in taskbar" width="150">

```html
<img src="/assets/images/terminal-icon.png" alt="Terminal icon in taskbar" width="150">
```

**Use for**: Small UI elements, icons, buttons, menu items

---

### Example 2: Medium Screenshot (600px)

<img src="/assets/images/vs-code-interface.png" alt="VS Code editor showing markdown file" width="600">

```html
<img src="/assets/images/vs-code-interface.png" alt="VS Code editor showing markdown file" width="600">
```

**Use for**: Code editor screenshots, dialog boxes, small application windows

---

### Example 3: Full Screenshot (800px)

```html
<img src="/assets/images/proxmox-dashboard.png" alt="Proxmox VE web interface dashboard" width="800">
```

```html
<img src="/assets/images/proxmox-dashboard.png" alt="Proxmox VE web interface dashboard" width="800">
```

**Use for**: Full application screenshots, complete desktop views, detailed dashboards

## VS Code Workflow

### 1. Taking Screenshots

**Linux (for your documentation)**:

- `gnome-screenshot -a` - Select area
- `gnome-screenshot -w` - Select window
- Save to your Downloads folder first

### 2. Adding Images in VS Code

1. **Create the image folder**: Right-click in Explorer → New Folder → `assets/images`
2. **Copy your screenshot**: Drag from file explorer into the `assets/images` folder
3. **Insert the image**: Type the HTML img tag or use Markdown syntax
4. **Preview**: Use `Ctrl+Shift+V` to preview your markdown

### 3. Quick Image Insert Snippet

In VS Code, create a snippet for faster image insertion:

1. `Ctrl+Shift+P` → "Configure User Snippets" → "markdown"
2. Add this snippet:

```json
"img": {
    "prefix": "img",
    "body": [
        "<img src=\"/assets/images/$1\" alt=\"$2\" width=\"$3\">"
    ],
    "description": "Insert responsive image"
}
```

Now type `img` + Tab to quickly insert image tags!

## File Naming Best Practices

### Good Names

- `proxmox-vm-creation-dialog.png`
- `linux-terminal-setup-complete.png`
- `docker-compose-file-structure.png`

### Bad Names

- `Screenshot1.png`
- `image.png`
- `pic.jpg`

**Rules**:

- Use lowercase
- Separate words with hyphens
- Be descriptive
- Include the article topic

## Testing Your Images

### Before Publishing

1. **Preview in VS Code**: `Ctrl+Shift+V`
2. **Check mobile sizes**: Images should be readable on phones
3. **Verify paths**: Make sure image paths work when published
4. **Alt text**: Always include descriptive alt text for accessibility

### Common Path Issues

```markdown
<!-- Wrong - won't work when published -->
![Screenshot](./images/screenshot.png)

<!-- Correct - works in production -->
![Screenshot](/assets/images/screenshot.png)
```

## Pro Tips

1. **Consistent sizing**: Pick 2-3 standard widths and stick to them
2. **Lazy loading**: Some Jekyll themes support `loading="lazy"`
3. **Dark mode**: Consider how images look on dark backgrounds  
4. **Compression**: Use tools like TinyPNG before adding large images
5. **Alt text**: Write it like you're describing to someone who can't see it

## Example Article Structure

```markdown
---
layout: post
title: "Setting up Proxmox VE"
draft: false  
date: 2025-09-25
description: Complete guide to installing Proxmox VE on a mini PC
---

## Setting up Proxmox VE

First, download the ISO from the official site:

<img src="/assets/images/proxmox-download-page.png" alt="Proxmox download page showing latest ISO" width="600">

Next, boot into the installer. You should see this screen:

<img src="/assets/images/proxmox-installer-menu.png" alt="Proxmox installer boot menu" width="800">

The installation will create a dashboard like this:

<img src="/assets/images/proxmox-final-dashboard.png" alt="Completed Proxmox installation dashboard" width="800">
```

Start with these basics, and you'll have professional-looking documentation with properly sized, accessible images!
