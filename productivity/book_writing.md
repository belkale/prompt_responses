# How do I write a book online using mdbook and github pages?

Using **mdBook** with **GitHub Pages** is a fantastic choice if you want a lightning-fast, minimalist, and clean online book. 

Because mdBook uses a plain Markdown file (`SUMMARY.md`) to define chapters, it is one of the easiest tools to structure.

Here is the step-by-step guide from installation to automatic deployment.

---

### Step 1: Install mdBook

Choose the method based on your operating system:

* **macOS / Linux:**
  ```bash
  brew install mdbook
  ```
* **Windows (via Winget):**
  ```powershell
  winget install mdbook
  ```
* **Any OS (Manual Download):**
  Download the single executable from the [mdBook GitHub Releases page](https://github.com/rust-lang/mdBook/releases), extract it, and add it to your system PATH.

---

### Step 2: Initialize Your Book

In your terminal, navigate to the folder where you want your book to live and run:

```bash
mdbook init indian-philosophy-book
```

It will ask you two setup questions:
1. *Do you want to use git?* $\to$ Press **Y** and Enter.
2. *Title of the book:* $\to$ Type `Indian Hindu Philosophy` and Enter.

Move into your new folder:
```bash
cd indian-philosophy-book
```

---

### Step 3: Understand the Structure

mdBook creates a very simple layout:

```text
indian-philosophy-book/
├── book.toml       # Settings (book title, author, search bar, theme)
└── src/
    ├── SUMMARY.md  # The Table of Contents (The most important file)
    └── chapter_1.md
```

#### How Chapters Work:
In mdBook, **`src/SUMMARY.md` dictates the entire navigation structure**. 

Open `src/SUMMARY.md` and edit it to organize your chapters:

```markdown
# Summary

[Introduction](README.md)

# Epistemology (Pramanas)
- [Means of Valid Knowledge](pramanas/overview.md)
  - [Pratyaksha (Perception)](pramanas/pratyaksha.md)
  - [Anumana (Inference)](pramanas/anumana.md)

# The Six Orthodox Systems (Shad-Darshana)
- [Samkhya](darshanas/samkhya.md)
- [Yoga](darshanas/yoga.md)
- [Advaita Vedanta](darshanas/advaita.md)

# Appendices
- [Glossary of Sanskrit Terms](glossary.md)
```

*(Tip: When you run mdBook, it will automatically create blank files for any `.md` file listed in `SUMMARY.md` that doesn't exist yet!)*

---

### Step 4: Preview Locally

To see your book in real-time as you write:

```bash
mdbook serve --open
```

* This spins up a local server (usually at `http://localhost:3000`) and opens your default browser.
* mdBook supports **instant reload**: when you save a file in `src/`, your browser updates in milliseconds.
* Press `Ctrl + C` in your terminal to stop.

---

### Step 5: Push to GitHub

1. Go to [GitHub.com](https://github.com) and create a new **public** repository named `indian-philosophy-book`. (Leave it blank).
2. Connect your local folder to GitHub by running:

```bash
git add .
git commit -m "Initial commit of mdbook"
git branch -M main
git remote add origin https://github.com/<YOUR-USERNAME>/indian-philosophy-book.git
git push -u origin main
```
*(Replace `<YOUR-USERNAME>` with your GitHub username).*

---

### Step 6: Automate Deployment with GitHub Actions

Unlike Quarto, mdBook doesn't have a single `publish` command built-in. Instead, we use a free **GitHub Action** that automatically builds and deploys your book every time you push edits.

1. Inside your project folder, create this nested folder structure:
   ```bash
   mkdir -p .github/workflows
   ```
2. Create a file inside it called `deploy.yml`:
   * On Mac/Linux: `touch .github/workflows/deploy.yml`
   * On Windows: `notepad .github/workflows/deploy.yml`

3. Paste this exact configuration into `deploy.yml`:

```yaml
name: Deploy mdBook to GitHub Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup mdBook
        uses: peaceiris/actions-mdbook@v2
        with:
          mdbook-version: 'latest'

      - name: Build Book
        run: mdbook build

      - name: Upload Artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./book

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

---

### Step 7: Enable Pages in GitHub Settings

1. Commit and push the new workflow file to GitHub:
   ```bash
   git add .github/
   git commit -m "Add deployment workflow"
   git push
   ```
2. Go to your repository on **GitHub.com**.
3. Go to **Settings** $\to$ **Pages** (in the left sidebar).
4. Under **Build and deployment**:
   * Change **Source** from *Deploy from a branch* to **GitHub Actions**.
5. Click on the **Actions** tab at the top of your GitHub repository. You will see your book building.
6. Once green (usually 30–45 seconds), your book is live at:
   `https://<YOUR-USERNAME>.github.io/indian-philosophy-book/`

---

### Your Daily Writing Workflow

From this point on, you never have to configure anything again:

1. Write your notes/chapters in standard Markdown inside the `src/` folder.
2. Update `src/SUMMARY.md` if you add new chapters.
3. Push to GitHub:
   ```bash
   git add .
   git commit -m "Updated chapter on Yoga Sutras"
   git push
   ```
GitHub Actions will automatically re-build and update your live website instantly.
