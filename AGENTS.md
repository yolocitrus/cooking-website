# AGENTS.md

Agent instructions for **cooking-website** (cooking.yolocitrus.com).

## Pipeline

```
cooklang_recipes/*.cook + *.jpg
  ↓ cooklang2nyum.sh  (Python: slugify images, convert .cook → .md)
_recipes/*.md
  ↓ build.sh  (Pandoc: extract metadata, render HTML templates)
_site/  ← individual recipe pages, category pages, index.html, search.json
```

## Commands

```bash
bash cooklang2nyum.sh && bash build.sh   # full rebuild (always run together)
bash build_pdf.sh                         # optional: PDF via puppeteer-cli
bash deploy.sh -n                         # dry-run deploy (rsync)
pipenv run pytest                         # parser tests
pipenv run black . && pipenv run flake8 && pipenv run mypy
```

`build.sh` flags: `-q` quiet, `-c` clean only, `-h` help.
`deploy.sh` flags: `-n` dry-run, `-q` quiet.

## Key Paths

| Path | Notes |
|---|---|
| `cooklang_recipes/` | Source `.cook` files + images — **edit here** |
| `_recipes/` | Generated Markdown — do not edit |
| `_site/` | Built HTML output — do not edit |
| `_temp/` | Build scratch space — do not edit |
| `cooklang/` | Python package: `parser.py`, `nyum.py`, `cli.py`, `chowdown.py` |
| `_templates/` | Pandoc templates: `recipe`, `category`, `index`, partials |
| `_assets/` | `style.css`, `search.js`, fonts, icons |
| `config.yaml` | Site title, labels, GitHub URL, deploy remote |
| `.github/workflows/build-ci.yml` | CI: full build + deploy to GitHub Pages on push to `main` |

## Adding a Recipe

1. `cooklang_recipes/<slug>.cook` — hyphens only, never named `index`
2. `cooklang_recipes/<slug>.jpg` — base name must exactly match the `.cook` file
3. Run full rebuild

Chinese filenames (e.g. `黄萌鸡.cook`) are auto-slugified; the slug becomes the URL.

## .cook Format

```
>> source: [来源]
>> original_title: [English — everyday terms, e.g. "Green Onion Pancake" not pinyin]
>> description: [生动描述，一两句]
>> time: [制作时间]
>> size: [份量]
>> author: yolocitrus
>> category: [Pastry | Main Course | Side Dish | Soup]
>> favorite: yes
>> spicy: yes
>> vegan: yes

第一步，@食材{用量}，@无量食材{}。#锅{}

第二步（空行 = 新步骤），计时~{20%minutes}。
```

- `favorite`, `spicy`, `vegan` etc. — omit the entire line if not applicable
- Flavor/diet tags: `spicy` `sweet` `salty` `sour` `bitter` `umami` `vegan` `meat`
- Ingredient braces always required: `@葱花{}` (no quantity), `@盐{3g}` (with quantity)
- `#器具{}` for cookware, `~{时间%minutes}` for timers
- Blank line = new step; split distinct actions into separate steps
- `*这是注释` shows in output · `-- 注释` stripped at build time

## Gotchas

- `original_title` English only; `author` is typically `yolocitrus` or the source platform (e.g. `youtube`).
- `uncategorized_label` in `config.yaml` must not contain an odd number of `"`.
- `updatedtime` comes from git commit dates, not file mtimes.
- **CI only:** `production: yes` is appended to `config.yaml` at build time — do not add it locally.
- Pin GitHub Actions to full commit SHAs: `uses: actions/checkout@9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0 # v7.0.0`

## Dependencies

- **Required:** Pandoc 2.8+, Python 3.12+, pipenv, Bash
- **Optional:** puppeteer-cli + img2pdf (PDF export), rsync (deploy)
- Python packages: see `Pipfile` (typer, python-frontmatter, python-slugify, pytest, black, flake8, mypy)
