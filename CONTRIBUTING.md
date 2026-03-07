# Contributing Guide

Welcome, and thank you for contributing! This guide walks you through everything you need to know to add or edit content in this repository — from setting up your environment to opening a pull request.

---

## Table of Contents

- [1. Fork & Clone the Repo](#1-fork--clone-the-repo)
- [2. Preview Locally with Quartz](#2-preview-locally-with-quartz)
- [3. Writing a Markdown File](#3-writing-a-markdown-file)
- [4. Frontmatter Fields](#4-frontmatter-fields)
- [5. Tags](#5-tags)
- [6. index.md vs README.md](#6-indexmd-vs-readmemd)
- [7. Code Blocks & Formatting](#7-code-blocks--formatting)
- [8. Images](#8-images)
- [9. How order.yaml Works](#9-how-orderyaml-works)
- [10. Opening a Pull Request](#10-opening-a-pull-request)
- [11. GitHub Actions & Automated Checks](#11-github-actions--automated-checks)

---

## 1. Fork & Clone the Repo

**Step 1 — Fork**

Click the **Fork** button at the top right of this repository page. This creates your own copy under your GitHub account.

**Step 2 — Clone your fork**

```bash
git clone https://github.com/YOUR_USERNAME/REPO_NAME.git
cd REPO_NAME
```

**Step 3 — Create a branch for your changes**

Never work directly on `main`. Always create a new branch:

```bash
git checkout -b add/my-new-note
```

Use a clear branch name that describes what you're adding, e.g. `add/docker-volumes-note` or `fix/typo-in-linux-basics`.

---

## 2. Preview Locally with Quartz

We use a custom version of Quartz to build and render this content. Before opening a PR, always preview your changes locally to make sure everything looks correct.

**Step 1 — Install our custom Quartz**

Follow the setup instructions in our Quartz repository:
👉 **[Link to your custom Quartz repo]** *(will be added)*

**Step 2 — Point Quartz at this content repo**

Once Quartz is installed, update the `quartz.config.ts` to point to your local clone of this content repo, or place your content inside Quartz's `content/` folder as described in the Quartz setup guide.

**Step 3 — Run the local preview**

```bash
npx quartz build --serve
```

Then open `http://localhost:8080` in your browser. Any changes you save will reload automatically.

**Things to check before opening a PR:**

- Your page renders without errors in the terminal
- Images appear correctly
- Links to other pages work
- The sidebar shows your file in the right position

---

## 3. Writing a Markdown File

Each piece of content is a single `.md` file. Place your file inside the correct folder that matches its topic — for example, a Docker note goes inside `DevOps/Docker/`.

**File naming rules:**

- Use **lowercase letters only**
- Separate words with `-` (hyphens)
- No spaces, no uppercase, no special characters
- No `.md` in the name when referencing it elsewhere (like in `order.yaml`)

```
✅  docker-volumes.md
✅  linux-file-system.md
❌  Docker Volumes.md
❌  Linux_File_System.md
```

**Basic file structure:**

```markdown
---
title: Docker Volumes
author: Your Name
---

Your content starts here.
```

---

## 4. Frontmatter Fields

Every `.md` file **must** start with a frontmatter block. This is a YAML section between two `---` lines at the very top of the file.

```yaml
---
title: Your Page Title
author: Your Name
---
```

| Field | Required | Description |
|-------|----------|-------------|
| `title` | ✅ Yes | The display title of the page shown in the site |
| `author` | ✅ Yes | Your name or GitHub username |

**Rules:**

- The frontmatter block must be the very first thing in the file — no blank lines before the opening `---`
- Both `title` and `author` must have a value, they cannot be empty
- Do not use keycap emoji (like `1️⃣ 2️⃣`) anywhere in the file — they break the site's image generation. Use plain numbers or text instead

---

---

## 5. Tags

Tags help readers discover related content across the site. You can add as many tags as you like — there is no fixed list, so use whatever makes sense for your content.

### Adding Tags

Add a `tags` field to your frontmatter:

```yaml
---
title: Docker Volumes
author: Your Name
tags:
  - docker
  - devops
  - containers
---
```

### Tag Naming Rules

- **Lowercase only**
- **Use `-` to separate words** in multi-word tags
- Be specific enough to be useful, but not so specific that no other page would share the tag

```
✅  docker
✅  linux-basics
✅  package-managers
❌  Docker
❌  linux basics
❌  my-very-specific-one-off-tag
```

### Tips

- Look at existing pages in the same section to reuse tags that are already in use — consistency makes the tag index more useful
- A page typically needs between 2 and 5 tags. More than that is usually too noisy
- Tags appear on the page and in the site's tag index, so readers can click a tag to find all related pages

---

## 6. index.md vs README.md

Every folder or section in this repo **must** contain either an `index.md` or a `README.md`. Our sidebar plugin requires one of these files to assign the folder a weight and display it correctly. Without it, the folder may not appear in the sidebar at all.

They serve different purposes:

| File | Where it shows | Purpose |
|------|---------------|---------|
| `index.md` | Quartz site | When a visitor clicks a folder name in the sidebar, `index.md` is the page that opens. Use it as a landing page that introduces the section — what it covers, what order to read things in, etc. |
| `README.md` | GitHub only | Rendered by GitHub when someone browses the repo. Use it to explain the folder's contents to contributors, not readers of the site. In Quartz it is treated as a regular page, not a landing page. |

### Which one should I create?

- **Always create `index.md`** if you want a proper landing page for the section on the site
- **Optionally create `README.md`** if you want to leave notes for contributors browsing the repo on GitHub
- If you are creating a new folder or section, **`index.md` is required** — the plugin needs it

### index.md example

```markdown
---
title: Docker
author: Your Name
---

This section covers Docker from the basics to advanced usage.
Start with [[what-is-docker]], then follow the notes in order from the sidebar.
```

### README.md example

```markdown
---
title: Docker
author: Your Name
---

## Docker Section

Notes covering Docker fundamentals and advanced topics.
Add new notes to this folder and register them in order.yaml.
```

> Both files still require the full frontmatter (`title` and `author`) — the automated checks will catch it if either is missing.

## 7. Code Blocks & Formatting

**Code blocks** — always specify the language for syntax highlighting:

````markdown
```bash
sudo apt update
```

```python
print("hello world")
```

```yaml
services:
  app:
    image: nginx
```
````

**Inline code** — use backticks for commands, file names, and paths:

```markdown
Run `docker-compose up` to start the containers.
The config file is at `/etc/nginx/nginx.conf`.
```

**Blockquotes** — use `>` for notes, tips, or warnings:

```markdown
> This only applies if you are using the bridge network driver.
```

**Headings** — use `#` hierarchy consistently. Start your content headings from `##` since `#` is reserved for the page title in the frontmatter:

```markdown
## Main Section
### Subsection
#### Detail
```

**Bold & italic:**

```markdown
**important term**
*emphasis*
```

---

## 8. Images

### Folder Structure

All images live in the `imgs/` folder at the root of the repo. Inside it, there is a subfolder for each section and category, mirroring the content structure:

```
imgs/
├── linux-basics/
│   ├── cli-and-linux-file-system/
│   └── boot-process/
├── devops/
│   └── docker/
└── git-and-github/
```

When you add an image for your note, place it in the subfolder that matches your content's location. If the subfolder does not exist yet, create it following the same naming rules.

### Image Naming Rules

- **Lowercase letters only**
- **Separate words with `-`**
- No spaces, no uppercase, no special characters
- Use a descriptive name that explains what the image shows

```
✅  docker-bridge-network.png
✅  linux-file-system-hierarchy.png
❌  DockerNetwork.png
❌  screenshot 1.png
❌  img_final_FINAL.png
```

### Referencing Images in Markdown

Use a path relative to the root of the repo:

```markdown
![docker bridge network](imgs/devops/docker/docker-bridge-network.png)
```

Always add descriptive alt text inside the `[]` — this is what shows if the image fails to load and helps with accessibility.

### Image Tips

- Keep image file sizes reasonable — compress screenshots before adding them
- Prefer `.png` for diagrams and screenshots, `.jpg` for photos
- Do not embed external image URLs — download and store them in `imgs/`

---

## 9. How order.yaml Works

The sidebar of the site is driven by `order.yaml` files. Each section of the content has its own `order.yaml` that defines the order folders and files appear in the sidebar.

### Structure

```yaml
order:
  - file: "index"
  - folder: "How-to-install"
    order:
      - file: "arch-linux-installation"
      - file: "ubuntu-installation"
  - folder: "Linux-Basics"
    order:
      - folder: "Introduction-to-Linux"
      - folder: "CLI--and--Linux-File-System"
```

### Rules

**Files:**

- Use the bare filename **without** the `.md` extension
- The file must actually exist on disk in the same directory

```yaml
✅  - file: "docker-volumes"
❌  - file: "docker-volumes.md"
```

**Folders:**

- Use the exact folder name as it appears on disk
- The folder must actually exist

```yaml
✅  - folder: "Docker"
❌  - folder: "docker"        # wrong case if folder is named "Docker"
❌  - folder: "Docker-Notes"  # folder doesn't exist on disk
```

**No duplicates** — never list the same file or folder twice under the same parent:

```yaml
# ❌ Wrong
order:
  - file: "docker-volumes"
  - file: "docker-volumes"   # duplicate!

# ✅ Correct
order:
  - file: "docker-volumes"
```

**Nesting** — folders can have their own nested `order:` block to define the order of their contents. If a folder has no `order:` block, its contents will appear in alphabetical order.

### When to update order.yaml

Every time you add a new file or folder, you **must** add it to the appropriate `order.yaml`. If you don't, it will not appear in the sidebar. The automated checks will catch this and block your PR until it's fixed.

---

## 10. Opening a Pull Request

Once your changes are ready and you've previewed them locally:

**Step 1 — Stage and commit your changes**

```bash
git add .
git commit -m "add: docker volumes note"
```

Use a clear commit message. Prefix it with:
- `add:` for new content
- `fix:` for corrections or typo fixes
- `update:` for improvements to existing content
- `imgs:` for image-only changes

**Step 2 — Push to your fork**

```bash
git push origin add/my-new-note
```

**Step 3 — Open a Pull Request**

Go to the original repository on GitHub and click **Compare & pull request**. Fill in:

- A clear title describing what you added or changed
- A short description in the body explaining the content and any decisions you made

**Step 4 — Wait for automated checks**

Two automated checks will run on your PR (see the next section). If either fails, a bot will post a comment explaining exactly what to fix. Push your fixes to the same branch and the checks will re-run automatically.

**Step 5 — Review**

A maintainer will review your PR. They may ask for changes or approve and merge it directly.

---

## 11. GitHub Actions & Automated Checks

When you open a PR, two automated workflows run to validate your content. You do not need to run these manually — they trigger automatically.

### Check 1 — Content Validation

Checks every changed `.md` file for:

| Check | What it catches |
|-------|----------------|
| **Frontmatter** | Missing or empty `title` / `author` fields |
| **Emoji** | Keycap emoji like `1️⃣` that break OG image generation |
| **Internal links** | Wikilinks `[[page]]` or markdown links pointing to files that don't exist |
| **Markdown lint** | General markdown syntax issues |

### Check 2 — order.yaml Validation

Checks every `order.yaml` file for:

| Check | What it catches |
|-------|----------------|
| **Missing files** | `file:` entries that don't have a matching `.md` on disk |
| **Missing folders** | `folder:` entries that don't exist as real directories |
| **`.md` extension** | `file:` entries that incorrectly include `.md` |
| **Duplicates** | The same file or folder listed more than once |

### What happens when a check fails

The bot will post a comment on your PR with a table showing every issue, which file it's in, and what the problem is. Fix the issues, push to your branch, and the checks will re-run. The comment will update automatically — it won't spam new comments on every push.

### What happens when all checks pass

The bot posts a ✅ passing comment and the PR is ready for maintainer review.

---

## Quick Checklist

Before opening a PR, go through this list:

- [ ] Branch name is descriptive and not `main`
- [ ] File name is lowercase with hyphens, no spaces
- [ ] Frontmatter has `title` and `author`
- [ ] Tags are lowercase and hyphenated if multi-word
- [ ] No keycap emoji (`1️⃣`, `2️⃣`, etc.) anywhere in the file
- [ ] New folder has an `index.md` (required for the sidebar plugin)
- [ ] Images are in the correct `imgs/` subfolder with lowercase hyphenated names
- [ ] Image paths in markdown are correct and the files exist
- [ ] New file is added to the correct `order.yaml` without `.md` extension
- [ ] No duplicate entries in `order.yaml`
- [ ] Previewed locally with Quartz and everything renders correctly

---

If you have any questions, open an issue or reach out to the maintainers. Happy contributing! 🎉