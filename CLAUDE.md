# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a cooking recipe website built on the **nyum** static site generator, with **cooklang** recipe format support. It's a hybrid system that:
1. Stores recipes in `.cook` format (cooklang) in `cooklang_recipes/`
2. Converts them to Markdown format in `_recipes/` using Python tools
3. Builds a static HTML site using Pandoc and Bash scripts

## Key Directories

- `cooklang_recipes/`: Source recipes in `.cook` format with accompanying images
- `_recipes/`: Generated Markdown recipes (created during build)
- `_site/`: Built static website output
- `_templates/`: HTML templates for Pandoc
- `_assets/`: CSS, fonts, icons, and JavaScript
- `cooklang/`: Python package for cooklang parsing and conversion
- `_temp/`: Temporary files generated during build process

## Development Workflow

### Building the Site

1. **Convert cooklang recipes to Markdown**:
   ```bash
   bash cooklang2nyum.sh
   ```
   This script:
   - Installs Python dependencies via pipenv
   - Slugifies image filenames
   - Converts `.cook` files to `.md` files in `_recipes/` using `pipenv run python -m cooklang to-nyum`

2. **Build the static site**:
   ```bash
   bash build.sh
   ```
   Options:
   - `-q` or `--quiet`: Suppress status messages
   - `-c` or `--clean`: Only reset `_site/` and `_temp/` directories
   - `-h` or `--help`: Show help

   The build process:
   - Extracts metadata from Markdown recipes using Pandoc templates
   - Groups recipes by category
   - Generates HTML pages using Pandoc templates
   - Creates search index JSON
   - Builds category pages and index page

3. **Generate PDF version** (optional):
   ```bash
   bash build_pdf.sh
   ```
   Creates a PDF of all recipe pages using puppeteer-cli screenshots and img2pdf

### Testing

Python tests are available for the cooklang parser:
```bash
pipenv run pytest
```

### Deployment

Deploy using rsync (configure `deploy_remote` in `config.yaml`):
```bash
bash deploy.sh
```
Options:
- `-n` or `--dry-run`: Test deployment without making changes
- `-q` or `--quiet`: Suppress status messages

### CI/CD Pipeline

The GitHub Actions workflow (`.github/workflows/build-ci.yml`) automates:
- Installing dependencies (Pandoc, pipenv, puppeteer-cli)
- Building cooklang recipes
- Building the site with production flag
- Generating PDF
- Deploying to GitHub Pages

## Architecture Notes

### Recipe Formats

- **Cooklang format**: Used for source recipes in `cooklang_recipes/`. This is a structured recipe format with special syntax for ingredients (`@ingredient{quantity}`), cookware (`#cookware{}`), and timers (`~timer{quantity units}`).
- **Markdown format**: Generated in `_recipes/` for nyum processing. Contains YAML front matter and step-by-step instructions with ingredients listed before each step.

### Build Pipeline

1. `cooklang2nyum.sh` → Converts `.cook` to `.md` using Python cooklang package
2. `build.sh` → Processes `.md` files with Pandoc to generate HTML
3. The build script creates:
   - Individual recipe pages (`_site/*.html`)
   - Category pages (`_site/*.category.html`)
   - Index page (`_site/index.html`)
   - Search index (`_site/search.json`)

### Python Cooklang Package

The `cooklang/` directory contains a Python package for parsing and converting cooklang recipes:
- `parser.py`: Core cooklang parser
- `nyum.py`: Conversion between cooklang and nyum markdown formats
- `cli.py`: Command-line interface with `to-nyum`, `to-chowdown`, `from-nyum` commands
- `chowdown.py`: Conversion to chowdown format (alternative recipe format)

### Template System

- `_templates/recipe.template.html`: Template for individual recipe pages
- `_templates/category.template.html`: Template for category listing pages
- `_templates/index.template.html`: Template for main index page
- `_templates/technical/`: Utility templates for metadata extraction and processing
- Partials: `head.partial.html`, `footer.partial.html`, `recipe_list_item.partial.html`

### Configuration

- `config.yaml`: Site configuration (title, description, labels, GitHub URL, deployment remote)
- `Pipfile`/`Pipfile.lock`: Python dependencies managed via pipenv
- Production mode: Set `production: yes` in `config.yaml` for CI builds

## Important Constraints

- Recipe filenames should not contain spaces (use slugs)
- No recipe should be named `index.md` (conflicts with generated index)
- Images must be placed alongside `.cook` files in `cooklang_recipes/`
- The `uncategorized_label` in `config.yaml` must not contain odd numbers of double quotes
- When running under GitHub Actions, file modification dates are set to checkout date, so git commit dates are used for `updatedtime`

## Common Tasks

- **Add a new recipe**: Create a `.cook` file in `cooklang_recipes/` with accompanying image
- **Update site configuration**: Edit `config.yaml`
- **Modify templates**: Edit files in `_templates/`
- **Change styling**: Edit `_assets/style.css`
- **Run tests**: `pipenv run pytest`
- **Check Python code quality**: `pipenv run black .`, `pipenv run flake8`, `pipenv run mypy`
- **Full development cycle**: `bash cooklang2nyum.sh && bash build.sh`

## Dependencies

- **Required**: Pandoc 2.8+, Python 3.12+, pipenv, Bash
- **Optional**: puppeteer-cli (for PDF generation), rsync (for deployment)
- **Python packages**: See `Pipfile` for complete list including typer, python-frontmatter, python-slugify, pytest, black, flake8, mypy

## File Naming Conventions

- Source cooklang files: `recipe-name.cook` (no spaces, slugified)
- Generated markdown files: `recipe-name.md` (slugified version of source)
- Generated HTML files: `recipe-name.html` (matches markdown filename)
- Images: `recipe-name.jpg` (must match recipe filename)

## Search Functionality

- Search index is built as `_site/search.json` during build
- Client-side JavaScript in `_assets/search.js` provides live search
- Search works on recipe titles, categories, and descriptions

## Recipe Template Guide

### Metadata Format
```
>> source: [来源]
>> original_title: [英文标题]
>> description: [生动描述]
>> time: [制作时间]
>> size: [份量]
>> author: [作者/来源平台]
>> category: [类别，例如：Pastry, Main Course, Side Dish, Soup]
>> favorite: [yes/留空]
>> [风味标签]: [yes/留空]
```

### Ingredients List
- Ingredients can be listed in sections with optional titles, using the `@ingredient{quantity}` format.
- Example: `蛋黄糊：@鸡蛋黄{3个}，@砂糖{12g}，@植物油{30g}`
- **IMPORTANT**: If an ingredient has no specific quantity, use empty braces: `@ingredient{}`
- Example: `剪点@葱花{}，锅中放@油{}`

### Steps
- Steps focus on narrative flow rather than strict numbering.
- Ingredients and tools can be directly referenced and tagged using `@ingredient{quantity}`.
- **IMPORTANT**: Always include empty braces `{}` for ingredients without specific quantities.
- Parenthetical notes are allowed, e.g., `（无需回温）` or `（例如从4档开始）`.
- Comments starting with `*` are allowed, e.g., `*食材重量仅供参考，具体可凭感觉行事`
